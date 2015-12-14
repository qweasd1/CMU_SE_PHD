#objmap - a framework to transform javascript object
This is an article about how I design a javascript framework - objmap, from incubation to full-featured. However, the article not stop at showing design details, it also share my framework development experience I learn from my daily work and study.(I 'Bold' them in the article) Hope you like the article.

## **Content**
- Start (jstree)
- Prepare
- Functional Design
- Implementation Design
- Improve the Design
- Extension Tool Design

### Start (Jstree)
The idea of objmap came occasionally when I want to include jstree(a jQuery plugin) into one of my angularjs app.
The inital obstacles I met was that, to use jsTree, I need to transform my domain model into a specific structure jsTree accepts.

Here is a simple case:
my domain model represents a file system:
```javascript
var my_domain_model = {
   name:'folder1',
   isDirecotry: true,
   subFiles:[
     {
       name:'file1',
       isDirecotry: false,
     },
     {
       name:'file2',
       isDirecotry: false,
     }
   ]
}
```
and what jsTree accepts:
```javascript
var jstree_model ={
         'text' : 'folder1',
         'children' : [
           { 
              text : 'file1' 
           },
           { 
              text : 'file2' 
           }
         ]
      }
```
To map my_domain_model to jstree_model, The simplest solution is to write a recursive function.

**solution 1: recursive function**
```javascript
var transform = function(origin){
 var target = {
   text:origin.name
 }
 if(origin.subFiles !== undefined){
   target.children = origin.subFiles.map(function(sub){
     return transform(sub)
   })
 }
 return target
}

var target = transform(origin)
```
It's a workable solution but it's not a good solution for the following reasons:

1. recursive function can be more erroneous when it grows complex.
2. It's not general, when I change my domain model, Ie need to rewrite the recursive function as well.
3. It's imperative than declarative. The logic is less clear at a glance.


To overcome the issues above, use high-order function is a good idea:

**solution 2: high-order recursive function**
```javascript
function transform(origin, node_transformer, children_selector)
{
  var target = node_transformer(origin)
 
  var children = children_selector(origin)
  
  if(children !== undefined)
  {
    target.children = children.map(
      function(child_model){
         return transform(child_model, node_transformer, children_selector)
      }
    )
  }
  return target
}

var jstree_model = transform(my_model, function(x){
   return {
     test:x.name
   }
}, function(x){
   if(x.subfiles){
      return x.subfiles
   }
})
```
Looks more generic and solves the issue 1, 2 this time. However, the sample we used above is a simplified version. The  jsTree's model can be more complex(to give you better control on jsTree)
```javascript
//jsTree model in real world
{
  id          : "string" // required
  parent      : "string" // required
  text        : "string" // node text
  icon        : "string" // string for custom
  state       : {
    opened    : boolean  // is the node open
    disabled  : boolean  // is the node disabled
    selected  : boolean  // is the node selected
  },
  li_attr     : {}  // attributes for the generated LI node
  a_attr      : {}  // attributes for the generated A node
  children:[]
}
```
Still the solution #2 is imperative than declarative(what we write in ```node_transformer``` can be complex when the transform logic become complex). We also can't specify the structure of our target in a declarative way since the logic is all in code. Moreover, we can only use this recursive logic with jstree, but such kind of recursive logic likely to occur in many different places. 

**So, can we made it more generic? A clear question came in mind: Can we map one javascript object to a new javascript object with different structure in a declarative way?**



### Prepare
##### Evaluate the idea
Is the idea meaningful? I think a while the found: the answer is **YES**!

Think about the following scenarios:
* When we build GUI in MVVM pattern, we usually need to transform our model to view model. These codes are sometimes boilerplate codes but sometimes with some transform logic. No matter which kind of code, it's takes considerable time. If we can use declarative way to do the object map, the code can be shortter, easy to read and maintain.
* When 2 domain needs to interact with each other, we always need to write some adapting code. In essential, we need to adapt one domain model to another domain model.(our jstree is an example for this case). 
* If we develop a language related module, we always need to work with AST(Abstract Syntax Tree). AST is actually an object aggregation. So we can use objmap to map the AST to the structure we want directly instead of using a traditional visitor pattern!(more declarative).
* **(other scenarios continue...)**

So it's reasonable to write a framework like objmap.


