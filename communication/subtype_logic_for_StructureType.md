### The logic of ```isSubtypeOf(...)``` of ```wyvern.target.corewyvernIL.type.StructuralType```
When I reading the logic of ```isSubtypeOf(...)``` of ```wyvern.target.corewyvernIL.type.StructuralType```, I found the logic is a little confusion for me. Not sure if it's my misunderstanding or the there are mistake in the code. Hope you could help guide on this.

The core logic for the ```isSubtypeOf`` method is as following:
```java
    for (DeclType dt : st.declTypes) {
			DeclType candidateDT = findDecl(dt.getName(), ctx);
			if (candidateDT == null || !candidateDT.isSubtypeOf(dt, extendedCtx)) {
				return false;
			}
		}
```
And the final result is rely on ```DeclType```'s ```isSubtypeOf(...)``` method. And in ```wyvern.target.corewyvernIL.decltype.DefDeclType```'s ```isSubtypeOf(...)``` logic as the following
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
we can see for both arguments' type and result's type we use ```isSubtypeOf(...)``` method. Here I thought it might have some issue.
Considering the following example:
```

```

