---
title: "Making a usable Multithreaded Task System - Part 1"
description: Series of posts in which I detail how I make my own multithreaded task/job system
categories:
 - Programming
tags:
 - multithreading
 - programming
 - learning
---

Getting out of my studies I understand how to create a thread and execute a task there, as well as making an extremely basic manager that controls how that is performed with little overhead.

But that same structure does not allow to chain tasks, know when a task is done without callbacks, no internal synchronization,... Therea re a lot of things that are present in other engines and existing libraries.

I could use those libraries or not even do my own engine, but I would not be properly learning about it, so here I am.

On this first post about the task system, the base structure will be laid out to execute tasks and schedule tasks. In the conclusion, the next steps and thigns I find are missing will be explained.

---

# Introduction

The base is pretty simple, you add a task to a queue that will be executed in a thread somewhere. In order to achieve such thing there are some things that these task must check:
 * Tasks are self contained, data used in that task is only used by that task and nowhere else at the same time.
 * Tasks + Setup are expensive enough to part away, or numerous enoughh to be worth subdividing and then synchronizing the results.
 * Tasks are not prioritary, whenever we finish it we can use it but is not needed **now**.

These constraints are the ones I currently know of and the ones, but my task manager and any task manager will take care of it. It is job of the programmer designing the tasks to acknowledge when to use them and how to set them up.

## Task

~~~c++
class Task
{
    public:
    virtual void run() = 0;
}

class Example_Task : public Task
{
    // Some data that it uses and has to be setup for parallel use
    uint32_t* data;

    // Override the execution function
    void run() override
    {
        // Do somethign
    }
}
~~~

This is the simplest form of a task in my manager. You `override` an executing function that the manager will `run` on a thread when available.

## Manager

~~~c++
class TaskManager
{
    public:
    bool Init();
    bool Close();

    void ScheduleTask(Task* task);

    private:
    std::vector<std::thread*> threads;
    std::queue<Task*> scheduled_tasks;
    std::mutex _queue_mtx;
    std::condition_variable _thread_event;

    std::atomic<bool> exit_flag;
}
~~~

The basic setup to hold tasks and feed the threads slowly. The mutex will serve as a barrier to ask permission to access the queue and get a task.

The `std::conditional_variable` warns the threads that are waiting for the queue to get tasks or when it is time to close.

### Initialization

Each thread has to be initialized and be running a function that in short: asks the mutex for permission to use the queue and then execute it, if there are no tasks available, one thread will be the one blocking the mutex and waiting for a task to be available.

~~~c++
void ThreadLoop()
{
    while(true)
    {
        Task* t = 0;
        {
            std::unique_lock<std::mutex> lock(_queue_mtx);
            while(t == NULL && !exit_flag)
            {
                if(scheduled_tasks.empty())
                    _thread_event.wait(lock);
                else
                {
                    t = scheduled_tasks.front();
                    scheduled_tasks.pop();
                }
            }
        }

        if(exit_flag) break;

        t->run();
    }
}
~~~

As for the initialization, we can arbitrarily assign a number of threads that running function, but I prefer to assign as many logic threads that are available (hyperthreads / 2x cores /... depends on CPU architecture).

I have not tested at which point too many threads in relation to the logical threads is an issue, or even if there are gains to be made really. Logic tells me that as long as you map it precisely or even less (main thread and other program tasks) should be at least decent.

~~~c++
bool Tasker::Init()
{
    for(uint16_t i = 0; i < std::thread::hardware_concurrency(); ++i)
        threads.push_back(new std::thread(ThreadLoop));

    return true;
}
~~~

### Closing

We notify the threads that we are done, we make them finisht he task and close themselves and delete the threads.

~~~c++
bool Tasker::Close()
{
    bool ret = true;
    
    exit_flag = true;
    _thread_event.notify_all();
    for (uint16_t i = 0; i < threads.size(); ++i)
    {
        threads[i]->join();
        delete threads[i];
    }

    threads.clear();

    return ret;
}
~~~

### Scheduling Tasks

As we have a single queue to add and supply tasks, multipel threads will be requesting access to it. We need to ask for permission to use it, add it quickly and notify the threads that a task has been added (in case they are waiting for tasks to be feed with).

~~~c++
void Tasker::ScheduleTask(Task* task)
{
    std::unique_lock<std::mutex> qLock(_queue_mtx);
    scheduled_tasks.push(task);
    _thread_event.notify_one();
}
~~~

---

# Closing

This base is extremely barebones. The only example of practicality would be to use it on do and forget type tasks, such as any asyncrhonous action like loadign non trivial data, networking,... or pinning down threads with a recurrent task like rendering which you only have to lock access to some data structure to send request to GPU.

What I want in my task/job system is to be able to schedule them in relation to toher tasks, have recurrent tasks, know what is being done or note, make sure there are no bottlenecks when asking for tasks...

There are a ton of things that come to mind which I don't know the solution of and until I understand it, I don't feel capable of using another library that helps me with that. I don't like feeling that I do things based on unknown black magic (although most of what we use might as well be).