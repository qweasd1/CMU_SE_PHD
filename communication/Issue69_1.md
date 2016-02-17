Hi professor Jonathan,
I will explain my idea on this issue here. Let me know if I misunderstand anything.

### Our Aim
We want our lambda expression only to capture free variables into its closure from environment.
for example:
```
type some_lambda
  def apply(x:int):int

val a = 1
val b = 2

some_lambda t = (x)=> a + x
```
So here, our lambda expression ```t``` should only capture ```a``` and ignore ```b```. Is that correct?


### Closure Implemented in CSharp
As I know, the C# compiler compiles lambda expression with free variables in the following way:
```
// the original lambda expression
int a = 1;
Func<int,int> a_lambda = (x)=> x + a;

// the compiler generate an annoymous_class for that lambda expression
class annoymous_class {
  public int a = 1;
  public int apply(int x){
    return x + a
  }
}

int a = 1;
annoymous_class lambda = new annoymous_class();
lambda.a = a;

```
Here, the C# compiler capture the free variable ```a``` and store it as a field in the generated annoymous class to let the apply class can only touch the free variable a. I think we can also do this in Wyvern.

### My plan
We can modify the ```generateIL()``` method in ```wyvern.tools.typedAST.core.expressions.Fn``` to let it generate ```wyvern.target.corewyvernIL.expression.New``` with fields that capture the free variables in lambda expression similar as C# compiler. 

Any suggestions? If you feel good with it, I will start to implement.
