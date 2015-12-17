### Core Classes

##### WyvernParser
* Parser: WyvernParser<AST,Type>
* position: $Root\C:\zw33264\Projects\academy\wyvern-master\tools\src\wyvern\tools\parsing\coreparser\WyvernParser.jj
  * [TODO] class ExpFlags: used for DSL or "new expression"
  * 
* [?>]how to set token_source for WyvernParser(line 49)?

##### ASTBuilder
* qualifiedName: wyvern.tools.parsing.coreparser.ASTBuilder
* position : $Root\tools\src\wyvern\tools\parsing\coreparser\ASTBuilder.java
* purpose: used to build ast

##### Others
* wyvern.tools.errors.FileLocation : $Root\tools\src\wyvern\tools\errors
* wyvern.tools.parsing.coreparser.WyvernTokenManager: $Root\tools\src\wyvern\tools\parsing\coreparser\WyvernTokenManager.java
  * this class is used as a token scanner(can record special token)
  

### Questions
* in $Root\tools\src\wyvern\tools\parsing\coreparser\WyvernParser.jj, how we can not define Regex for Token
