### Notation
* [?]: question
* [?<]: minor question
* [TODO]
* [TODO>]: important TODO
* [Mark]: some information catch attenson


### Notes
* quote language: $root\tools\src\wyvern\tools\parsing\quotelang



* lexerGrammar: $root\tools\src\wyvern\tools\lexing\WyvernLexer.x
  * [TODO] find the facade method of it
  * [TODO] find a general method to read this file
  * [?] Is there any graph tool to show Copper grammar to graph?
  * [?] how do we handle the following left-recursive grammar? (<start> (<next>)* ?)
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
  * [Mark] LexerUtils.isSpecial(..): $root\tools\src\wyvern\tools\lexing\LexerUtils.java  | return true when the token is SINGLE_LINE_COMMENT, MULTI_LINE_COMMENT, WHITESPACE
  * [Mark] Token definition namespace: wyvern.tools.parsing.coreparser.WyvernParserConstants
  * [?] Token.image : what's this mean?
  * [?] program: what does the incomplete line mean in #branch2? (to compatible with no newline case?)
```
program ::= lines:p {:
	             	RESULT = p;
	             	Token t = ((LinkedList<Token>)p).getLast();
	             	RESULT.addAll(possibleDedentList(t));
	           	:}
	          | lines:p lineElementSequence:list {:
	          		//#branch2 handle the case of ending in an incomplete line
	          		RESULT = p;
	          		List<Token> adjustedList = adjustLogicalLine((LinkedList<Token>)list);
	          		RESULT.addAll(adjustedList);
	             	Token t = ((LinkedList<Token>)adjustedList).getLast();
	          		RESULT.addAll(possibleDedentList(t));
	          	:}
	          | lineElementSequence:list {: RESULT = list; :} // a single-line program with no carriage return
	          | /* empty program */ {: RESULT = emptyList(); :};
```
  * [?<] lexerGrammar: adjustIndent in ident bracn, what's the meaning of following:
```
if (newIndent.startsWith(currentIndent)) {
				indents.push(newIndent);
				return 1;
			} else {
				throw new CopperParserException("Illegal indent at line "+tokenLoc.beginLine+": not a superset of previous indent level");
			}
```
  * [TODO>]: write a digram of the lexerGrammar
  * 
  
### Task
* CopperComposer: $Root\tools\copper-composer
  * [TODO| copper's Java API]: what does it do? (compose host language with extension language ? see details)
  * [main class] : ParserBean(is it a in-memory representation for a Copper parser meta data?)

### Token Reference
* ARROW : "=>"
* TARROW: "->"
* POUND: "#"
* DASH: '-'
