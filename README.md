Download Link: https://assignmentchef.com/product/solved-spl181-assignment-2
<br>
In the following assignment you are required to implement an Actor ThreadPool, and to use it to build a University Management System. In Actor thread pool, each actor has a queue of actions. One can submit new actions for actors and the pool makes sure that each actor will run its actions in the order they were received, while not blocking other threads. The threads in the pool are assigned dynamically to the actors. As an SPL181 team you will implement an Actor Thread Pool.

It is very important to read the entire work before starting. Do not be lazy here, the work will be much easier if you read and understand the entire work in advance.

<h1>1           Part 1: Actor Thread Pool</h1>

<h2>1.1           Detailed Description</h2>

In actor thread pool, each actor has a queue of actions. One can submit a new action to the actor. The threads in the actor thread pool are assigned dynamically to the actors, in the following way. Each thread searches for an action to execute in all the actors’ queues. Once the thread found such an action, it will prevent any other thread from fetching actions from that queue. Once it nished executing the action, it will allow other threads to fetch actions from this queue. And that thread will try to nd another action to execute. Note, although a thread prevents other threads from processing actions from the queue which it is executing an action from it, the other threads are not blocked. The threads have to search for an action from other queues, and only if all the queues are empty or not available (threads work on them) then the threads will go sleep and should wake up once an action from an available queue is ready to be fetched.

An important observation, in Actor Thread Pool, the amount of actors can be signi cantly greater than the amount of threads. See gure 2.

3.1.1             Design Pattern: Event Loop

In this assignment you will implement the Event Loop design pattern. In such pattern, each thread in the Thread Pool has a loop. In each iteration of the loop, the thread tries to fetch an action and execute it. An important point in such design pattern is not to block the thread for a long time, unless there is no work for it. Blocking the thread for a long time while there is a work for it will have a bad e ect in your implementation, since threads are idle although there is work for them. See gure 3.

<h2>1.2           Actors and Actions</h2>

An action is a computational task. An actor is a computational entity that in response to actions it receives, an actor can: make local decisions, create more actors, send message to other actors. Actors may modify private state, but can only a ect each other through messages.

In this assignment, the messages between actors are actions. An actor can submit an action to another actor’s queue. A thread will fetch this action from the receiver queue and execute it. Note, that actors submit actions to another actor’s queue, when that action may a ect the private state of the receiver’s queue, and by that avoiding locks since only one thread can access the private state of an actor at the same time.

You must not synchronize on the state of the actor, but you can use the state for the implementation of your Actor Thread Pool.

(a) Two Actors and Two Threads (b) Thread 1 fetches action 1 from ac- (c) Action 1 has been completed. There(Threads), actor 1 has two actions in his tor 1 and executes it. Thread 2 can not fore, the queue of actor 1 is available for

queue                                                                  fetch actions from actor 1 queue                   both threads

(d) Thread 2 fetches action 2 from actor

1 and executes it

Figure 2: Actor Thread Pool Description

<h2>1.3           Dependency Between Actions and Using Promises</h2>

In some applications the actions have interdependence constraints. Your thread pool should ful ll these constraints. The actions execution should be suspended until the actions it depends on are completed. Suspending an action should not suspend the thread or the actor. When an action is suspended, the thread should continue with another action, and the actor’s queue should be available for fetching actions by any thread. The suspended action should be eventually continued only when all actions it waits for them are done. The approach to handle this is to enqueue the continuation of the resumed action on the same actor’s queue whenever it is ready to be continued. In some point, one thread will fetch the continuation and execute it.

3.3.1           Promise Design Pattern

In order to enable the mentioned above. We use the promise design pattern (Using the Promise class you will implement in this assignment). A Promise is used for deferred computations and represents the result of an operation that has not been completed yet. Each action has a Promise Object which hold its result, whenever an action needs a result of another action, it passes to the action’s Promise object a callback. This callback will be executed once the event is completed. Here is a motivation for using Promises. Suppose you want to execute an action which takes time, such as multiplication of a two matrices, <em>A,B</em>. You may write this code statement (assume you have Mult function):

