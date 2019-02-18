.. _state:

===========
Actor State
===========

In this section we describe the state that persists (or not) through Abaco actor container executions.

State
-----

When an actor is registered, its ``stateless`` property is automatically set to ``true``. An actor must be registered with ``stateless=false`` to be stateful (maintain state across executions).

Once an actor is executed, the associated worker ``GETs`` data from the ``/actors/v2/{actor_id}/state`` endpoint and injects it into the actor's ``state``/``_abaco_actor_state`` environment variable?**. While an actor is executing, it can update its state by ``POSTing`` to ``/actors/v2/{actor_id}/state``.


Notes
~~~~~
- The worker only ``GETs`` data from the state endpoint one time as the actor is initiated. If the actor updates its state endpoint during execution, the worker does not inject the new state until a new execution.
- Stateful actors may only have one associated worker in order to avoid race conditions. Thus, generally, stateless actors will execute quicker as they're runnable in parallel.
- Issuing a state to a stateless actor will return a ``actor is stateless.`` error.
- The ``state`` variable must be JSON-serializable. An example of passing JSON-serializable data can be found under :curl:**.

Utilizing State in Actors to Accomplish Something
-------------------------------------------------
**WIP**


Examples
--------

curl
~~~~
Here are some examples interacting with state using curl.


Registering an actor specifying statefulness: ``stateless=false``.

.. code-block:: bash

  $curl -H "$header" \
  -X POST \
  -d "image=abacosamples/test&stateless=false" \
  https://api.tacc.utexas.edu/actors/v2

POSTing a state to a particular actor; keep in mind we must indicate in the header that we are passing content type ``application/json``.

.. code-block:: bash

  $curl -H "$header" \
  -H "Content-Type: application/json" \
  -d '{"some variable": "value", "another variable": "value2"}' \
  https://api.tacc.utexas.edu/actors/v2/<actor_id>/state

GETting information about a particular actor's state.

.. code-block:: bash

  $curl -H "$header" \
  https://api.tacc.utexas.edu/actors/v2/<actor_id>/state


Python
~~~~~~
**WIP**

"The ``agavepy.actors`` module provides access to the above data in native Python objects.
Currently, the actors module provides the following utilities:

* ``get_context()`` - returns a Python dictionary with the following fields:
    * ``raw_message`` - the original message, either string or JSON depending on the Contetnt-Type.
    * ``content_type`` - derived from the original message request.
    * ``message_dict`` - A Python dictionary representing the message (for Content-Type: application/json)
    * ``execution_id`` - the ID of this execution.
    * ``username`` - the username of the user that requested the execution.
    * ``state`` - (for stateful actors) state value at the start of the execution.
    * ``actor_id`` - the actor's id.
* ``get_client()`` - returns a pre-authenticated ``agavepy.Agave`` object.
* ``update_state(val)`` - Atomically, update the actor's state to the value ``val``."





Additional Work
---------------
- Create a pipeline between worker and actor to exchange state without HTTP latency. (Not worker->server->actor)
- Develop 'stateful' actors that can execute in parallel (utilizing CRDT data-types)








:reference:`state`
``