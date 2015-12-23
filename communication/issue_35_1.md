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
##### I added a new Declaration Ast called ```TypeAbbrevDeclaration``` and its ```doExtend(...)``` logic like the following:
```java
  protected Environment doExtend(Environment old, Environment against) {
		return old.extend(new TypeBinding(alias, resolveReferenceType(old)));
	}
	
	private Type resolveReferenceType(Environment env) {
		Type resolved_type = TypeResolver.resolve(reference, env);
		return resolved_type;
	}
```
What I did here is let TypeResolver reolve the real type object of the UnresolvedType and let alias bind to the realType in our ```Environment```. So when the following sequence declaration use the alias type name, they can lookup the realType in ```Environment```.

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
The test works fine. My trouble came when I want to let ```ILTests.testTypeAbbrev()``` pass. I check the logic and find we don't call ```tpyecheck(...)``` method on ```TypedAst``` here. What we did is call the ```generateIL(...)``` here. 

I want to use the same solution I used before. The environment object we work with here is the ```GenContext```, so I want to let ```TypeAbbrevDeclaration``` add the binding information into it. However this seems not work since the  ```TypeGenContext``` doesn't have any method like ```CoreWyvernIL.ValueType lookupType(string typeName)```. What it restores is a ```objName``` and ```typename```.

I think I can finally get it work in my way, but it will definitely change a lot of code and destroy the current design. Do I misunderstand the design, any suggestion?



##### more analysis
I also try when I remove the type alias in unit test ```ILTests.testTypeAbbrev()``` like the following:
```java
@Category(CurrentlyBroken.class)
	public void testTypeAbbrev() throws ParseException {
		String input = "val i : system.Int = 5\n\n" +
			       "i\n"
				     ;
		TypedAST ast = TestUtil.getNewAST(input);
		// bogus "system" entry, but makes the text work for now
		GenContext genCtx = GenContext.empty().extend("system", new Variable("system"), null);
		Expression program = ast.generateIL(genCtx);
    	TypeContext ctx = TypeContext.empty();
		ValueType t = program.typeCheck(ctx);
		Assert.assertEquals(Util.intType(), t);
		Value v = program.interpret(EvalContext.empty());
    	IntegerLiteral five = new IntegerLiteral(5);
		Assert.assertEquals(five, v);
	}
```
though it can work, but when I debug it, I saw inside class ```ValDeclaration```, the logic here doesn't set ```declaredType``` since the parameter ```type``` is not ```UnresolvedType```. It's a ```QualifiedType```
```java
//inside wyvern.tools.typedAST.core.declarations.ValDeclaration
public ValDeclaration(String name, Type type, TypedAST definition, FileLocation location) {
		if (type instanceof UnresolvedType) {
			UnresolvedType t = (UnresolvedType) type;
			
			// System.out.println("t = " + t);
			
			TaggedInfo tag = TaggedInfo.lookupTagByType(t); // FIXME:
			
			// System.out.println("tag = " + tag);
			
			// if (tag != null) {
				//doing a tagged type
				ti = tag;
				
				variableName = name;
				
				declaredType = type; // Record this.
				
				//type = null;
			// }

		}
		
		this.definition=definition;
		binding = new NameBindingImpl(name, type);
		this.location = location;
	}
```

so, when we generate corewyvernIL's AST from TypedAST for the statement ```val i : system.Int = 5```, we go into the second branch in the following branch:
```java
//inside wyvern.tools.typedAST.core.declarations.ValDeclaration
private ValueType getILValueType(GenContext ctx) {
		ValueType vt;
		if (declaredType != null) {
		
			vt = declaredType.getILType(ctx);
		} else {
			// [here]: we go this branch
			vt = definition.generateIL(ctx).typeCheck(ctx);
		}
		return vt;
	}
```
this means, we actually ignore the declaredType totally and use the definition to inference the corewyvernIL type here. which means we ignore the ```system.Int``` inside the  ```val i : system.Int = 5```. This makes me confused...
