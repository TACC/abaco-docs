.. _context:

=================================
Abaco Context & Container Runtime
=================================

In this section we describe the environment that Abaco actor containers can utilize during their execution.

Context
-------

When an actor container is launched, Abaco injects information about the execution into a number of environment
variables. This information is collectively referred to as the ``context``. The following table provides a complete
list of variable names and their description:

+----------------------+-------------------------------------------------------------------------+
| Variable Name        | Description                                                             |
+======================+=========================================================================+
| _abaco_actor_id      | The id of the actor.                                                    |
+----------------------+-------------------------------------------------------------------------+
| _abaco_actor_dbid    | The Abaco internal id of the actor.                                     |
+----------------------+-------------------------------------------------------------------------+
| _abaco_container_repo| The Docker image used to launch this actor container.                   |
+----------------------+-------------------------------------------------------------------------+
| _abaco_worker_id     | The id of the worker for the actor overseeing this execution.           |
+----------------------+-------------------------------------------------------------------------+
| _abaco_execution_id  | The id of the current execution.                                        |
+----------------------+-------------------------------------------------------------------------+
| _abaco_access_token  | An OAuth2 access token representing the user who registered the actor.  |
+----------------------+-------------------------------------------------------------------------+
| _abaco_api_server    | The OAuth2 API server associated with the actor.                        |
+----------------------+-------------------------------------------------------------------------+
| _abaco_actor_state   | The value of the actor's state at the start of the execution.           |
+----------------------+-------------------------------------------------------------------------+
| _abaco_Content-Type  | The data type of the message (either 'str' or 'application/json').      |
+----------------------+-------------------------------------------------------------------------+
| _abaco_username      | The username of the "executor", i.e., the user who sent the message.    |
+----------------------+-------------------------------------------------------------------------+
| _abaco_api_server    | The base URL for the Abaco API service.                                 |
+----------------------+-------------------------------------------------------------------------+
|  MSG                 | The message sent to the actor, as a raw string.                         |
+----------------------+-------------------------------------------------------------------------+


Notes
~~~~~

- The ``_abaco_actor_dbid`` is unique to each actor. Using this id, an actor can distinguish itself from other actors registered with the same function providing for SPMD techniques.
- The ``_abaco_access_token`` is a valid OAuth token that actors can use to make authenticated requests to other TACC Cloud APIs during their execution.
- The actor can update its state during the course of its execution; see the section :ref:`state` for more details.
- The "executor" of the actor may be different from the owner; see :ref:`sharing` for more details.


Access from Python
~~~~~~~~~~~~~~~~~~

The ``agavepy.actors`` module provides access to the above data in native Python objects.
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
* ``update_state(val)`` - Atomically, update the actor's state to the value ``val``.


Runtime Environment
-------------------

The environment in which an Abaco actor container runs has been built to accommodate a number of typical use cases
encountered in research computing in a secure manner.


Container UID and GID
~~~~~~~~~~~~~~~~~~~~~

When Abaco launches an actor container, it instructs Docker to execute the process using the UID and GID associated
with the TACC account of the owner of the actor. This practice guarantees that an Abaco actor will have exactly the
same accesses as the original author of the actor (for instance, access to files or directories on shared storage)
and that files created or updated by the actor process will be owned by the underlying API user.
Abaco API users that have elevated privilleges within the platform can override the UID and GID used to run the
actor when registering the actor (see :ref:`registration`).


POSIX Interface to the TACC WORK File System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When Abaco launches an actor container, it mounts the actor owner's TACC WORK file system into the running container.
The owner's work file system is made available at ``/work`` with the container. This gives the actor a POSIX
interface to the work file system.