1             result = Mult(A,B);

Figure 3: Event Loop

As Mult is a heavy operation, assume it takes 10 seconds to complete. Meanwhile, your thread is blocked from doing another tasks. One might want that the threads do other work while waiting for the result of Mult. This is exactly the goal of Promise. With Promise you can write something like this:

<ul>

 <li>promise = Mult(A,B);</li>

 <li>then ( promise , ()−&gt;{print ( result )});</li>

 <li>int simple = 9−5;</li>

</ul>

In this example, instead of the actual result of Mult, you get a promise that you will get the actual result once Mult is completed. Now, Mult does not block your thread, the thread simply gets the promise and gets back. After the thread received the promise, it specify what to do with the result when it is ready (by passing a callback to then method). And continue working on other work. When the result is ready, the callback passed to then is executed.

<h2>1.4           Example of Actor Thread Pool</h2>

In order to explain how Actor Thread Pool works, assume that we want to write a program to manage banks. Each bank has a list of customers, where customers can transfer money to customers in di erent accounts. The goal is that banks do not block each other.

In actor thread pool, we can think about each one of the banks as actor. We submit actions for each bank. The threads in the thread pool will search for an available action in one of the banks and execute it while preventing other threads from executing an action of the same bank, that ensures that actions of the same bank are executed in the order the were received. Lets consider the action of transferring money from a customer A in bank 1 to customer B in bank 2. This action involves the authorization of both banks, updating the customers records and a receipt for both customers. Therefore, in the implementation of such action, bank 1 inserts a new action for bank 2 asking for con rmation for the transferring and updating customer B records. Bank 1 gets a promise from bank 2 to return him an answer. Meanwhile, bank 1 will continue handling other transactions for his customers. Once the action sent by bank 1 to bank 2 is completed, bank 2 will resolve the promise given to bank 1. Once the promise is resolved, the main action will be continued by inserting it again the queue of bank 1. In some point later, the action is fetched again, however, during the second execution the action should not be handled from scratch, and only a continuation of the action should be executed. The continuation will be executed and the money will be transferred from customer A to B. (see section 3.7 for code example)

<h2>1.5           Implementation Of Actor Thread Pool</h2>

In this part of work you are supplied with 4 classes that you will have to implement in order to construct a working actor thread pool. All of the classes are stored in the bgu.spl.a2 package. You are advised to download and read the javadoc of the supplied interfaces as a complementary material to this section. The rest of this section describes the given classes and the way they should be implemented.

3.5.1                    The Actor Thread Pool Components: Actors and Threads

To this end, in our framework, each Actor has its own action’s queue. Each actor has own id which is a String, and a PrivateState object. The Actor Thread Pool holds all the queues of all the actors in our system. The submit method of the the Actor thread pool, receives as parameter, an action (or list of actions) along with corresponding actor. If the actor is present in the thread pool, it enqueues the action to the queue. Otherwise, if is not present, it will create the actor’s queue and submit the action to it.

The actor thread pool maintains also a set of threads. The thread mission is to execute the actions submitted to the actor. An important observation in this matter, is that the thread number can be too smaller from the number of actors. Therefore, one can not assign a thread per each actor. Rather, the assigning of actors to thread is done dynamically. That is, a thread searches for an action to execute in all the actors queue. Once the thread found such an action, it will prevent any other thread from executing tasks of the same actor.

3.5.2          Actions and Promises

The queue of each actor holds objects of type Action. The Action is an abstract class, where each sub class has to de ne the method start only, which describes the behavior of the action. Each Action holds an object of type Promise which represents its deferred result. That is, this object will hold eventually the result of the action when it completes its computation. A Promise object holds a list of callbacks that will be executed when the Promise object is resolved (.i.e when it holds the actual result).

The ow is as follows, a thread fetches an action from a queue. The thread calls the handle method of the action, which in turn calls either the start method of the action or a callback we refer as continuation to continue the execution of the actor after it gets resumed.

