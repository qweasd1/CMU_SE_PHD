
From your e-mail, I see that we actually want to treat the the TSL parsing module  no different as other normal Wyvern Module except some constraints. To treat the module as a normal Wyvern module, we can leverage other existing Wyvern library to help us develop wyvern program more easily. To add constraints like TSL parsing module can't reference resource module, we can prevent the TSL parsing from corrupting the host Wyvern compiler by making unwanted side effect.

I've consider the question and here is my ideas on it.  

###  First, setup the environment for our discussion
Before I explain my ideas, let me first setup the environment for it.

##### module type
I think it's a good idea we give an explicit type name for different Wyvern module type:

* For a wyvern module with no TSL, we call it a **pure Wyvern module**(**pure module** for short)
* For a wyvern module with both TSL and Wyvern language, we call it a **compound Wyvern module**. ( **compound module** for short)
* For a wyvern module which define the TSL compiler, we call it a **TSL Compiler module** (**t-compiler module** for short)
* For a wyvern module with pure TSL, we call it a **pure TSL module** (**pure-t module** for short). If you are confused why I list this kind of module here, don't worry, I will explained it in the following discussion.


##### Wyvern complier workflow
Here we describe the big workflow for a wyvern compiler. The point is we give an explicit name for each phase to help us discuss latter.

* ```TSL parser compile phase```: To use a TSL, we should first have parser for them, so the first step for Wyvern compiler is to parse t-compiler module. Since t-compiler module is actually a Wyvern module, so it can reference other wyvern modules, which means we need to first resolve and parse the module it depends on.  After the dependencies is resolved successfully, we start to typecheck the t-compiler module. I call the whole process for them ```TSL parser parsing pahse```.  After that, we will incorporate the TSL parser logic(including how to parse the TSL, how to do typecheck on it, etc) into our **Wyvern Compiler Runtime**. We call it ```TSL parser generation phase```

* ```Application compile phase```: After we finished the ```TSL parser compile phase```, we already include the information needed to parse the TSL, so we can then parse our application which is written in both **compound module** or **pure module**. Again, there will be a dependency resolve phase, then a parsing and typecheck phase(```application parsing phase```) and finally the code generation phase(```application code generation phase```). We didn't discuss so much on ```application code generation phase``` since it's all about what we already done in Wyvern.(the passes after the TypedAST layer).

The workflow is an **eager** design, since we will parse all TSL definition before parsing application. We can also have **lazy** one which will parse TSL definition logic on demand. But the lazy design is more complicated so I choose to use the **eager** design to help discuss here.

### problems I'm considering
Since we already setup the environment for our discussion, let move to some concrete problems I have considered!

##### [Q1] t-compiler module design
Before we discuss this question, I think it's a good idea to first think, what's the essential function for a t-compiler? From a bird's-eye view, t-compiler module provides the function to parse, typecheck the specific TSL and generate the **Wyvern TypedAST** from them. 

To fullfill our requirement, the t-compiler will at least know the following thing:

* the Wyvern's TypedAST: it's the bridge betwwen the Wyvern Host and the TSL parser. Let's call it ```wyvern.tools.TypedAST``` module here

To be more clear, let's write some pseduo code for it:

```
//v1: the most simple t-compile module
import wyvern.tools.TypedAST

def parse(tslCode: String): TypedAST
  ... // here goes the TSL parsing logic which finally return a TypedAST object

```

This design might be handy for simple case, since we try to map tsl string directly to TpyedAST object. For more complex TSL, we might need to introduce it's own AST. We also need to introduce the typechecker and a transformer. So let's add more things on our pseduo code:

```
//v2: an enhancement version for a TypedAST
import wyvern.tools.TypedAST

// here, we define the TSL AST
type TSL_AST_1
  ...
type TSL_AST_2
  ...

// here, we define severl typeChecker for TSL AST
def TSL_Checking_Rule1(ast:TSL_AST)
  ...

....


// here, we define the transformer to Transform the TSL AST to TypedAST
def tansform_TSL_AST_1
  ...


// finally a facade parse method to be invoked by Wyvern Compiler Runtime. Inside it , it calls the typecheck and transform method.
def parse(tslCode: String): TypedAST

```
The module looks good now. However these is important issue with such kind of design: we don't have an explicit mechanism to report our errors to the Wyvern Compiler Runtime when we use the ```parse``` method during ```Application Parsing Phase``` Though we can throw an exception and let the Wyvern Compiler Runtime try catch it, it shall not be a good way. We can actually define an interface(which means a Type in Wyvern) with utility methods on it and pass it as a parameter into the parse method. The Wyvern Compiler Runtime then check it's properties to know what happened during the ```Application Parsing Phase```. Here is an sample:
```
//v3: parse method with an Context Object

def parse(tslCode:String, ctx:TSLContext):TypedAST
  //if error happened
  ctx.setErrorMessage("your error message goes here")
```