##### Is there any existing framework?

Usually, we look around to see if anyone do the similar thing before we start our research.
After a couple hours searching, I found a great case: **[jsonpath-object-transform](https://github.com/dvdln/jsonpath-object-transform)**.
the core idea of this framework is to use a ```template``` object to specify the structure of the target object and use the ```JSONPath``` as properties of ```template``` object to select the fields from the origin object.
Here is a sample:
```javascript
var transform = require('jsonpath-object-transform');

var template = {
  a:{
    b:'$.x',
    c:'$.y'
  }
};

var origin = {
  x:1,
  y:2
};

var target = transform(origin, template);
```
Result:
```javasript
{
  a:{
      b: 1,
      c: 2
    }
  
}
```
This is really a good idea, since it gives us the flexibility to specify what our target like in an elegant declarative way. Howerver it's more like a good start not a full-feature solution. 
Here are some of the unsolved issue:
- It doesn't support recursive(so we can't apply it to our jsTree scenario)
- It doesn't support calculation(or expression). e.g. it can't do something like the following
```javascript
var template = {
  firstName:'$.firstname',
  lastName:'$.lastname'
  name: //expected to be $.firstname + ' ' + $.lastname, but is not available here
}
```

- Lacking reusable: when the transform becomes complex, we can not re-use our existing template object
-  ...(**to be continure**)

So let's begin our functional design.


### Function Design

##### Step 1: add expression and context inference
**expression is a powerful tools to make your our framework flexible especially you embed it into a host environment.**

Angularjs is such a good exmple.In the following code, we use expression to dynamically change the button's visibility according to model's products.count
```html
<button ng-show="products.count >= 4">Select a gift</button>
```
It's really short and convenient. 

In the meanwhile, Angularjs also use context inference: the ```product.count``` we used in expression is actually on object ```$scope```. But we don't specify ```$scope``` explicitly. Angularjs knows, under current context, ```product.count``` is actually refer to ```$scope.product.count```. I call such function **context inference**. Since we infer more from our context, we can write less code and thus, let our framework user develop more quickly and fluently.

So, our first step is add this 2 feature to our objmap:
Here we use javscript expression in our template. We use '((' '))' to tell our template parser that the value is an expression
```javascript
var template = {
  sum: '((1 + 2 + 3))' 
}
```


Next, with the context-inference we can refer to some delcared fields 
```javascript
var = template = {
     firstname:'$.firstname',
     lastname:'$.lastname',
     name: '((firstname + " " + lastname))' // we reference firstname and lastname here
}
```
The objmap looks much more flexible now.

To be more advanced, we might don't want to output firstname and lastname in our target object, so let's add a new concept: private field(or intermediate variable). Think it as a intermediate varialbe in some imperative languange or a ```let``` binding in some functional programming:
```javascript
var = template {
     _fn:'$.firstname', //private
     _ln:'$.lastname',  //private
     name:'((_fn + ' ' + _ln))'
}
```

You might feel tired to see string concat like ```_fn + ' ' + _ln``` here. So let's add a string template engine
```javascript
var = template {
     _fn:'$.firstname',
     _ln:'$.lastname',
     name:'{{_fn}} {{_ln}}', //mustache flavor string template
     greetings:'Hi {{name}}, welcom to our website'
}
```

So far everhthing looks good! But we still need to add more feature make it powerful

#### Part2: add abstraction ability
Abstraction is really cruial for nearly every framework! it's the base of reusable and extensibility. A most familiar example is functions or subroutines in nearly every languange. abstraction can let your framework's users have the ability to encapsulate their special logic. 

Abstraction also bring you the ability to extend your framework. For example, in Angularjs, we have concept called ```directive```, the familiar ```ng-repeat```, ```ng-show``` are the default implmentation of such concept. User can also write their own directive to extend existing functions. This feature, thought look simple but made Angularjs extremely flexible.

let's see what we can do with our objmap:

###### add transform functions
Think about the following case:
```javascript
//copy all fields start with 'a'
var origin = {
   a_1:1,
   a_2:2,
   a_3:3,
   a_4:4,
   b_1:5,
   b_2:6
}

var target = {
   a_1:1,
   a_2:2,
   a_3:3,
   a_4:4
}

```
Obviously, it is not convenient to do using our existing framework. So let's add the new feature:
```javascript
//something like
var template = {
  _copy_:"a*" // _<function name>_ represents a function, the value of it will be the parameter pass to the function
}
```
The template here will copy all properties which start with 'a' from orgin to target object.Looks perfect