As we mentioned, some actions might need the help of other actors in order to continue their computation. For this matter, that action will send a message to the other actor by sendMessage method. This methods gets as parameter the message which is of type Action, and the receiver actor. It will simply enqueue the message (action) to the other actor’s queue (and if it is not present, will create it). By sending the message, the actor receives a Promise object from the receiver, this Promise will hold the result of the sent action in some point later. After the actor has sent all the messages, the action will be suspended for waiting on the sent actions. The actor proceeds with other actions on its queue. When all the actions it waits on are completed, the suspended action will be inserted again to the actor’s queue. At some point, the thread will execute the suspended action again, and it actually will execute the continuation of the action.

3.5.3                Summary and Additional Clari cations

The following is a summary and additional clari cations about the implementation of di erent parts of the framework.

IMPORTANT: It is mandatory to read the all javadocs of the interfaces carefully, as these provides necessary hints of the implementation. You can add methods and elds to the classes. But you are NOT allowed to remove or change any method signature that we provided you with. Otherwise you will fail our automatic tests. Notice that you still can add the “synchronized” keyword to some methods if needed. However, remember that In concurrent programming, you should try to nd good ways to avoid blocking threads as much as possible. This also applies to this assignment.

(a) Two actors, and two Threads. Actor (b) Thread 1 fetches Action 1 from actor (c) Action 1 sent a message Action 1.1

1 has three actions                                                    1 and executes it. Thread 2 is prevented to Actor 2, and waits for its result by

from fetching action from actor 1               Action 1.1 promise object. The Threads can fetch actions from both actors.

(d) Thread 1 fetches Action 1.1 from Ac- (e) Thread 1 completes Action 1.1. The tor 2, and Thread 2 fetches action 2 from Promise object of Action 1.1 is resolved,

Actor 1                                                                and a callback sent by Action 1 is called.

Therefore, Action 1 is inserted again to actor’s 1 queue

Figure 4: Task life cycle – a sample of class interaction within the Actor Thread Pool .

Action: an abstract class that represents an action. An action is an object which holds the required information to handle an action in the system. The action also holds a Promise&lt;R&gt; object which will hold its result. The main action methods are:

start: This is the abstract method, that should be implemented for each action type, it implements the action behavior.

sendMessage: This methods submit an action (message) to other actor.

then: add a callback to be executed once all the given action are completed.

complete: resolves the internal result – should be called by the action derivative once it is done.

getResult: returns the task promised result.

Promise: this class represents a promise result i.e., an object that eventually will be resolved to hold a result of some operation, the class allows for getting the result once it is available and registering a callback that will be called once the result is available. Promise includes these methods:

get: return the resolved value if such exists.

resolve: called upon completing the operation, it sets the result of the operation to a new value, and trigger all the subscribed callbacks.

subscribe: add a callback to be called when this object is resolved if while calling this method the object is already resolved – the callback should be called immediately.

isResolved: return true if this object has been resolved.

VersionMonitor: Describes a monitor that supports the concept of versioning – its idea is simple, the monitor has a version number which you can receive via the method getVersion() once you have a version number, you can call await() with this version number in order to wait until this version number changes. You can also increment the version number by one using the inc() method.

ActorThreadPool: manages all the the actor and threads in our system. The constructor of ActorThreadPool creates an ActorThreadPool which has <em>n </em>threads. The ActorThreadPool includes these methods:

submit: Enqueues an action to an actor’s queue. If the actor is not present in the system, it will create it. start: start the threads belongs to this thread pool.

shutdown: closes the thread pool – this method interrupts all the threads and wait for them to stop – it returns only when there are no live threads in the queue. After calling this method, one should not use the queues anymore

<h2>1.6           JUnit Tests for The Actor Thread Pool</h2>

When building a framework, one should change the way they think. Instead of thinking like programmer which writes software for end users, they should now think like a programmer writing a software for other programmers. Those other programmers will use this framework in order to build their own applications. For this part of the assignment you will build a framework (write code for other programmers), the programmer which will use your code in order to develop its application will be the future you while he works on the second part of this assignment.

