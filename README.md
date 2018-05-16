# jz18sub
## Json/Zmq 2018 Submission/Details ##
```
    So you start a new position and the first question is along the
    lines of 'we are rebuilding our systems, ZMQ, RabbitMQ, or 
    Protobuffers which do you like more?' ummmmm let me look?
```

We are going to walk through using ZMQ and JSON to build a messaging platform which we use to help us process credit card transactions.
Our structure is made up of a primary daemon as a MCP.  
Then we have individual resources or processing apps that when started register with the MCP.  
We will go over how and why we found using JSON was the easiest and most effective, even with converting everything to strings.


### Notes ###
- Primary Linux Development, but cross platform libraries and tools.

- Master Control Program - database agnostic?  use in  memory tables or sqlite.
- Resource App - used to proxy something - rti/hsm/filetran.  comm with deal/router
- Processing App - pull from resource apps with pub/sub list
- How we use and convert structures/values with json.

About Json/Zmq and application network for financial transactions.

#### Why ZMQ?  ####
- Advantages

#### Why json? ####
- Zmq is all CHAR
- Database is all CHAR

#### Code Requirements ####
- Library Base
  - App class
  - ZmqHandler
  - ZBSocket
  - Conversion Utilities (Safe String)
- Automation 
  - SQLite DB
- Resource App
  - HSMProxy/RTI
  - File Read/Write
- Compute/Processing App
  - GPGPU
  - In/Out


##### Other Projects Used ######
Here are a couple other projects I reference for ease.

- [ZMQ PP - C++ Interace for libzmq](https://github.com/zeromq/zmqpp)
- [SpdLog - Fast header only logger](https://github.com/gabime/spdlog)


### Outline ###

Quickly: Transactions  
- What is it? Types (card/emv/web).  
- What does this have to do with when I swipe my card at the grocery store? 
- Example of credit card data transactions routing through the network. 
- Quickly demonstrate settlement pieces we re-created in our example code. 
- PCI surrounds everything we do. 

Background: Why did you do this?
- We are addressing speed, performance, and maintainability.   Modernizing sections of older software, we have a rewrite done as a refactor thanks to our applications being very modular.  Single threaded Windows applications running multiple instances is inefficient.   
- The difficulty in the current system is inefficient use of resources for modern operating systems. We have a lot of database dependency and reliance on stored procedures. Further Visa/MC systems have some legacy (MML/EBCDIC) systems to integrate with. 
  - Also... Visa/MC specs are massive.  Just EMV has its own books and site emvco.com and it's primarily just chip! 
- Looking at similar problems solved previous we looked for the shortest path to solutions. There are several libraries we were able to use to help us reach a desired impact in less time which we will show. 
- When upgrading software from the level we were, performance increases were a given.  However, that was a bias we would find would catch us if we were not careful.  Every line of legacy code added as a bug fix, makes it harder to translate in new code. 

Approach: What did you do? 
- Current systems are modular, we started with a resource management application for Hardware Security Module (HSM).  Followed by processing applications (Settlement) and finally application/system management. 
- We use timing as the primary performance measure as a transaction processor. 
- Using lots of logging we looked at existing timing and code and designed new fixes, incremental in some cases. 
  - HSM Manager was singled threaded – now its queued and multithreaded. 
- Each app was built in the support of the next. 
  - Cryptographic hardware is required to manage card data. PCI. 
  - Settlement app updated next to use new HSM Manager. 
  - Control app created to managed resources and processing applications. 

Findings: What did you learn or discover? 
- We found implementing ZMQ library along with JSON gave us a fast flexible and easy way to move data between the different applications on the network. 
- We give an example of implementing a robust framework in a financial network. 
- Due to OS limitations in production we were originally bound by C++98.  We moved to C++11 but then found we had need of C++14 features and once again had to move.  This entailed managing compiler and library differences in older production OS. 
- ZMQ and JSON was able to help us speed up our networks by reducing data (vs xml) and was easy to implement in modern C++. 
- Not everything we wrote was a speed increase. Threads have to be managed. Timing is always an issue. ZMQ will wait for a connection to come up and continue, can make testing interesting when something comes up with 20 requests waiting and floods the new app. 

Conclusion: What does it mean? 
- In our previous configuration we had a A/B failover. Now we can have multiple redundancies across the entire platform. 
- Clients will be able to receive more responses faster. Our parent company is able to onboard more clients. 
- Will something be cheaper, faster, or better? YES! 



Resources
[HSM - Hardware Security Module](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms)
