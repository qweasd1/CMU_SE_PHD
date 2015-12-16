### Notes
* quote language: $root\tools\src\wyvern\tools\parsing\quotelang



* lexerGrammar: $root\tools\src\wyvern\tools\lexing\WyvernLexer.x
  * [TODO] find the facade method of it
  * [TODO] find a general method to read this file
  * [?] Is there any graph tool to show Copper grammar to graph?
  * [?] how do we handle the following left-recursive grammar?
  ```
  lineElementSequence ::= indent_t:n {: RESULT = makeList(n); :}
	                      | nonWSLineElement:n {:
	                            // handles lines that start without any indent
								if (DSLNext)
									throw new CopperParserException("Indicated DSL with ~ but then did not indent");
	                      		RESULT = n;
	                        :}
	                      | lineElementSequence:list anyLineElement:n {: list.addAll(n); RESULT = list; :};
  ```
  * [Mark] LexerUtils.isSpecial(..): $root\tools\src\wyvern\tools\lexing\LexerUtils.java
  
### Task
* CopperComposer: $Root\tools\copper-composer
  * [TODO| copper's Java API]: what does it do? (compose host language with extension language ? see details)
  * [main class] : ParserBean(is it a in-memory representation for a Copper parser meta data?)