Therefore, Before your class implementations are used in a production environment, you will have to verify that they work as expected. In order to do so, you must write unit tests for your classes. These unit tests should be based on the documentation of the classes, without being related on the implementation. After you implement the classes, you can run the unit test and check that your implementation is correct. An example of unit test is for sendMessage method in the Action class. This method should submit an action to other actor, but it must not wait for it (if the action needs to wait, the waiting should be done in the start method). You have to submit the unit tests alongside your project. Take care to write good tests.

Listing 1: Example of a Unit Test for resolve method in Promise

<ul>

 <li>@Test</li>

 <li>public void        testResolve (){</li>

 <li>try{</li>

 <li>Promise&lt;Integer&gt; p = new Promise &lt;&gt;();</li>

 <li>resolve (5);</li>

 <li>try{</li>

 <li>resolve (6);</li>

 <li>Assert . f a i l ();</li>

 <li>}</li>

 <li>catch ( IllegalStateException ex){</li>

 <li>int x = p. get ();</li>

 <li>assertEquals (x ,5);</li>

 <li>}</li>

 <li>catch ( Exception ex){</li>

 <li>Assert . f a i l ();</li>

 <li>}</li>

 <li>}</li>

 <li>catch ( Exception ex){</li>

 <li>Assert . f a i l ();</li>

 <li>} 21 }</li>

</ul>

In Unit Tests we also check that the method throws the expected exception. For example, resolve throws IllegalStateException if we try to resolve an already resolved promise.

You need to write Unit Tests for Promise and VersionMonitor classes and submit them by 07.12.17. Here are some instructions. Notice: In Maven the unit tests are placed in src/test/java – this considered the same package as: bgu.spl.a2 These are the steps:

<ol>

 <li>Download the interfaces</li>

 <li>Extract them</li>

 <li>Import Maven Project in Eclipse</li>

 <li>To add a test for class X, right click on the class le X, add -&gt; new -&gt; JUNIT Test Case, and place it in src/test/java</li>

</ol>

Submit all your package, with all the classes.

The package hierarchy as speci ed in section 5.1. Make sure you compile with Maven.

<h2>1.7           A Code Example</h2>

The following simple example clari es the usage of the framework to implement a system of banks. Each bank has an actor. The Transmission action is an action submitted for the actor of the sender bank in order to transfer an amount of money from its customer to another customer in di erent bank. In the start method, the bank sends a message to the second banks asking for con rmation, since the con rmation involves access to the private state of the customer in the other bank. Therefore, it is more e cient to ask the other bank to do this work (so we do not need locks, why?). The message is actually submitting an action to actor of the second bank. In then method, we wait until we receive an answer, and then call complete to resolve the result.

Note: You MUST NOT wait for the actions in sendMessage, since it is not necessary that the action waits for the actions it sends to the other actors.

In the Con rmation action we actually do some checks and decide on the answer.

First, in the main method we start the thread pool and submit actions. The following is an example of a simple main method which submits one action to the Thread Pool,

3.7.1           Simple Main Method

Listing 2: Submitting Transmission action to the Actor Thread Pool

<ul>

 <li>ActorThreadPool pool = new ActorThreadPool (8);</li>

 <li>Action&lt;String&gt; trans = new Transmission (100 , “A” ,”B”                , “bank2” , “bank1” );</li>

 <li>pool . start ();</li>

 <li>pool . submit( trans , “bank1” , new BankStates ());</li>

 <li>CountDownLatch l = new CountDownLatch (1);</li>

 <li>trans . getResult (). subscribe (() −&gt; {</li>

 <li>l . countDown ();</li>

 <li>});</li>

 <li>l . await ();</li>

 <li>pool . shutdown ();

  <ol>

   <li>In line 1 we create a thread pool with 8 threads.</li>

   <li>In line 2 we create an action of type Transmission. Transferring 100 from “A” in “bank1” to “B” in “bank2”.</li>

   <li>In line 3 we start the thread pool.</li>

   <li>In line 4 we submit the action to the thread pool. We pass the id of the actor of bank1. Also, in order for the thread pool to maintain a Private State object for “bank1” actor, we send an object of Private State (here we send BankState which extends PrivateState). We send this object as indicator of the Thread Pool of the the type of the actor. If the actor is already in the Pool, this object should be ignored. That is, the thread pool should hold only one PrivateState object per actor which is the one received when we submit an action for the actor for the rst time. (Although we send an object each time we submit an action to the actor, these redundant objects are to be ignored.).</li>

   <li>Note that you need to wait for the action to complete before existing the program. To achieve this we use CountDownLatch. Thus, we exit only when are sure that that action is done.</li>

  </ol></li>

