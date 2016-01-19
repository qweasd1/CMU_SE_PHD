For [Q1], I used to think I can do the following change on WyvernParser.jj to support the lambda inferred
```
Object Formal() :
{ Token id; Type type = null; }
{
  id = <IDENTIFIER> [<COLON> type = Type()] {
      return build.formalArg(id.image, type);
  }
}
```

But when I see your reply, I didn't got your idea clearly. Did you mean, I shouldn't do it that way, but instead implement a new method like the following which implement the specific logic only for LambdaFn. So, there will be no affect on the DefDeclaration() and DefDeclType()
```
AST LambdaFn(ExpFlags flags) :
{ List formals; AST body; }
{
    formals = Formals_TypeOptional() <ARROW> body = Expression(flags) {
        return build.fn(formals, body);
    }
}


...

List Formals_TypeOptional() :
{ List args = new LinkedList(); }
{
  <LPAREN> [ FormalsList_TypeOptional(args) ] <RPAREN> {
			return args;
		}
}


```

Do I understand correct?
