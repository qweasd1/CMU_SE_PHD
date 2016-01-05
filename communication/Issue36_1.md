### Questions about Issue #36
When I look at the ```testSimpleDelegation()``` test in ```ILTests.java```, I got a few questions.

First paste the code from ```testSimpleDelegation()```
```java
String input = "type IntResult\n"
             + "    def getResult():system.Int\n\n"
             + "val r : IntResult = new\n"
             + "    def getResult():system.Int = 5\n\n"
             + "val r2 : IntResult = new\n"
             + "    delegate IntResult to r\n\n"
             + "r2.getResult()\n"
              ;
```

##### for ```type``` declaration
When we define abstract method in wyvern, we don't explicit use the ```abstract``` keywords? since in ```testSimpleDelegation()``` , we write the following
```
type IntResult
     def getResult():system.Int
```
a def declaration with **no body** will be viewed as an abstract method?

and when we create an concrete class for a ```type```, we simplely give the abstract method a concrete body without any ```override``` keywords?


##### for ```new``` expression
for the following code:
```
val r : IntResult = new
    def getResult():system.Int = 5
```
the semantic is same as the annoymous class in Java, right? we just create an inline instance of ```type``` IntResult with the definition of the abstract method?


##### for ```delegate``` declaration
```
val r2 : IntResult = new
    delegate IntResult to r
```

Here we declare another instance of ```IntResult``` type. But we delegate its function from IntResult to object r, which means, we store a private field for r in r2 and when we call ```r2.getResult()```, it actually called ```r.getResult()```.

Do I understood it correct?


##### More questions on ```delegate```

* What's the rule to check and determine  whether an object can be the target of a delegation? In our unit test, we delegate the implmentation of ```IntResult``` object ```r2``` to another ```IntResult``` object ```r```. They are in the same type, so we fullfill our delegation semantic. We can also delegate implmentation of ```Some Type``` object to ```Some Type```'s subclass. We might also allow delegate implementation of  ```Some Type``` object to ```Another Type```'s instance where ```Some Type``` has no hierarchy relationship with ```Another Type```, but ```Another Type```'s methods has a subset to fullfill all methods required in the ```Some Type```. We can give a explicit mapping to map the delegation relationship between the 2 object. Here is a sample:
```
type A
   def method_1(): int

type B
   def method_2(): int = 1

val b = new B()
var a:A = new 
   delegate A to b
      mapping method_1 -> method_2 

```
As I think, we can start with the simple check which only accept delegation target with same type or subtype of delegation source 
@Professor Jonathan, any suggestion?

* 
* 
* Any
One thing

### My plan to add delegate support 




### Other questions:
what does ```selfName``` mean in ```wyvern.target.corewyvernIL.expression.New```?