</ul>

Next is a possible implementation of the Transmission action.

3.7.2                Transmission Action – Basic Implementation

<ul>

 <li>public class        Transmission extends Action&lt;String &gt;{</li>

 <li>int amount ;</li>

 <li>String sender ;</li>

 <li>String receiver ;</li>

 <li>String receiverBank ;</li>

</ul>

6

<ul>

 <li>public Transmission ( int amount ,               String      receiver ,               String      sender ,</li>

 <li>String receiverBank ,      String senderBank) {</li>

 <li>this . senderBank = senderBank ;</li>

 <li>this . sender = sender ;</li>

 <li>this . receiver = receiver ;</li>

 <li>this . amount = amount ;</li>

 <li>this . receiverBank = receiverBank ;</li>

 <li>}</li>

</ul>

15

<ul>

 <li>@Override</li>

 <li>protected void        start () {</li>

 <li>List&lt;Action&lt;Boolean&gt;&gt; actions = new ArrayList &lt;&gt;();</li>

 <li>Action&lt;Boolean&gt; confAction = new Confirmation ( sender , receiver ,</li>

 <li>receiverBank , new BankStates ());</li>

 <li>actions . add( confAction );</li>

 <li>sendMessage( confAction , receiverBank , new BankStates ());</li>

 <li>then ( actions , () −&gt;{</li>

 <li>Boolean result = actions . get (0). getResult (). get ();</li>

 <li>i f ( result == true ){</li>

 <li>complete (“transmission succeed” );</li>

 <li>System . out . println (“transmission succeed” );</li>

 <li>}</li>

 <li>else {</li>

 <li>complete (“transmission failed ” );</li>

 <li>System . out . println (“transmission failed ” );</li>

 <li>}</li>

</ul>

33

<ul>

 <li>});</li>

 <li>}</li>

 <li>}

  <ol>

   <li>line 1. By Action&lt;String&gt; we de ne the result type of this action to be String.</li>

   <li>line 17. We override the abstract start method. Here we place the behavior of the action.</li>

   <li>line 19. We create another action of type Con rmation (we do not show in this document). This action will be submitted for “bank2″’s actor, since it requires access to its private state.</li>

   <li>line 22. In sendMessage we put the action (which is the message) in “bank2″’s actor queue.</li>

   <li>line 23. We announce that we are interested in each result of the actions in the list to proceed.</li>

   <li>line 24-32. This is the continuation. When the result of the action is resolved, the action will be inserted again to the “bank1″’s actor, this continuation will be run in some point.</li>

  </ol></li>

</ul>

<h2>1.8           Testing the Actor Thread Pool</h2>

Before you hurry into implementing the University Management, you are advised to test your implementation of the Actor Thread Pool. To achieve this you might complete the implementation of the Banks System. Complete the Con rmation action so it does some non trivial work. Submit actions and check it does the work.

<h1>2           Part 2: University Management System</h1>

In this section you will simulate a University Management System using your Actor Thread Pool framework from part 1. The university has a set of departments, each department o ers a set of courses to students (of all the departments) and each student can register for courses if he meets the prerequisites and there is an available space for him. The register/unregister request are considered until the end of the registration period. In this System we de ne Actors and their Private States. It is obligatory to comply to our de nitions without any change.

In this section you are not allowed to use any type of synchronization on the Actors private state. You have to plan your implementation of the actions in such way that you will not need synchronization.

Synchronization is possible in SuspendingMutex class only. It can be implemented without synchronization, though.

<h2>2.1           Program Flow</h2>

