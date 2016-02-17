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


### Closure Implemented in C#
As I know, the C# compiler usually compiles lambda expression with free variables as the following shows:
```
// the same lambda expression
int a = 1;
Func<int,int> a_lambda = (x)=> x + a;

// after compiled
class an_annoymous_class {
  private int a = 1;
  public int apply(int x){
    return x + a
  }
}
```
As we can see the the C# compiler actually capture the free variable ```a``` and store it as a field in the genreated annoymous class.
