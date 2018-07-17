================
Welcome to Abaco
================

   
What is Abaco
_____________
    
**Abaco** is an NSF-funded web service and distributed computing platform providing functions-as-a-service (FaaS)
to the research computing community. Abaco implements functions using the Actor Model of concurrent computation. In
Abaco, each actor is associated with a Docker image, and actor containers are executed in response to messages posted
to their inbox which itself is given by a URI exposed over HTTP.

Abaco will ultimately offer three primary higher-level capabilities on top of the underlying Actor model:
 
 * *Reactors* for event-driven programming
 * *Asynchronous Executors* for scaling out function calls within running applications, and
 * *Data Adapters* for creating rationalized microservices from disparate and heterogeneous sources of data.

Reactors and Asynchronous Executors are available today while Data Adapters are still under active development.

Using Abaco
___________

Abaco is in production and has been adopted by several projects. Abaco is available to researchers and students. To
learn more about the the system, including getting access, follow the instructions in :doc:`getting-started/index`.