<table width="72">

 <tbody>

  <tr>

   <td width="32">three</td>

   <td width="40"> </td>

  </tr>

 </tbody>

</table>

The program is divided into                    phases. Once a phase is completed, you will proceed to the next phase.

<table width="519">

 <tbody>

  <tr>

   <td width="20">All</td>

   <td width="23">the</td>

   <td width="32">open</td>

   <td width="41">course</td>

   <td width="15">ac</td>

   <td width="30">tions</td>

   <td width="16">ap</td>

   <td width="28">pear</td>

   <td width="15">in</td>

   <td width="39">Phase</td>

   <td width="15">1.</td>

   <td width="38">There</td>

   <td width="38">might</td>

   <td width="16">ap</td>

   <td width="28">pear</td>

   <td width="35">other</td>

   <td width="15">ac</td>

   <td width="30">tions</td>

   <td width="16">as</td>

   <td width="29">well.</td>

  </tr>

 </tbody>

</table>

<table width="133">

 <tbody>

  <tr>

   <td width="27">Any</td>

   <td width="15">ac</td>

   <td width="25">tion</td>

   <td width="24">can</td>

   <td width="16">ap</td>

   <td width="26">pear</td>

  </tr>

 </tbody>

</table>

Phase 1: Phase 2:

<table width="186">

 <tbody>

  <tr>

   <td width="37">Phase</td>

   <td width="15">3:</td>

   <td width="28">Any</td>

   <td width="15">ac</td>

   <td width="25">tion</td>

   <td width="24">can</td>

   <td width="16">ap</td>

   <td width="26">pear</td>

  </tr>

 </tbody>

</table>

At rst you open all the suggested courses of all the departments. Once the rst phase is completed, it is possible for the students to register for the courses.

Note: Students can be added while the registration is run.

Note: Students can register for courses in di erent departments.

<h2>2.2           Actors</h2>

In order to simulate the university, you should create three types of actors:

An actor per student.

An actor per course.

An actor per department’s secretary.

<h2>2.3           Private States Of Actors</h2>

Each actor should maintain in its private state a log of all the action it has preformed. In addition, the private states include:

Department: A department’s private state includes list of courses and list of students in all the department.

Course: A course’s private state includes number of available spaces and list of students in the course, number of registered students, and prerequisites.

Student: A student’s private state includes grades sheet, and department’s signature. In the grades sheet appear all the courses the students learnt along with his grades.

Important: You are not allowed to use concurrent data structures to maintain the the private states of the actors.

Important: You are not allowed to extend the private state beyond what is described above, e.g, a department can not hold a list of students enrolled in each course, such data is part of the private state of the course only. Also, the department can not hold the grades sheet of the students, such is part of the private state of the student only.

4.3.1          Logging Actions

In its private state, the actor maintains a list of all the actions it has executed. The list holds only the description of the action as it is shown in the JSON le in section 4.9.2. Example: “Open Course”, “Add Student”, etc.

<h2>2.4           Actions</h2>

Following is a list of actions that you should enable.

Important: In each action you might create new actors, create new actions and submit them to other actors, etc. You must implement the action in the better way that ensures no synchronization on the private states of the actors.

<ol>

 <li>Open A New Course:</li>

</ol>

Behavior: This action opens a new course in a speci ed department. The course has an initially available spaces and a list of prerequisites.

Actor: Must be initially submitted to the Department’s actor.

<ol start="2">

 <li>Add Student:</li>

</ol>

Behavior: This action adds a new student to a speci ed department.

Actor: Must be initially submitted to the Department’s actor.

<ol start="3">

 <li>Participating In Course:</li>

</ol>

Behavior: This action should try to register the student in the course, if it succeeds, should add the course to the grades sheet of the student, and give him a grade if supplied. See the input example.

Actor: Must be initially submitted to the course’s actor.

<ol start="4">

 <li>Unregister:</li>

</ol>

Behavior: If the student is enrolled in the course, this action should unregister him (update the list of students of course, remove the course from the grades sheet of the student and increases the number of available spaces).

Actor: Must be initially submitted to the course’s actor.

<ol start="5">

 <li>Close A Course:</li>

