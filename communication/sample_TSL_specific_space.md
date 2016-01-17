suppose we have a TSL module A which provides convenient way to format the datetime. We can write the format syntax as following
```
//format datetime variable t in format: yyyy-MM-dd
t | yyyy-MM-dd

```
And in our host module B, we need to use the format function many time as the following:
```
// in Module B
val t1 ...
val t2 ...
...


m1(~)
  t1 | yyyy-MM-dd

m2(~)
  t2 | yyyy-MM-dd
...

```

Soon the issue came, the 'yyyy-MM-dd' format duplicates many time in our code. It's better we can have a function called ```render``` which encapsulate our logic for date formating like the following:
```
def render(datetime:DateTime ): String
  datetime.toString("yyyy-MM-dd")
  
// then we can replace our code in module B with the following 
...
m1(~)
  t1 | render

m2(~)
  t2 | render
...

```
To use it this way, we need a module to contains our ```render``` method. This module needs to be load before we parse the application module which rely's on it. I call the space holds the reusable logic from this module, TSL-specific space. Any TSL in application module, can refer to the reusable logic defined in this module without any import statement 
