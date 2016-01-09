### The logic of ```isSubtypeOf(...)``` of ```wyvern.target.corewyvernIL.type.StructuralType```
When I reading the logic of ```isSubtypeOf(...)``` of ```wyvern.target.corewyvernIL.type.StructuralType```, I found the logic is a little confusion for me. Not sure if it's my misunderstanding or there are mistake in the code. Hope you could help guide on this.

The core logic for the ```isSubtypeOf``` method of ```wyvern.target.corewyvernIL.type.StructuralType``` is as following:
```java
    StructuralType st = (StructuralType) t;
		st = (StructuralType) st.adapt(new ReceiverView(new Variable(st.selfName), new Variable(selfName)));
		
		TypeContext extendedCtx = ctx.extend(selfName, this);
    for (DeclType dt : st.declTypes) {
			DeclType candidateDT = findDecl(dt.getName(), ctx);
			if (candidateDT == null || !candidateDT.isSubtypeOf(dt, extendedCtx)) {
				return false;
			}
		}
```
And the final result is rely on ```DeclType```'s ```isSubtypeOf(...)``` method. In ```wyvern.target.corewyvernIL.decltype.DefDeclType```'s ```isSubtypeOf(...)``` method:
```java
public boolean isSubtypeOf(DeclType dt, TypeContext ctx) {
		if (!(dt instanceof DefDeclType)) {
			return false;
		}
		DefDeclType ddt = (DefDeclType) dt;
		if (args.size() != ddt.args.size())
			return false;
		for (int i = 0; i < args.size(); ++i) {
			if (! (ddt.args.get(i).getType().isSubtypeOf(args.get(i).getType(), ctx))) {
				return false;
			}
		}
		return ddt.getName().equals(getName())
			&& this.getRawResultType().isSubtypeOf(ddt.getRawResultType(), ctx);
	}
```
we can see for both arguments' type and result's type we use ```isSubtypeOf(...)``` method. As I think, it might have some issue if we do it this way.
Considering the following example:
```
type A
  def m1():int
  
type B
  def m1():int
  def m2():int
  
// here type B's structualType should be the subType of type A's structuralType

type C
  def m(p:A):int
  
type D
  def m(p:B):int
  
// here type D's structuralType should also be the subType of type C's structuralType

// now if we assign a objectValue of type D to a variable declared as type C as following
val o1:D = new 
  def m(p:B):int
    // here we use m2() method
     return p.m2()

val o2:C = o1
// it can pass our typecheck. However, when we called method m(...) on o2, from the type definition of type A, we should use the signature m(p:A), but actually we call the m(p:B):int define in o1. When we  we pass in object of type A as following

var _p:A = new
  def m1():int
    return 1
o2.m(_p)

//we will get an error since _p doesn't have a method defined as m2 which is needed in o1's m(p:B)'s definition
```
Any suggestions on this?