</ol>

Behavior: This action should close a course. Should unregister all the registered students in the course and remove the course from the department courses’ list and from the grade sheets of the students. The number of available spaces of the closed course should be updated to -1. DO NOT remove its actor. After closing the course, all the request for registration should be denied.

Actor: Must be initially submitted to the department’s actor.

<ol start="6">

 <li>Opening New places In a Course:</li>

</ol>

Behavior: This action should increase the number of available spaces for the course.

Actor: Must be initially submitted to the course’s actor.

<ol start="7">

 <li>Check Administrative Obligations:</li>

</ol>

Behavior: The department’s secretary have to allocate one of the computers available in the warehouse, and check for each student if he meets some administrative obligations. The computer generates a signature and save it in the private state of the students.

Actor: Must be initially submitted to the department’s actor.

<ol start="8">

 <li>Announce about the end of registration period:</li>

</ol>

Behavior: From this moment, reject any further changes in registration. And, close courses with number of students less than 5.

Actor: Must be initially submitted to the department’s actor.

<h2>2.5           Feature : Support Preferences List</h2>

As you will experience, when it comes to interesting (or easy) elective courses it is not a trivial mission to nd a space for you. Therefore, we wish to support Preferences List, in which the student supply a list of courses he is interested in them, and wish to register for ONLY one course of them. The courses are ordered by preference. That is, if he succeed to register for the course with the highest preference it will not try to register for the rest, otherwise it will try to register for the second one, and so on. At the end, he will register for at most one course.

<h2>2.6           Warehouse</h2>

The warehouse class holds a nite amount of computers. Each computer has a Suspending Mutex (de ned next section). When the department wants to acquire a computer, it should lock its mutex if it is free. And release it once it nished the work with the computer. If the computer is not free, the department should not be blocked. It should get a promise which will be resolved later when the computer becomes available.

4.6.1        Computer

A computer has a method of checkAndSign. This method gets a list of the grades of the students and a list of courses he needs to pass (grade above 56). The method checks if the student passed all these courses. Each computer has two di erent signatures. One signatures for sign that the student meets the requirement and the other to sign the he does not.

<h2>2.7           Suspending Mutex</h2>

Holds a ag which indicates if the computer is free or not, and has a queue of promises to be resolved once the Mutex is available. In the Suspending Mutex there are two methods:

Up: Release the mutex.

Down: Acquire the mutex, If the mutex is not free, the thread should no be blocked. It should get a promise which will be resolved later when the printer becomes available.

Note: The Suspending Mutex can be implemented without any synchronization. However, using synchronization will be accepted as long as the implementation is blocking free.

<h2>2.8           Simulator</h2>

The simulator class is tasked with running the simulation. We may replace your ActorThreadPool implementation with our own during testing, so be sure to implement all simulation functionality in the Simulator class, not in ActorThreadPool!.

Once constructed, calling the simulators start() function will perform the following:

Parse the Json Files.

Submit actions to the thread pool passed to the method attachActorThreadPool.

DO NOT create an ActorThreadPool in start. You need to attach the ActorThreadPool in the main method, and then call start.

Calling simulator.end() will preform the following:

shut down the simulation.

returns a HashMap containing all the private states of the actors as serialized object to the le “result.ser”. You may do so using this code, or similar:

Listing 3: A Snippet Code For Writing The Output

<ul>

 <li>HashMap&lt;String , PrivateState&gt; SimulationResult ;</li>

 <li>SimulationResult = SimulatorImpl . end ();</li>

 <li>FileOutputStream fout = new FileOutputStream (” result . ser” );</li>

 <li>ObjectOutputStream oos = new ObjectOutputStream( fout );</li>

 <li>oos . writeObject ( SimulationResult );</li>

</ul>

Your output        lename MUST BE result.ser

<h2>2.9           Input Format</h2>

4.9.1           The JSON Format

All your input les for this assignment will be given as JSON les. You can read about JSONs syntax: http://www.json.org. In Java, there are a number of di erent options for parsing JSON les. Our recommendation is to use the library Gson. See the Gson User Guide (https://github.com/google/gson/blob/master/UserGuide.md) and APIs to see how to work with Gson. There are a lot of informative examples.

