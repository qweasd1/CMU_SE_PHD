## Stategy
### How TypedAST generated? (the process sequence)



## Types
* Type: Root
  * subclasses:
      * Unit: NULLObject of Type 



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
  

### expression: $Root\tools\src\wyvern\tools\typedAST\core\expressions



## Basic Interfaces:  $Root\tools\src\wyvern\tools\typedAST\interfaces
* ```TypedAST```: the root object of all AST
  * position:  TypedAST.java
  * [?>]purpose : typecheck(Environment env, Optional<Type> expected), getType(), Expression generateIL(GenContext ctx), CallableExprGenerator getCallableExpr(GenContext ctx);
  * methods: 
      * Expression generateIL(GenContext ctx) : generate wyvernIL code from current AST 
  * subclasses: 
      * ```CoreAst```: just can accept CoreASTVisitor
      * 
* ```Value``` : just a sub domain root
* ```Sequence``` extends AbstractTypedAST implements CoreAST, Iterable<TypedAST> : 
  * purpose: used to create the sequence 

