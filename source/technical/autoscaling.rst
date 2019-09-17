.. _autoscaling:

==================
Autoscaling Actors
==================

The Abaco platform has an optional autoscaler subsystem for automatically managing the pool of workers associated with
the registered actors. In general, the autoscaler ignores actors that are registered with ``stateless: False``, as it
assumes these actors must process their message queues synchronously. For `stateless` actors without custom
configurations, the austocaling algorithm is as follows:

1. Every 5 seconds, check the length of the actor's message queue.
2. If the queue length is greater than 0, and the actor's worker pool is less than the maximum workers per actor, start a new worker.
3. If the queue length is 0, reduce the actor's worker pool until: a) the worker pool size becomes 0 or b) the actor receives a message.

In particular, the worker pool associated with an actor with 0 messages in its message queue will be reduced to 0 to
free up resources on the Abaco compute cluster.


Official "sync" Hint
--------------------

.. Important::
   Support for actor hints and the official "sync" hint was added in version 1.4.0.

For some use cases, reducing an actor's worker pool to 0 as soon as its message queue is empty is not desirable.
Starting up a worker takes significant time, typically on the order of 10 seconds or more, depending on configuration
options for the actor, and adding this overhead to actors that have low latency requirements can be a serious issue.
In particular, actors that will respond to "synchronous messages" (i.e., ``_abaco_synchronous=true``) have low
latency requirements to respond within the HTTP timeout window.

For this reason, starting in version 1.4.0, Abaco recognizes an "official" actor hint, ``sync``. When registered
with the ``sync`` hint, the Abaco autoscaler will leave at least one worker in the actor's worker pool up to a
configurable period of idle time (specific to the Abaco tenant). For the Abaco public tenant, this period is 60
minutes.

The ``hints`` attribute for an actor is saved at registration time. In the following example, we register an
actor with the ``sync`` hint using curl:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-type: application/json" \
    -d '{"image": "abacosamples/wc", "hints": ["sync"]}' \
    https://api.tacc.utexas.edu/actors/v2




