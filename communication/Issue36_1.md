### Questions about delegation
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
