* In TypedAST.ValDeclaration - getILValueType, shall we need to guarantee if defintion is Fn, the declType should not be null. Not sure if we still use TypedAST. When shall I took this check.
* do we also need to consider the assignment to var variable for the typeless lambda expression
* In current Wyvern's lambda implementation, does the Type used as a lambda type should only have one method with fixed name "apply"? (Not like Java that any interface with only one method can be used as a lambda type)
* In the Parser, we used Formals() both in Def's declaration and lambda expression. So when we change Formals, we also let Def's declaration open to issue with implicit argument. So we need to add checks here, right?(where shall we do this check? in generateIL or in Wyvern IL)
