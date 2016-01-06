### My plan on this issue
##### for ```wyvern.tools.typedAST.core.declarations.DelegateDeclaration```
________________
* leave method ```genILType(...)``` alone, since DelegateDeclaration actually won't represent a concrete declaration
* implement ```generateDecl(...)``` method to generate ```wyvern.target.corewyvernIL.decl.delegateDeclaration```
* leave method ```topLevelGen(...)``` alone, since it won't be used

##### for ```wyvern.target.corewyvernIL.expression.ObjectValue```, add logic for delegate
________________
* if the ObjectValue has delegate target, here is my plan to resolve the method-invoke and field-get. If the method/field define only in target object or the new object, we use it. If the method/field define both in target object and the new object we use the definition in new object.
* I will assume we only consider single delegate here


##### for ```wyvern.target.corewyvernIL.decl.delegateDeclaration```
_________________
* check the logic **delegate target objcet** fullfill all declarations in the **delegate type**. Which means **delegate target objcet**'s ```structuralType``` should be subType of **delegate type**'s ```structuralType```.


##### for ```wyvern.target.corewyvernIL.expression.New```
* modify method ```typeCheck(...)``` to let it compatible with delegate declaration
* modify method ```interpret(...)``` to let it return the new version of ```wyvern.target.corewyvernIL.expression.ObjectValue```
