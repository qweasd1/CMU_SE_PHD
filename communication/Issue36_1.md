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

and when we create an concrete class for a ```type```, we simplely give the abstract method a concrete body with any ```override``` keywords?


##### for ```new``` expression
for the following code:
```
val r : IntResult = new
    def getResult():system.Int = 5
```
the semantic is same as the annoymous class, right? we just create an inline instance of ```type``` IntResult?


##### for ```delegate``` declaration
```
val r2 : IntResult = new
    delegate IntResult to r
```

Here we declare another instance of ```IntResult``` type. But we delegate its function to object r, which means, we store a private field for r in r2 and when we call ```r2.getResult()```, it actually called ```r.getResult()```.

Do I understood it correct?