##### add sub-template
Next, let's add sub-template. You can register template in the transformer object and then invoke them in your other template.
```javascript
// recall our file system example for jstree
// we can specify our transform logic like this 
var tansformer.register_template("file", {
  text: '$.name',
  children: ['$.subfiles', 'each &file' ] // just ingnore the grammar we used here, will explain it latter. But what we mean here is '$.subfiles' is used to select values as source, '&' represent invoke a sub-template and apply it to an item. each means call the template to each item of the source.
})

var target = transformer.transform(origin, 'file')

```
I have to say with sub-template, our transform logic looks extremely simple and declarative

##### add pipeline functions
When we process object, it's very common to use functions like map, filter, group, reduce(I use the their name in javascript. ). So we can't miss them in objmap. Moreover, to make them more easy to use, I choose a light weight flavor lamba expression for it.
```javascript
  tansformer.register_template("file", {
  text: '$.name',
  children: ['$.subfiles', 'filter _.name.startWith("a")', 'each &file' ] // '_' represent one item from upstream of our pipeline function.
})
```

#### Part3: make it dynamic
**To make a framework easy to use, we should let user write less but do more. One of the useful techniques is to facilitate this is let part of your component determined dynamically according to the context.** Or, in other words, it means we put expression in more places.

##### add dynamic parameters to transform function
```javascript
var template = {
  _best_author = ['$.authors', 'sort _.vote "desc"', 'first'],
  _copy_best_author_profile_: "{{_best_author.name}}" //use dynamically parameters here
}
```

##### add dynamic paramters to your jsonpath
```javascript
var tempalte = {
  the_most_popular_blog:["$.blogs","sort like 'desc'", "first"] // get the most popular blog
  author_of_the_most_popular_blog: "$.authors[?(@.name == {{the_most_popular_blog.author}})]" //use {{the_most_popular_blog.author}} to inject dynamic parameter
}
```
Here we extend the existing jsonpath expression. **As we said before embed expression in host environment can be flexible**
Now, our framework more flexible to a next level. **As a conclusion, let your framework use as much context as possible can make it dramatically flexible**




#### Part4: don't miss those things look trivial but not
There are some things look trivial but not. Missing them might cause your user use workaround for them.

###### what return: single value? array
```javascript
// which value will return for the following condition?
// when $.a has no match
// when $.a has one match
// when $.a has more than one match
var template = {
  prop: "$.a"
}
```
The desire answer may depends on your user's requirement:
```javascript
var template = {
  blogs:"$.a.b.blogs" 
}
//target.blogs will be use as a input in a array function(like map)
target.blogs.map(...)
```
In this situation, we definitely want ```blogs``` to be an array, even if it returns only 1 object.

However in such situation:
```javascript
var origin = {
   best_blog: {...}
}

var template = {
   blog: '$.best_blog'
}
```
we definitely know we only select one result, so we want to return a single value.

**So here, we need to let our users have the chance to decide which way they want, not make decisions for them! I have seen many framework set some presumed implemention and make their users use some work around to achieve their goals!** 

So let's do it like this:
```javascript
//this will always return array
var template =  {
  'blogs[]':"$.blogs"
}
//this will always return the first single result
var template = {
  'blog':"$.blog"
}

```
The example above is only one aspect. There are also other aspects like: if our jsonpath selector return nothing, shall we set the target's property as ```null``` or totally ignore it as ```undefined```? It can be meaningful for your user, since they might has checking logic like ```if(target.prop !== undefined) ``` in their code. 

###### a real world sample
```javascript
TODO: select blog
```

### Implementation Design
It looks great for the function by now, so let's consider how to implement it!

##### rethink before you start (use abstraction)
From the section **Function Design** we can see that we have many things need to develop: expression, object-template, text-template, dynamical-parameters, functions and pipe-functions. **But wait! Let's see what's in common of them:**

