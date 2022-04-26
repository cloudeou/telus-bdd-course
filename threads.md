### Multithread execution, use of threads

[![N|Solid](https://images.ctfassets.net/fikanzmkdlqn/5NoHRB1q6lrNzSSpekhrG5/cf22f3d7d9e82aed5e79659800458b57/TELUS_TAGLINE_HORIZONTAL_EN.svg)](https://www.telus.com/en/)

##### [Framework documentation](https://github.com/telus/telus-bdd-docs)

In this section you will learn how BDD framework multiple threads and parallel execution works and how to use it.

<img src="https://cdn4.iconfinder.com/data/icons/48-bubbles/48/23.Videos-512.png" width="30px" margin-top="15px"/> [Session video recording]()

#### Contents:

1. [Threads scaling mechanism](#threads-scaling-mechanism)
2. [Threads related CLI parameters](#threads-related-cli-parameters)
3. [Multiple threads usage. General](#multiple-threads-usage-general)

#### **Threads scaling mechanism**
BDD framework supports multi thread execution of automations.

Scaling happens on the level of Node.js processes, it means that every single run is executed in a separate Node.js process. We use processes scaling for 2 reasons:
- Processes are totally independent, they have their own Node.js event-loop and all the libraries initialized, which makes them more reliable.
- Processes are totally isolated from each other in terms of any accidental Node.js interactions, this helps to isolate the data properly, processes alsp have their own environment which is widely used by the framework.

Framework launch has two main steps:
1. *Lunch preparation.*

    On this step framework 
    - filters feature files to be launched by identification tags
    - parses there feature files 
    - executes database bootstrap (if needed)
    - and finally creates a launch queue with objects, describinf each run
2. *Launch execution.*

    On this step framework creates a number of child processes with first run configs from the queue (number of this processes is configured). After this each thread is independent, as soon as some processes finish, framewrok sees that it has according number of unused threads and creates according number of processes to fill these threads.
    
    When the number of remaining run configs becomes smaller than number of threads, framework kills unused threads one by one.

    This helps to use time and resources more effectively. 
#### **Threads related CLI parameters**
To regulate number of parallel threads framework has an argument `threadsNumber`, default of this argument can be specified in `bdd.config` [file](/framework-intro.md/#bddconfig-file). Also the value for it can be passed for each launch using cli parameter `--threadsNumber`.

As well `--runTimes` cli parameter, which defines how many times feature should be ran, is related to threads, threads number cannot be bigger than run times.
#### **Multiple threads usage. Example**
Multiple threads can be used when length of the queue is bigger than one. So it can be done in two cases:
- one feature is launched multiple times (run times)
- multiple fetures are executed by a shared identification tags

In both cases framework would create a queue from multiple run configs.

```
Important thing to understand that relation between threads number and total execution time is not linear, it means that increasing number of threads by 100% would give you 80-85% decrease in execution time, because environment is not perfect and consumption of more resources may result into slower work, especially if resource consumption is almost at max for the machine or pod. 
```

>Note: The thing you need to take into consideration is the quantity of the runs that would be launched and execution time of these runs (you should relatively estimate it by number of API calls etc, or practically).
>
> You have to calculate what benefit you will gain from using parallel threads, because using them in every case is not the right solution, parallel processes consume considerably bigger amount of resources.
>
> For example if you need to launch 5 fetures by shared identification tags, you know that all of the features are relatively fast (up to 20 seconds) and you know that 1 of 5 fetures takes 80% of all 5 execution time, there is no sense at all to use parallel threads, you would gain 15-17% maximum of the time benefit, which in particularly in numbers is 15-17 seconds, it is not worth consuming two times of the resources.
>
> On the other hand if you launch 3 features by shared tag, all features take relatively same considerable (2 minute) time to run, you would gain around 3.5 minutes benefit if use 3 threads, in this case it may make sense, because time difference is quite big, still, consumed resources are 3 times bigger.
>
>Or you have 10000 datasets to run in one feature, they only take 5 seconds to run, by increaing threads only up to 2 in this case, you would gain 45% faster execution, which in practically is 75 minutes!!!

So as a general rule use threads when you know it would give a considerable execution time benefit.

And for our example, we have a case where many datasets are run using one feature, we are testing product inventpry API, have 10 datasets in DB and one run executed approximately in 25 seconds. By increasing threads up to 2 we would gain more than a minute of time benefit, which is not very big, but considerable for our CI/CD pipeline, that is ran 10-15 times a day, so a command would look like.

`npm run bdd:start -- --suiteName=PIComponentTest --runTimes=10 --threadsNumber=2`
