.. _state:

===========
Actor State
===========

In this section we describe the state that persists (or not) through Abaco actor container executions.

State
-----

When an actor is registered, its ``stateless`` property is automatically set to ``true``. An actor must be registered with ``stateless=false`` to be stateful (maintain state across executions).

Once an actor is executed, the associated worker ``GETs`` data from the ``/actors/v2/{actor_id}/state`` endpoint and injects it into the actor's ``_abaco_actor_state`` environment variable. While an actor is executing, it can update its state by ``POSTing`` to ``/actors/v2/{actor_id}/state``.


Notes
~~~~~
- The worker only ``GETs`` data from the state endpoint one time as the actor is initiated. If the actor updates its state endpoint during execution, the worker does not inject the new state until a new execution.
- Stateful actors may only have one associated worker in order to avoid race conditions. Thus, generally, stateless actors will execute quicker as they're runnable in parallel.
- Issuing a state to a stateless actor will return a ``actor is stateless.`` error.
- The ``state`` variable must be JSON-serializable. An example of passing JSON-serializable data can be found under `Examples`_ below.

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
Here are some examples interacting with state using Python. The ``agavepy.actors`` module provides access to an actor's environment data in native Python objects.

Registering an actor specifying statefulness: ``stateless=false``.

.. code-block:: bash

  >>> from agavepy.agave import Agave
  >>> ag = Agave(api_server='https://api.tacc.utexas.edu', token='<access_token>')
  >>> actor = {"image": "abacosamples/test",
				"stateless": "False"}
  >>> ag.actors.add(body=actor)

POSTing a state to a particular actor; again keep in mind we must pass in JSON serializable data.

.. code-block:: bash

  >>> from agavepy.actors import update_state
  >>> state = {"some variable": "value", "another variable": "value2"}
  >>> update_state(state)

GETting information about a particular actor's state. This function returns a Python dictionary with many fields one of which is state. 

.. code-block:: bash

  >>> from agavepy.actors import get_context
  >>> get_context()
  {'raw_message': '<text>', 'content_type': '<text>', 'execution_id': '<text>', 'username': '<text>', 'state': 'some_state', 'actor_dbid': '<text>', 'actor_id': '<text>', 'raw_message_parse_log': '<text>', 'message_dict': {}}



Additional Work
---------------
- Create a pipeline between worker and actor to exchange state without HTTP latency. (Not worker->server->actor)
- Develop 'stateful' actors that can execute in parallel (utilizing CRDT data-types)








:reference:`state`