```
// the Wyvern Compiler Runtime then check the ctx object to do better reporting
...

try{
   TypedAST ast = TSLMeta.parse("some tsql", tslCode, ctx)
   if(ctx.hasError()){
      //report error
   }
   else{
      // use the correct ast in next step
   }
}
catch(Exception e){

}
...

```

The concept of the TSLContext is very useful, since it's actually act as a bridge  between the host and t-compiler module. We can provide the utility function in TSLContext so that the user who write the t-compiler module get what they want. Since how these utility function act is defined by us, we also limit the client to do what's dangerous. Of course user can still do dangerous thing if they reference the Wyvern Compiler API, but we can avoid this by make the Wyvern Compiler API as private in the module, so the User can't touch them. The constraint that we can't reference resource module also help prevent user from doing dangerous thing to Wyvern Compiler Runtime.

To give the user who write the t-compiler module, we'd better provide some stateless helper object to them. For instance, a fluent AST builder is such a helper class.

@professor jonathan, what's your opinion on this discussion?

##### [Q2] can t-compiler module also be a compound module?
I raised this question for the following reason: as we all know, with tools like Copper or javacc, we are more easy to write parser for DSL. But the syntax for Copper and javacc are absolutely not Wyvern. So if we want to use them in Wyvern, we have to choice:

* Use Copper and javacc externally and generate the Java class. Since Wyvern supports to import java object, we can keep the t-compiler as a pure Wyvern module.
* import Copper or javacc(or any other parser generation framework) as pre-defined t-compiler module and reference them inside your own t-compiler module. In this situation, our t-compiler module would be a compound module with both Wyvern and parser-generation syntax. This will add make the ```TSL parser compile phase``` more difficult, but it actually has its benefits since the user can define their dsl more easily! We can also develop library with helper method to make the user experience more fluently. So it will be a good choice.

For myself, I think both way is reasonable and should be support, the key is we make both methods easy to use by users. 


##### [Q3] What information is needed for the TSL compiler when parsing and typechecking TSL in Application compile phase?

When we parse and typecheck the TSL in Application compile phase, we can't do it right without information out of TSL. for instance:
```

val t = 1
val t2 = new Type1
   def m():int
      1
def method_needs_a_tsl(p:SomeTSLType):int

method_needs_a_tsl(~)
  //here is a tsl which used t and t2
  t + t2.m()

``` 
Here, we can't do the typecheck if we only have the TSL segment ```t + t2.m()``` since the t and t2 are not defined in the TSL. when do the typecheck, we need to pass the information for t and t2 to the ```parse``` method defined in the t-compiler module. Since we already have TSLContext type, we can add utility function on it.

This is just one case, we might encounter more case like this but since we have TSLContext, we know where we can put these function

#####  [Q4] discover the typeCheck type
What I mean here is we need to figure out the different type of typeCheck during the Whole compile time of Wyvern. Let me explain more on **different type** here:
As I show in [Q1], we define typeCheck inside t-compiler module and it can only touch what provided in TSLContext and the TSL AST it get. So we can actually category every typecheck inside t-compiler module into one type. it is powerful but also limit actually. There might be other type of check needs to use more information.(Though I can't come up with some concrete examples, but I think we might encounter them when develop Wyvern) It might be a good idea we can summary them into more generic and reusable form. By using design pattern like template method, we inject hooks represent them in our Wyvern Compiler Runtime and make them more easily to develop and integrate into Wyvern.


##### [Q5] cyclic module dependency: a theory existing but might not happened in real world issue

Here is a sample for this issue:
```
//this is a t-compiler module named A  uses elements defined in a compound module B
import B
....


//this B which in turn use TSL defined in module A
def language A.x at t
...

```

Here we first need to parse module A since in B we have TSL defined in A, however to parse module A, we need to first parse module B and this caused some conflict. There should be several variant versions for this issue, however, I guess seldom user will do it this way, so not a urgent issue we need to consider now.
