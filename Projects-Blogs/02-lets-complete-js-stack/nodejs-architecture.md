## Node JS Architecture

Before starting some Node JS programming examples, it’s important to have an idea about Node JS architecture. We will discuss about “How Node JS works under-the-hood, what type of processing model it is following, How Node JS handles concurrent request with Single-Threaded model” etc. in this post.

### Node JS Single Threaded Event Loop Model

As we have already discussed, Node JS applications uses “Single Threaded Event Loop Model” architecture to handle multiple concurrent clients.

There are many web application technologies like JSP, Spring MVC, ASP.NET, HTML, Ajax, jQuery etc. But all these technologies follow “Multi-Threaded Request-Response” architecture to handle multiple concurrent clients.

We are already familiar with “Multi-Threaded Request-Response” architecture because it’s used by most of the web application frameworks. But why Node JS Platform has chosen different architecture to develop web applications. What is the major differences between multithreaded and single threaded event loop architecture.

Any web developer can learn Node JS and develop applications very easily. However without understanding Node JS Internals, we cannot design and develop Node JS Applications very well. So before starting developing Node JS Applications, first we will learn Node JS Platform internals.

### Node JS Platform

Node JS Platform uses “Single Threaded Event Loop” architecture to handle multiple concurrent clients. Then how it really handles concurrent client requests without using multiple threads. What is Event Loop model? We will discuss these concepts one by one.

Before discussing “Single Threaded Event Loop” architecture, first we will go through famous “Multi-Threaded Request-Response” architecture.

### Traditional Web Application Processing Model

Any Web Application developed without Node JS, typically follows “Multi-Threaded Request-Response” model. Simply we can call this model as Request/Response Model.


Client sends request to the server, then server do some processing based on clients request, prepare response and send it back to the client.

This model uses HTTP protocol. As HTTP is a Stateless Protocol, this Request/Response model is also Stateless Model. So we can call this as Request/Response Stateless Model.

However, this model uses Multiple Threads to handle concurrent client requests. Before discussing this model internals, first go through the diagram below.

**Request/Response Model Processing Steps:**

* Clients Send request to Web Server.
* Web Server internally maintains a Limited Thread pool to provide services to the Client Requests.
* Web Server is in infinite Loop and waiting for Client Incoming Requests
* Web Server receives those requests.
  * Web Server pickup one Client Request
  * Pickup one Thread from Thread pool
  * Assign this Thread to Client Request
  * This Thread will take care of reading Client request, processing Client request, performing any Blocking IO Operations (if required) and preparing Response
  * This Thread sends prepared response back to the Web Server
  * Web Server in-turn sends this response to the respective Client.
  * Server waits in Infinite loop and performs all sub-steps as mentioned above for all n clients. That means this model creates one Thread per Client request.

If more clients requests require Blocking IO Operations, then almost all threads are busy in preparing their responses. Then remaining clients Requests should wait for longer time.

