## Core classes

### Binding
* Environment (! think if we can refactor the create environment tree using a factory)
  * position: $Root\tools\src\wyvern\tools\types\Environment.java
  * purpose: [#] used to store the binding information (NameBinding, TypeBinding, StaticTypeBinding)
  * method:()
      * extend (#Ln34-Ln45): use extend(Binding binding), extend(Environment env), getEmptyEnvironment() to create environment

* Binding: [interace] { getName(); getType() }
  *  AbstractBinding: [abstract] (implement Binding POJO)
      * NameBindingImpl  
  *  NameBinding: [interface]  {TypedAST getUse();	Value getValue(EvaluationEnvironment env);}
  *  [?]StaticTypeBinding: use typeName:string instead of  [Type]. used for import?
  * EvaluationBinding: 
  





### Core $Root\tools\src\wyvern\tools\typedAST\interfaces
* TypedAST the root object of all AST
  * position:  TypedAST.java
  * purpose 
