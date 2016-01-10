### [Q1]About the backend of Wyvern
During the past few weeks, I spent most of my time on Wyvern's frontend part including the lexer and parser. I also read the TypedAST layer and corewyvernIL. I used to work on several projects focus on compiler frontend technology so they are not quite difficult for me. But for backend, it's my first time to work on a real project. Hope you could suggest where should I start my learning on this part in our Wyvern project. A general description is very helpful, you don't need to tell the details, I will read and learn myself.

For the LLVM grammar and API, I will learn them myself. A good news is I used to learn meta-programming on .Net platform using ```Emit``` which is a technology used to generate IL code for .Net on the fly. It's very similar with LLVM in some aspects. So I might catch them up quickly! :)


### [Q2]About the module system of Wyvern
For this part, I got some concept when I reading the code. But Is there any up-to-date document or paper for it? I know I can got what I want to know from code but it takes time, hope there can be some description materials. :)


### [Q3] The Type-Specific Languages 
I'm quite interesting in the design of this part and I'm very curious about our progress on this part. Will we open API for user to help them define their interpreter for theiw own DSL more easily? Hope read more material and see more prototype on this part! 

I'm quite interesting in this area for the following reason. As I think, a complex system can have serveral quite different domains. For all the domains, it's not efficient to express them only in a general purpose language. A cheap solution is to develop an internal DSL with technology like fluent interface. But internal DSL is less flexible than external ones. It also lacks the compile time checking. With the TSL feature, we can introduce external DSLs to improve the flexibility. Another benefit by ysing TSL is very TSL can be desgined seperately, which means they works more like plugins. In traditional language development, When we want to add new feature to this language, we need to make sure our new syntax won't conflict with existing grammar. Sometimes when it's hard to reconcile the conflict, we had to change large part of the grammar to make them compatible, which means we might at least change the raw AST layer which then caused many many change. But if we decompose a big language into several plugin-like little languages, we can develop them seperately and independently! Moreover, every little languages can actually be reused and they are composable! When we develop a new language for a new domain, we don't need to do everything from start, we can reuse small DSL unit to compose it. Here Wyvern acts like a goood infrastructure framework. So the core job here as I think is we need to figure the interface between the TSL and the host language and how they interact with each other. I think I'm very interesting in designning this part. BTW, The implementation of TSL also remind me of the DLR([dynamic language runtime](https://msdn.microsoft.com/en-us/library/dd233052(VS.100).aspx)) from .Net platform. 