4.9.2        Input File

De ned below is a json input le that describes a single execution of our university. The simulation JSON le will be provided to your program as the rst command line argument.

The following is an example of that       le.

Important Note: Assume that the input is legal. That is, student does not try to register for course which is not exist, etc.

{

“threads”: 8,

“Computers” : [

{

“Type”:”A”,

“Sig Success”: “1234666”,

“Sig Fail”: “999283”

},

{

“Type”:”B”,

“Sig Success”: “4424232”,

“Sig Fail”: “5555353”

}

],

“Phase 1” : [

{

“Action”:”Open Course”,

“Department”: “CS”,

“Course”: “SPL”,

“Space”: “400”,

“Prerequisites” : [“Data Structures”, “Intro to CS”]

},

{

“Action”:”Open Course”, “Department”: “CS”,

“Course”: “Data Bases”,

“Space”: “30”,

“Prerequisites” : [“SPL”]

},

{

“Action”: “Add Student”,

“Department”: “CS”,

“Student”: “123456789”

}

],

“Phase 2” : [

{

“Action”: “Add Student”,

“Department”: “Math”,

“Student”: “132424353”

},

{

“Action”: “Participate In Course”,

“Student”: “123456789”,

“Course”: “SPL”,

“Grade”: [“98”]

},

{

“Action”: “Add Student”,

“Department”: “CS”,

“Student”: “5959595959”

},

{

“Action”: “Add Spaces”,

“Course”: “SPL”,

“Number”: “100”

},

{

“Action”: “Participate In Course”,

“Student”: “123456789”,

“Course”: “Data Bases”,

“Grade”: [“-“]

},

{

“Action”: “Register With Preferences”,

“Student”: “5959595959”,

“Preferences”: [“Data Bases”,”SPL”],

“Grade”: [“98″,”56”]

},

{

“Action”: “Unregister”,

“Student”: “123456789”,

“Course”: “Data Bases”

},

{

“Action”: “Close Course”,

“Department”: “CS”,

“Course”: “Data Bases”

},

{

“Action” : “End Registeration”

},

{

“Action” : “Administrative Check”,

“Department”: “CS”,

“Students”: [“123456789″,”5959595959”],

“Computer”: “A”,

“Conditions” : [“SPL”, “Data Bases”]

}

],

“Phase 3”: [

{

“Action” : “Administrative Check”,

“Department”: “CS”,

“Students”: [“123456789″,”5959595959”],

“Computer”: “A”,

“Conditions” : [“SPL”, “Data Bases”]

}

]

}

The        le holds a json object which contains the following       elds:

threads – an integer de ning the amount of threads in the Actor Thread Pool.

Computers – an array of computers in the warehouse, each computer has two signatures.

Phase 1 – An array of all the open courses actions, and some other action might appear. All the actions in Phase 1 should be completed before proceeding to Phase 2.

Phase 2 – An array of different actions. All the actions in Phase 2 should be completed before proceeding

<table width="66">

 <tbody>

  <tr>

   <td width="14">to</td>

   <td width="39">Phase</td>

   <td width="13">3.</td>

  </tr>

 </tbody>

</table>

Phase 3 – An array of different actions.

Note: the number and order of the actions might be di erent. You must insert the actions in the order they appear in the json.

Participate In Course: In this action, you should try to register the student in the course (if there is available space and he meets the prerequisites of the course). If the mission is succeed, then if the grade is not “-“, you should give him a grade.

Register With Preferences: In Grade, as in Participate In Course. Here we give a list of grades, each grade for a course. Remember: the student can register for one course at most from the supplied list.

Important:The simulation le should be given as argument in the main function. That is, you can’t use a prede ned name or location. The run command is: mvn exec:java -Dexec.mainClass=”bgu.spl.a2.sim.Simulator” -Dexec.args=”myFile.json”.

In Dexec.args we pass arguments to the main method – in our case this argument represents the simulation le (the le name is NOT necessary myFile.json).