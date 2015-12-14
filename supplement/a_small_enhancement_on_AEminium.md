Here is a feature I thought might be useful if we add to AEminium.

### Add explicitly workflow control into AEminium
> After I read some papers about AEminium, I know it's a language which can make program parallel with compiler magic. The developer only needs to declare the data dependency and permission and the complier will optimize the program to parallel according to these information. 

But sometimes in reality, different methods might have no data dependency but the logic dependency.
Here is a scenario:
> - we have system ```A```
- system ```A``` needs to process 3 tasks: ```task1```,```task2```,```task3``` and then after **all of them finished**, it will send out an ```notification```. 

We can convert this scenario to the following code:
```csharp
void task1(){...}
void task2(){...}
void task3(){...}
void send_notification(){...}

//the main method
void main(){
  task1();
  task2();
  task3();
  
  send_notification();
}
```
The issue is actually, ```task1()```, ```task2()```, ```task3()``` share no data with ```send_notification()``` method, they are only **logic related**. So 
the AEminium might consider they can run in parallel. But we need to let ```send_notification()``` run only after the 3 tasks methods finished. So actually we need some **synchronous logic** but we don't have this in AEminium.

Of course we can use a workaround like the following:
```csharp

void task1(unique bool flag){...}
void task2(unique bool flag){...}
void task3(unique bool flag){...}
void send_notification(unique bool flag_task1, unique bool flag_task2, unique bool flag_task3){...}


//the main method
void main(){
  unique bool flag_task1 = false;
  unique bool flag_task2 = false;
  unique bool flag_task3 = false;
  
  task1(flag_task1);
  task2(flag_task2);
  task3(flag_task3);
  
  send_notification(flag_task1, flag_task2, flag_task3);
}
```
But seems it's not convenient since we introduce more variables.

Here is Some other similar cases:
> System ```B``` needs to let several tasks run one by one, but different tasks share no data, so they will paralell by compiler. (still we can have a workaround for this case if we declare a unique variable)

> System ```C``` like system ```A``` has 3 tasks but will notify a message out  **once one of 3 tasks finished**. (I can't come up with a workaround for this situation)

So I think maybe it's a good idea to include a new feature to explicity control the workflow.

Here is the structure I come up.
```csharp
// wait all tasks to finish then go on
main(){
  waitall: {
    task1();
    task2();
    task3();
  }
  send_notification();
}

// wait any tasks to finish then go on
main(){
  waitany: {
    task1();
    task2();
    task3();
  }
  send_notification();
}

// let the tasks run in synchronous
main(){
  sequence: {
    task1();
    task2();
    task3();
  }
  send_notification();
}
```
A block like the following will tell the compiler to re-calculate the schedule inside the block. The ```schedule_name``` and ```paramters``` give more hint to compiler
```csharp
<schedule_name> [parameters] :{
}
```
One step further, the structure above is actually a generic structure. So we can use it not as keyword but as a **language feature**.
In this way we can give our user the ability to express their specific schedule logic. Think about something like the following:
```csharp
//here we define a schedule which control the max parallel count
schedule limit_parallel {
 // some definition...
 ...
}

//use the defined shedule limit_parallel
main(){
  //so only 2 tasks run in parallel at a time.
  limit_parallel(2): {
    task1();
    task2();
    task3();
  }

}
```
I thought this feature can make some parallel specific design patterns easy to implemented in AEminium. And design patterns are really useful for parallel programming.  
