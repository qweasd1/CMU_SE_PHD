### Detail Plan

##### Single Delegate in this iteration
I plan to implement single Delegate in this iteration since multi-delegate is much more complex. 
I want to For multi-delegate, we need to consider more on method mapping(same situation in Scala's multi-trait).

It requires to think carefully for every details for the whole function. So won't introduce the complexity in this iteration.

##### delegate mapping

##### method override

##### reference the new delegate inside the code(so delegate Declaration will transformed into to a val declaration)
Here is an example
```
type A
  def m1(): int
  def m2(): int

var o1:A = new
  def m1(): int = 1
  def m2(): int = 2

var o2: A = new
  delegate A  to r
  def m2() : int
    return r.m2() + 1
  
```
* check if the name conflict with existing declarations
* will add a new unit test for this situation

##### 

