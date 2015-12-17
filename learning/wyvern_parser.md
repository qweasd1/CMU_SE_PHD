### Core Classes

##### WyvernParser
* Parser: WyvernParser<AST,Type>
* position: $Root\C:\zw33264\Projects\academy\wyvern-master\tools\src\wyvern\tools\parsing\coreparser\WyvernParser.jj
  * [TODO] class ExpFlags: used for DSL or "new expression"
  * 
* [?>]how to set token_source for WyvernParser(line 49)?
* functions:
  * freshName(): create anonymous variable name

##### ASTBuilder
* qualifiedName: wyvern.tools.parsing.coreparser.ASTBuilder
* position : $Root\tools\src\wyvern\tools\parsing\coreparser\ASTBuilder.java
* purpose: used to build ast
* [?>]: what does tagInfo(...) mean?
* [?>]: what does setNewBody(...) mean? | build.setNewBody(flags.getNewExp(), decls) // #ln:376

##### Others
* wyvern.tools.errors.FileLocation : $Root\tools\src\wyvern\tools\errors
* wyvern.tools.parsing.coreparser.WyvernTokenManager: $Root\tools\src\wyvern\tools\parsing\coreparser\WyvernTokenManager.java
  * this class is used as a token scanner(can record special token)
  
### Grammar

##### need to understand
* [?]TAGGED: is it related to pro jonathan's paper? 

##### components
* Expression:
* ExpressionLine: a wrapper for Expression and pattern match
  * normal expression: <expr> + sub expr
  * pattern match (no default, only match type by now)
* Declaration : the unit of Wyvern language
  * DefDeclaration : method definition
  * ValDeclaration : val declaration
  * VarDeclaration : var declaration
  * [?<]DelegateDeclaration: delegate <type> to <expressionline>  
  * TypeDeclaration: type(class)
* DeclType: DefDeclType | ValDeclType
### Questions
* in $Root\tools\src\wyvern\tools\parsing\coreparser\WyvernParser.jj, how we can not define Regex for Token