<table>
  <thead>
    <tr>
      <td>Component</td>
      <td>Input</td>
      <td>Output</td>
      <td>Is Pure Function?</td>
    </tr>
  </thead>
  <tbody>
   <tr>
      <td>expression</td>
      <td>context(can access both public and private fields)</td>
      <td>value</td>
      <td>true</td>
    </tr>
    <tr>
      <td>object-template</td>
      <td>context, target(the target that will return, only contains public fields)</td>
      <td>target</td>
      <td>Not Sure</td>
    </tr>
    <tr>
      <td>text-template</td>
      <td>context</td>
      <td>string</td>
      <td>true</td>
    </tr>
     <tr>
      <td>dynamical-parameters</td>
      <td>context</td>
      <td>value</td>
      <td>true</td>
    </tr>
     <tr>
      <td>functions</td>
      <td>context, target</td>
      <td>target</td>
      <td>Not Sure</td>
    </tr>
    <tr>
      <td>pipe-functions</td>
      <td>context, target</td>
      <td>value</td>
      <td>Not Sure</td>
    </tr>
  </tbody>
</table>

See it, they are actually has the same structure, what we need to do is parse them into the following formats:
```javascript
function(context) : return_value
function(context, target) : return_value //target object is need, since we might change the state of target. _copy_ function is such a example
```
Let's give a name for these 2 function: ```calculation function```
**Make things homogeneous than heterogeneous can simplify your inner implementation**

##### list the structure of your module
- a functionRepo to save/fetch all the parsed ```calculation function```
- several parsers to parse several components. They need to use:
  - a text-template engine
  - a language transformer for expression
  - a parser for pipeline functions(it's more like coffeescript not javascript so a parser is needed)
- a root facade object has methods : ```register_template```, ```register_function```, ```transform```
- register some embedded functions : e.g. ```_copy_```

After Implementation, we now have a workable solution! Let's move on to the next section to see how to improve our design.

### Improve the Design

##### Extensibility
Why extensibility important? It's because user always need some specific functions. If your module can't be extended easily, it will definitely frustrate your user. [Mustache](https://github.com/janl/mustache.js/) is such a example: Mustache is a string template engine, but you can't add a function to join string in it. Mustache's successor [handlerbar](https://github.com/wycats/handlebars.js) not only overcome this but also also support much more better extension point.

in our objmap, the extension is really easy!(use register_function to register the function then use them in your template).


##### Don't Block Your User
Sometimes user has some **non-trivial logic** that can't be expressed easily in the way your framework provided, we should give a way to let them **has the opportunity to fullfill their logic**. Since your framework is always built upon a more powerful but quite low level platform(like jquery to javascript), a usually handy solution is to let your user can go back to write code in that low level platform if necessary. So in the objmap, le's add a new feature like:
```javascript
var template = {
  prop:function(context, origin) {
     
  }
}
```
See it? the function(...) we add do the same thing as our expression do, but you can turn back to your familiar javascript when you feel it's hard to express your logic in expression.

##### Let your User leverage the same power as you
Well, the previous feature doesn't come to an end. In obj model, we can leverage **jsonpath** like ``` prop: '$.origin_prop'```. But when it turns to use ```funtion```, we can't use the jsonpath unless we include the reference to it.(We don't expose it from our framework). It's usually inconvenient, so let's change our function signature to ```function(context, origin, utility)``` then we can use jsonpath utility like ```utility.jsonpath(some_obj, '$.prop')```.
BTW, you can also register other utility if you want like following:
```javascript
tansformer.register_ultility("your utility name", your_utility) 
```
Now, objmap becomes more extensible! And it's time to move to the next topic!

### Extension Tool Design
What does **Extension Tool** mean here? Actually, what I mean is **a tool to help people use your framework better**. This is just like an IDE to a languange, with the syntax highlight, auto-completion,..., you work not only more efficiently but have less chance to make errors.

The most simple form of extension tool might be the a REPL console: You directly see what does your code do. Some other examples maybe like **Regex Testor** which highlight the match item when you develop complex regex expression. This sort of tool has the same feature: **let you see the result immediately**. This feature is very useful for the following reasons:
- it can help new user quickly play with your framework and get familiar.
- it can be handy for test the integrity when user using your framework to develop.

So, for our objmap, we need at least **an input area** to get our ```origin``` object, **an input area** to edit our ```template``` and **an output area** to display our ```target```

There are absolutely more entension function we can discuss here, but since it's already not a short essay, let's omit them this time.