![https://cdn.journaldev.com/wp-content/uploads/2015/04/Request-Response-Model.png](https://cdn.journaldev.com/wp-content/uploads/2015/04/Request-Response-Model.png)

<p><strong>Diagram Description</strong>:</p>
                                <ul>
                                    <li>Here &#8220;n&#8221; number of Clients Send request to Web Server. Let us assume they are accessing our Web Application concurrently.</li>
                                    <li>Let us assume, our Clients are Client-1, Client-2… and Client-n.</li>
                                    <li>Web Server internally maintains a Limited Thread pool. Let us assume &#8220;m&#8221; number of Threads in Thread pool.</li>
                                    <li>Web Server receives those requests one by one.
                                        <ul>
                                            <li>Web Server pickup Client-1 Request-1, Pickup one Thread T-1 from Thread pool and assign this request to Thread T-1
                                                <ul>
                                                    <li>Thread T-1 reads Client-1 Request-1 and process it</li>
                                                    <li>Client-1 Request-1 does not require any Blocking IO Operations</li>
                                                    <li>Thread T-1 does necessary steps and prepares Response-1 and send it back to the Server</li>
                                                    <li>Web Server in-turn send this Response-1 to the Client-1</li>
                                                </ul>
                                        </ul>
                                        <ul>
                                            <li>Web Server pickup another Client-2 Request-2, Pickup one Thread T-2 from Thread pool and assign this request to Thread T-2
                                                <ul>
                                                    <li>Thread T-2 reads Client-1 Request-2 and process it</li>
                                                    <li>Client-1 Request-2 does not require any Blocking IO Operations</li>
                                                    <li>Thread T-2 does necessary steps and prepares Response-2 and send it back to the Server</li>
                                                    <li>Web Server in-turn send this Response-2 to the Client-2</li>
                                                </ul>
                                        </ul>
                                        <ul>
                                            <li>Web Server pickup another Client-n Request-n, Pickup one Thread T-n from Thread pool and assign this request to Thread T-n
                                                <ul>
                                                    <li>Thread T-n reads Client-n Request-n and process it</li>
                                                    <li>Client-n Request-n require heavy Blocking IO and computation Operations</li>
                                                    <li>Thread T-n takes more time to interact with external systems, does necessary steps and prepares Response-n and send it back to the Server</li>
                                                    <li>Web Server in-turn send this Response-n to the Client-n</li>
                                                </ul>
                                        </ul>
                                        <p>If &#8220;n&#8221; is greater than &#8220;m&#8221; (Most of the times, its true), then server assigns Threads to Client Requests up to available Threads. After all m Threads are utilized, then remaining Client’s Request should wait in the Queue until some of the busy Threads finish their Request-Processing Job and free to pick up next Request.</p>
                                        <p>If those threads are busy with Blocking IO Tasks (For example, interacting with Database, file system, JMS Queue, external services etc.) for longer time, then remaining clients should wait longer time.</li>
                                            <li>Once Threads are free in Thread Pool and available for next tasks, Server pickup those threads and assign them to remaining Client Requests.</li>
                                            <li>Each Thread utilizes many resources like memory etc. So before going those Threads from busy state to waiting state, they should release all acquired resources.</li>
                                </ul>
                                <p><strong>Drawbacks of Request/Response Stateless Model</strong>:</p>
                                <ul>
                                    <li>Handling more and more concurrent client’s request is bit tough.</li>
                                    <li>When Concurrent client requests increases, then it should use more and more threads, finally they eat up more memory.</li>
                                    <li>Sometimes, Client’s Request should wait for available threads to process their requests.</li>
                                    <li>Wastes time in processing Blocking IO Tasks.</li>
                                </ul>
                                <h3>Node JS Architecture &#8211; Single Threaded Event Loop</h3>
                                <p>Node JS Platform does not follow Request/Response Multi-Threaded Stateless Model. It follows Single Threaded with Event Loop Model. Node JS Processing model mainly based on Javascript Event based model with Javascript callback mechanism.</p>

<p>You should have some good knowledge about how Javascript events and callback mechanism works. If you don’t know, Please go through those posts or tutorials first and get some idea before moving to the next step in this post.</p>
                                <p>As Node JS follows this architecture, it can handle more and more concurrent client requests very easily. Before discussing this model internals, first go through the diagram below.</p>
                                <p>I tried to design this diagram to explain each and every point of Node JS Internals.</p>
                                <p>The main heart of Node JS Processing model is &#8220;Event Loop&#8221;. If we understand this, then it is very easy to understand the Node JS Internals.</p>
                                <p><strong>Single Threaded Event Loop Model Processing Steps</strong>:</p>
                                <ul>
                                    <li>Clients Send request to Web Server.</li>
                                    <li>Node JS Web Server internally maintains a Limited Thread pool to provide services to the Client Requests.</li>
                                    <li>Node JS Web Server receives those requests and places them into a Queue. It is known as &#8220;Event Queue&#8221;.</li>
                                    <li>Node JS Web Server internally has a Component, known as &#8220;Event Loop&#8221;. Why it got this name is that it uses indefinite loop to receive requests and process them. (See some Java Pseudo code to understand this below).</li>
                                    <li>Event Loop uses Single Thread only. It is main heart of Node JS Platform Processing Model.</li>
                                    <li>Even Loop checks any Client Request is placed in Event Queue. If no, then wait for incoming requests for indefinitely.</li>
                                    <li>If yes, then pick up one Client Request from Event Queue
                                        <ul>
                                            <li>Starts process that Client Request</li>
                                            <li>If that Client Request Does Not requires any Blocking IO Operations, then process everything, prepare response and send it back to client.</li>
                                            <li>If that Client Request requires some Blocking IO Operations like interacting with Database, File System, External Services then it will follow different approach
                                                <ul>
                                                    <li>Checks Threads availability from Internal Thread Pool</li>
                                                    <li>Picks up one Thread and assign this Client Request to that thread.</li>
                                                    <li>That Thread is responsible for taking that request, process it, perform Blocking IO operations, prepare response and send it back to the Event Loop</li>
                                                    <li>Event Loop in turn, sends that Response to the respective Client.</li>
                                                </ul>
                                            </li>
                                        </ul>
                                    </li>
                                </ul>
                                <p>
                                    <a href="https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model.png"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" data-lazy-type="image" data-lazy-src="https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-1024x768.png" alt="NodeJS-Single-Thread-Event-Model" width="660" height="495" class="lazy lazy-hidden aligncenter size-large wp-image-7465" data-lazy-srcset="https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-1024x768.png 1024w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-450x338.png 450w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-700x525.png 700w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-150x113.png 150w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model.png 1062w" data-lazy-sizes="(max-width: 660px) 100vw, 660px" />
                                        <noscript><img src="https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-1024x768.png" alt="NodeJS-Single-Thread-Event-Model" width="660" height="495" class="aligncenter size-large wp-image-7465" srcset="https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-1024x768.png 1024w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-450x338.png 450w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-700x525.png 700w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model-150x113.png 150w, https://cdn.journaldev.com/wp-content/uploads/2015/04/NodeJS-Single-Thread-Event-Model.png 1062w" sizes="(max-width: 660px) 100vw, 660px" /></noscript>
                                    </a>
                                </p>
                                <p><strong>Diagram Description</strong>:</p>
                                <ul>
                                    <li>Here &#8220;n&#8221; number of Clients Send request to Web Server. Let us assume they are accessing our Web Application concurrently.</li>
                                    <li>Let us assume, our Clients are Client-1, Client-2… and Client-n.</li>
                                    <li>Web Server internally maintains a Limited Thread pool. Let us assume &#8220;m&#8221; number of Threads in Thread pool.</li>
                                    <li>Node JS Web Server receives Client-1, Client-2… and Client-n Requests and places them in the Event Queue.</li>
                                    <li>Node JS Even Loop Picks up those requests one by one.
                                        <ul>
                                            <li>Even Loop pickups Client-1 Request-1
                                                <ul>
                                                    <li> Checks whether Client-1 Request-1 does require any Blocking IO Operations or takes more time for complex computation tasks.</li>
                                                    <li>As this request is simple computation and Non-Blocking IO task, it does not require separate Thread to process it.</li>
                                                    <li>Event Loop process all steps provided in that Client-1 Request-1 Operation (Here Operations means Java Script’s functions) and prepares Response-1</li>
                                                    <li>Event Loop sends Response-1 to Client-1</li>
                                                </ul>
                                            </li>
                                            <li>Even Loop pickups Client-2 Request-2
                                                <ul>
                                                    <li> Checks whether Client-2 Request-2does require any Blocking IO Operations or takes more time for complex computation tasks.</li>
                                                    <li>As this request is simple computation and Non-Blocking IO task, it does not require separate Thread to process it.</li>
                                                    <li>Event Loop process all steps provided in that Client-2 Request-2 Operation and prepares Response-2</li>
                                                    <li>Event Loop sends Response-2 to Client-2</li>
                                                </ul>
                                            </li>
                                            <li>Even Loop pickups Client-n Request-n
                                                <ul>
                                                    <li> Checks whether Client-n Request-n does require any Blocking IO Operations or takes more time for complex computation tasks.</li>
                                                    <li>As this request is very complex computation or Blocking IO task, Even Loop does not process this request.</li>
                                                    <li>Event Loop picks up Thread T-1 from Internal Thread pool and assigns this Client-n Request-n to Thread T-1</li>
                                                    <li>Thread T-1 reads and process Request-n, perform necessary Blocking IO or Computation task, and finally prepares Response-n</li>
                                                    <li>Thread T-1 sends this Response-n to Event Loop</li>
                                                    <li>Event Loop in turn, sends this Response-n to Client-n</li>
                                                </ul>
                                            </li>
                                        </ul>
                                    </li>
                                </ul>
                                <p>Here Client Request is a call to one or more Java Script Functions. Java Script Functions may call other functions or may utilize its Callback functions nature.</p>
                                <p>So Each Client Request looks like as shown below:</p>
                                <p>
                                    <a href="https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism.png"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" data-lazy-type="image" data-lazy-src="https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism.png" alt="Node JS Architecture, Single Threaded Event Loop" width="579" height="102" class="lazy lazy-hidden aligncenter size-large wp-image-7470" data-lazy-srcset="https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism.png 579w, https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism-450x79.png 450w, https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism-150x26.png 150w" data-lazy-sizes="(max-width: 579px) 100vw, 579px" />
                                        <noscript><img src="https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism.png" alt="Node JS Architecture, Single Threaded Event Loop" width="579" height="102" class="aligncenter size-large wp-image-7470" srcset="https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism.png 579w, https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism-450x79.png 450w, https://cdn.journaldev.com/wp-content/uploads/2015/04/javascript-callback-mechanism-150x26.png 150w" sizes="(max-width: 579px) 100vw, 579px" /></noscript>
                                    </a>
                                </p>
                                
                                
                                <p>For Example:</p><pre class="prettyprint">
function1(function2,callback1);
function2(function3,callback2);
function3(input-params);
</pre>
                                <p><strong>NOTE</strong>: &#8211;</p>
                                <ul>
                                    <li>If you don’t understand how these functions are executed, then I feel you are not familiar with Java Script Functions and Callback mechanism.</li>
                                    <li>We should have some idea about Java Script functions and Callback mechanisms. Please go through some online tutorial before starting our Node JS Application development.</li>
                                </ul>
                                <h3>Node JS Architecture &#8211; Single Threaded Event Loop Advantages</h3>
                                <ol>
                                    <li>Handling more and more concurrent client’s request is very easy.</li>
                                    <li>Even though our Node JS Application receives more and more Concurrent client requests, there is no need of creating more and more threads, because of Event loop.</li>
                                    <li>Node JS application uses less Threads so that it can utilize only less resources or memory</li>
                                </ol>
                                <h3>Event Loop Pseudo Code</h3>
                                <p>As I’m a Java Developer, I will try to explain &#8220;How Event Loop works&#8221; in Java terminology. It is not in pure Java code, I guess everyone can understand this. If you face any issues in understanding this, please drop me a comment.</p><pre class="prettyprint">
public class EventLoop {
while(true){
        	if(Event Queue receives a JavaScript Function Call){
        		ClientRequest request = EventQueue.getClientRequest();
                            If(request requires BlokingIO or takes more computation time)
                                    Assign request to Thread T1
                            Else
                                  Process and Prepare response
                  }
            }
} 
</pre>
                                <p>That&#8217;s all for Node JS Architecture and Node JS single threaded event loop.</p>
