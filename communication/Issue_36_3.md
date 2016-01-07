### Detail Plan

##### Single Delegation in this iteration
For multi-delegate, we need to consider more on method conflict(same situation in Scala's multi-trait). Though they are not difficult to implement, as I think, it needs to think carefully for every details for the whole function. So won't introduce the complexity in this iteration.

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

