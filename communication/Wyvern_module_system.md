
Hi professor Jonathan, from your e-mail, I see that we actually want to treat the the TSL parsing module  no different with other normal Wyvern Module to let it leverage functions in other wyvern module. The only difference is we add some constraints on a TSL parsing module to prevent it from doing any wrongly side effect which might corrupt Wyvern Compiler Runtime.(e.g. a TSL parsing module can't import resource) 

In this article, I will explain some of the ideas I had for the design of TSL module system. The discussion might not be systemic, I just list some questions I could come up with. I will try to write a system one in the future. :-)

Let's first name some term which will make our discussion more clear.

###  First, setup the environment for our discussion

##### module type
When I start to think on Wyvern's module system, I feel we can actually put our modules into different categories according to the content it have. Here are the details:

* For a wyvern module with no TSL, we call it a **pure Wyvern module**(**pure module** for short)
* For a wyvern module with both TSL and Wyvern language, we call it a **compound Wyvern module**. ( **compound module** for short)
* For a wyvern module which define the TSL compiler, we call it a **TSL Compiler module** (**t-compiler module** for short)
* For a wyvern module with pure TSL, we call it a **pure TSL module** (**pure-t module** for short). If you are confused why do we need a pure TSL module here, don't worry, I will explained it in one of the following questions.


##### Wyvern complier workflow
Next, let's go through Wyvern's compiler's workflow and give each phase an explicit name.

* **TSL parser compile phase**: To use a TSL, we should first have a parser for it, so the first step for Wyvern's compilation is to parse t-compiler module to get build our TSL parser. Since t-compiler module is actually a Wyvern module, so it can reference other wyvern modules, which means we need to first resolve and parse the module it depends on.  After the dependencies is resolved successfully, we start to typecheck the t-compiler module. I give the name **TSL parser parsing pahse** for this phase.  After it, we will generate the real TSL parser and merge it into our **Wyvern Compiler Runtime**. I call this phase **TSL parser generation phase**

* **Application compile phase**: After we finished the **TSL parser compile phase**, we already have the parser to parse the TSL, so we can then parse our application which is written in both **compound module** or **pure module**. Again, there will be a dependency resolve phase, then a parsing and typecheck phase(called **application parsing phase** here) and finally the code generation phase(**application code generation phase**). We didn't discuss so much on **application code generation phase** since it's actually what we have done after we transform TypedAST to corewyvernIL.

The workflow is an **eager** design, since we will parse all TSL definition before parsing any application code. We can also have a **lazy** design which will parse TSL definition logic on demand. But the lazy design is more complicated so I choose to use the **eager** design here, so we can discuss more easily.

##### When shall we merge TSL and Wyvern?
For discussion here, I will suppose we merge TSL and Wyvern at TypedAST level. Though I think, it's also possible to merge at corewyvernIL level.

### Now let's start our problems


##### [Q1] t-compiler module design
One of the most important question for TSL module system design is how we design our t-compiler module. Before we discuss it, I think it's a good idea to first think, what's the essential function for a t-compiler? From a bird's-eye view, t-compiler module provides the function to parse, typecheck the specific TSL and generation. As we state before, t-compiler module is supposed to be normal wyvern module, so, what we do is actually define some class and object to parse TSL and the Wyvern Compiler Runtime will directly use them.

Though the bird's-eye view is helpful, there are several questions haven't been described clearly:

Let's first see this question: What's the result will return from a t-compiler's parse method? From my point view, it will return the **TypedAST**.Since when Wyvern Compiler Runtime invoke the parse method to parse TSL, it will finally get an typedAST back which is easy to incorporate the host Wyvern AST. 

To be more clear, let's write some pseduo code for it:

```
//v1: the most simple t-compile module
import wyvern.tools.TypedAST.*

def parse(tslCode: String): TypedAST
  ... // here goes the TSL parsing logic which finally return a TypedAST object

```

This design fullfill our basic needs but obviously it miss some important parts. one of them is we don't have an explicit mechanism to communicate with our Host Wyvern Compiler Runtime. For instance, if we want to report an parsing error to the host, we had to throw an exception. Usually this is not good design, since except ParsingException, we might encounter other type exception and this will soon complicated the error handling code in host. What I want to do is to pass a context object to our parse method. The object works just like a bridge between the host and TSL parse method. Here are the code 

```
//v2: add context object
import wyvern.tools.TypedAST.*

// Here we add the bridge object of type TSLContext
def parse(tslCode: String, ctx:TSLContext): TypedAST
  ... // here goes the TSL parsing logic which finally return a TypedAST object

```

To return to the error reporting question we raised before, we can now add a utility method on ctx like 	```ctx.reportError(...)```. See [Q3] to see more things we could add on TSLContext object. The design of adding TSLContext object also help us to build more stable system, since on the one hand we provide the author of t-compiler module the function they want as utility methods on TSLContext, on the other hand, these function is implemented by us not the user, so we can guarantee it's safe to use.

Now, we already had a quite stable  design, the next step is how we let the user write thier TSL parser more easily. As I think there are several aspects we can add support:

* the parse method is actually a facade method, so we can decompose it's functions into more small part. We can define our TSL-specific AST(t-AST for short) inside the t-compiler module, we can then build lexer and parser for the t-AST, we can build TSL-specific typechecker for t-AST. Finally we need a transformer to transform the t-AST to the TypedAST. Since now we decompose the function, we can add common utility class to make working with each specific task more easily. For example, when working with transformer, a fluent builder for TypedAST is very helper.

