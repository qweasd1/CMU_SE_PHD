### SQL optimizer
When I work in <conpany>. One of my job is to generate complex report for traders using SQL. The main challenge for this job is usually we need to join more than 20 tables or table functions with complex filter condition to get the final report. The embede SQL complier just can't optimize it and usually caused long running query. To get the desired performance usually we need to spend many time manually docompose the huge sql code into small pieces to find the part that slow down the whole performance and then optimize it. It was absolutely time consuming to do this, so I think we can actually made it automatically. 

The solution I gave is not difficult in design:

1. Write a parser to parse the sql query into AST
2. Use the parsed AST and statistic information(like table size) from DB to recalculate the join relationship to generate a new sql. To be more detail, If we encounter big tables with a good filter condition which can make them smaller, we will extract them out from the origin big sql into a temp table and then let the origin big sql join the temp table. At the end of this step, we will have an in-memory model for our new SQL query.
3. Finally, we just need to regenerate the new sql query from our in-memory model.

Again it looks quite easy at first, but here is the difficulty I met:

For step 1, the real challenge came from the parsing of the SQL. The SQL dialect we use is t-sql(for Microsoft's SQL Server) which has some of its unique feature. For example, it has many different ways to express the same syntax:  ```... from tableA as t ...```, ```... from tableA as "t"...```, ```... from tableA as [t]``` and ```... from tableA t``` has the same meaning. Another example is, t-sql has its unique feature to work with xml like ```select T.c.query('.') as result  from @dataSource.nodes('/root/info') as T(c) ``` which is highly used in our report. To compatible with all these special condition, it's quite challenge to implement the parser. But it's nice we already had some good tools on language development such as antlr and xtext. I use xtext since it also provide ultility classes for unit test though it's based on Antlr3 which doesn't support more powerful grammar mapping like Adaptive LL(*) which Antlr4 has.

For step 2, the real challenge came as some syntax different strucute share the same semantic meanning. For example: ```select * from t1 join t2 on t1.key = t2.key``` has the same meaning with ```select * from t1, t2 where t1.key = t2.key``` in tsql. So it took me some effort to transform the AST into a standard optimizer domain model. The algorithm using statistic data in DB to generate the execution plan is also non-trivial, so won't discuss it here since it took a lot of space.

From this project, I realized that some simple ideas can be difficult when we go down to details and implement them. Sometimes, unit test is quite important to make sure we did every step correct.
