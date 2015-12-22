##### I've already finish the parser change which add a new Declaration type in WyvernParser.jj: 
```
AST TypeAbbrevDeclaration() :
{Token t = null; Type reference = null; Token alias = null; }
{
   t = <TYPE> alias = <IDENTIFIER> <EQUALS> reference = Type() <NEWLINE> {
     return build.typeAbbrevDecl(alias.image, reference, loc(t));
   }
}

...

AST Declaration(boolean inModule) :
{ AST exp; }
{
  

  exp = DefDeclaration()  { return exp; }
|
  exp = ValDeclaration()  { return exp; }
|
  exp = VarDeclaration()  { return exp; }
|
  LOOKAHEAD(2) exp = TypeAbbrevDeclaration() {return exp; }
|
  exp =   TypeDeclaration()  { return exp; }
|
  exp = DelegateDeclaration()  { return exp; }
|
  exp = ExpressionLine(inModule) { return exp; }
|
  exp = Instantiation() {return exp; }
}
```
##### I added a new Declaration Ast called TypeAbbrevDeclaration and its doExtend logic like the following:
```java
  protected Environment doExtend(Environment old, Environment against) {
		return old.extend(new TypeBinding(alias, resolveReferenceType(old)));
	}
	
	private Type resolveReferenceType(Environment env) {
		Type resolved_type = TypeResolver.resolve(reference, env);
		return resolved_type;
	}
```
What I did here is let TypeResolver reolve the real type object of the UnresolvedType and let alias bind to the realType in our ```Environment```. So when the following sequence declaration use the alias name, the can lookup the realType in ```Environment```.

##### I then added a new unit test in CoreParserTests.java
```java
@Test
	public void testTypeAbbrevDeclaration() throws ParseException {
		String input = "type i = Int\n" +
	                   "val t:i = 2\n" + 
				       "t";
		//String input = "var t = 1\n";
		
		//String input = "val t = 1\nval t2 = 2";
		TypedAST ast = TestUtil.getNewAST(input);
		Environment standardEnv = Globals.getStandardEnv();
		Type intType = ast.typecheck(standardEnv, Optional.empty());
		assertEquals(new Int(), intType);
		
	}
```
The test works fine. My trouble came when I want to let ```ILTests.testTypeAbbrev()```. I check the logic and find we don't call ```tpyecheck(...)``` method on ```TypedAst``` here. What we did is call the ```generateIL()``` here. 

I want to use the same solution I used in ```TypeAST.typecheck()```, however, the environment object we work with here is the ```GenContext``` and its subType ```TypeGenContext``` which works as a type binder looks strange to me. I hope it can have a function like ```CoreWyvernIL.ValueType lookupType(string typeName)```. But seems it doesn't have.

Any suggestion here?