* to make working lexer and parser more easily, we can incorporate the existing parser/lexer generation framework like javacc and copper into our system. It might be a good idea if we can leverage the Wyvern's TSL host feature to directly host the grammar definition syntax into our t-compiler module. For this topic, I write them in details in [Q2].


In the end, I know in the TSL paper, we use a little bit different form of definition for a TSL parser.(we define every DSL type as an object which implement a parse method), I'm still thinking whether it's a good choice since many elements can share the same parser, so define a parse method on them could be redundant. Of course, this situation is still compatible with our discussion above, since we can simply delegate the implement of parse method on DSL type object to the facade parse method we give above. Will give a more detail discussion on this point in the future!

##### [Q2] can t-compiler module also be a compound module?
I raised this question for the following reason: as we all know, with tools like Copper or javacc, we are more easy to write lexer/parser for DSL. But the syntax for Copper and javacc are absolutely not Wyvern. So if we want to use them in t-compiler module, we have 2 choice:

* Use Copper and javacc externally and generate the Java class. Since Wyvern supports to import java object, we can keep the t-compiler as a pure Wyvern module.
* import Copper or javacc(or any other parser generation framework) as pre-defined t-compiler module and reference them inside your own t-compiler module. In this situation, our t-compiler module would be a compound module with both Wyvern and lexer/parser-generation DSL. This will make the ```TSL parser compile phase``` more difficult, but it actually has its benefits since the user can define their dsl more easily! We can also develop library with helper method to make the user use the lexer/parser-generation DSL more fluently. So it will be a good choice.

For myself, I think both way is reasonable and should be support. For the first choice, I guess it's our current implementation in Wyvern. This implementation is easier to implement. The second choice is much harder to implement, but it might be easier for user to use. So we can start with a first implementation and try to support the second one in the future. 


##### [Q3] What information is needed for the TSL parser when parsing and typechecking TSL segments in **Application Compile Phase**?

When we parse and typecheck the TSL in Application compile phase, we can't do it right without information from the context of TSL. for instance:
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
Here, we can't do the typecheck if we only have the TSL segment ```t + t2.m()``` since the t and t2 are not defined in the TSL. when do the typecheck, we need to pass the information for t and t2 to the ```parse``` method defined in the t-compiler module. Since we already have TSLContext type, we can add utility function on it like ```ctx.getType(name:String)```.

This is just one case, we can figure all them out during we develop Wyvern. Since we alreadt have TSLContext, we know where we can put these function.

#####  [Q4] discover the typeCheck type
What I mean here is we need to figure out the different type of typeCheck during the Whole compile time of Wyvern. Let me explain more on **different type** here:
As I show in [Q1], we define typeCheck method inside t-compiler module and it can only touch what provided in TSLContext and the TSL AST it get. So we can actually category every typecheck inside t-compiler module into one type. For this kind of typecheck, it's powerful but also limit actually. Maybe some kind of typecheck can't be done via this form. (Though I can't come up with some concrete examples, but I think we might encounter them when develop Wyvern) It might be a good idea we can summary them into more generic and reusable type and use design pattern like template method to inject them as hooks in our Wyvern Compiler Runtime. This can help us make typecheck logic reusable and more easily to develop and integrate into Wyvern.


##### [Q5] cyclic module dependency: a theory existing issue but might not happened in real world

Here is a sample for this issue:
```
//this is a t-compiler module named A  uses elements defined in a compound module B. 
import B
....


//this module B which in turn use TSL defined in module A
def language A.x at t
...

```

Here we first need to parse module A since in B we have TSL defined in A, however to parse module A, we need to first parse module B and this caused the cyclic dependency issue. There should be several variant versions for this issue, however, I guess seldom user will do it this way, so not a urgent issue we need to consider now.

##### [Q6] do we need to add runtime error handler for TSL object since the TSL 's typecheck is implemented by users.

Since the TSL typecheck is defined by the user, there might be chance that a wrong-typed TSL node will pass the typecheck and caused runtime error. For Wyvern itself, we are quite confident with its type-safe and it's reasonable to ignore some runtime error handling, but for TSL object, we are not that confident, so do we need to consider the runtime error handling for it? We might also need to consider how to report suitable error message. In runtime, we are far from our raw AST and might already lose our TSL specific information, which makes report suitable error message harder. So this might be question needs our attention. 


##### [Q7] shall TSL has its global space to hold reusable logic?
I've mentioned in [module type] that we have t-pure module and I don't give explanation why we need it. Here is the explanation:

First, let's think a question: if you are working with a language that can't extract reusable logic, what will happened? Just think about you are using C language but you are not allow to define functions. You might feel good when working with toy program but for complex program, it might drive you mad! The same situation applies to our DSL fragments in compound moudle. If we couldn't define reusable logic somewhere and let DSL fragments use them, our DSL fragment code might contains many duplicate logic. 

To fix this issue, a straight idea is we can setup a DSL-specific global space in Application compile time to store all the resuable logic. Code in DSL fragments can refer to them directly without any Import statement. In this way, we need a place to define those reusable logic, so we need the t-pure module! As I think, in application compile phase, we will first compile all t-pure module to and put the reusable logic into the DSL-specific global space according to what DSL is used in the module. Then we started to compile the pure or compound module.

##### [Q8] for hand-written parser and lexer, we might need to enhance our pattern match function in wyvern. 
