.. _complex:

==================
Networks of Actors
==================

Working with individual, isolated actors can augment an existing application with a lot of additional functionality, but the
full power of Abaco's actor-based system is realized when many actors coordinate together to solve a common problem.
Actor coordination introduces new challenges that the system designer must address, and Abaco provides
features specifically designed to address these challenges.


Actor Aliases
-------------

An `alias` is a user-defined name for an actor that is managed independently of the actor itself. Put simply, an alias
maps a name to an actor id, and Abaco will replace a reference to an alias in any request with the actor id defined by
the alias at the time. Aliases are useful for insulating an actor from changes to another actor to which it will
send messages.

For example, if actor A sends a message to actor B, the user can create an alias for actor B and configure A to send
messages to that alias. In the future, if changes need to be made to actor B or if messages from actor A need to be
routed to a different actor, the alias value can be updated without any code changes needed on the part of actor A.

Creating and managing aliases is done via the `aliases` collection.

cURL
~~~~

To create an alias make a POST request passing the alias and actor id. For example, suppose we have an actor that counts
word sent in a message. We might create an alias for it with the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "alias=counter&actorId=6PlMbDLa4zlON" \
    https://api.tacc.utexas.edu/actors/v2/aliases

Example response:

.. code-block:: bash

    {
      "message": "Actor alias created successfully.",
      "result": {
        "_links": {
          "actor": "https://api.tacc.utexas.edu/actors/v2/6PlMbDLa4zlON",
          "owner": "https://api.tacc.utexas.edu/profiles/v2/jstubbs",
          "self": "https://api.tacc.utexas.edu/actors/v2/aliases/counter"
        },
        "actorId": "6PlMbDLa4zlON",
        "alias": "counter",
        "owner": "apitest"
      },
      "status": "success",
      "version": "1.1.0"
    }

With the alias `counter` created, we can now use it in place of the actor id. For example, we can get the actor's details:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/counter

The response returned is identical to that returned when the actor id is used.


Nonces Attached to Aliases
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Important::
   Support for Nonces attached to aliases was added in version 1.1.0.

Nonces can be created for aliases in much the same way as creating nonces for a specific actor id - instead of using
the `nonces` endpoint associated with the actor id, use the `nonces` endpoint associated with the alias instead. The
POST message payload is the same. For example:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "maxUses=5&level=READ" \
    https://api.tacc.utexas.edu/actors/v2/aliases/counter/nonces

will create a nonce associated with the `counter` alias.

.. Note::
  Listing, creating and deleting nonces associated with an alias requires the analagous permission for both the alias
  **and** the associated actor.


Actor Events and Actor Links
----------------------------

.. Important::
   Support for Actor events and links was added in version 1.2.0.

Abaco provides a facility to automatically send a message to a specified actor whenever certain events occur. This
mechanism is called an actor `link`: if actor A is registered with a `link` property specifying actor B, then Abaco will
automatically send actor B a message whenever any of the following events occur:

  * Actor A's status changes (for example, from SUBMITTED to READY or from READY to ERROR).
  * An execution for actor A completes.

Adding a Link
~~~~~~~~~~~~~

Registering an actor with a link (or updating an exisitng actor to add a link property) follows the same semantics as
defined in the :ref:`registration` section; simply add the `link` attribute in the payload. For example, the following
creates an actor with a link to actor id `6PlMbDLa4zlON`.

.. code-block:: bash

  $ curl -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image": "abacosamples/test", "name": "test", "link": "6PlMbDLa4zlON", "description": "My test actor using the abacosamples image.", "default_environment":{"key1": "value1", "key2": "value2"} }' \
  https://api.tacc.utexas.edu/actors/v2

It is also possible to link an actor to an alias: just pass `link=<the_alias>` in the registration payload.

Events and Event Message Format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whenever a supported event occurs, Abaco sends a JSON message to the linked actor with data about the event. The
included data depends on the event type, as documented below. Note that all the typical context variables, as documented
in :ref:`context`, will also be injected, excepted where noted below:

+---------------------+--------------------------------------------------------------------------+--------------------+
| Variable Name       | Description                                                              | Event Type         |
+=====================+==========================================================================+====================+
| actor_id            | The id of the actor.                                                     | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| tenant_id           | The id of the tenant of the actor.                                       | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| event_type          | The event type associated with the event. (see table below)              | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| _abaco_link         | The actor id of the linked actor (the actor receiving the event message  | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| _abaco_username     | 'Abaco Event'                                                            | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| status_message      | A message indicating details about the error status.                     | ACTOR_ERROR        |
+---------------------+--------------------------------------------------------------------------+--------------------+
| execution_id        | The id of the completed execution.                                       | EXECUTION_COMPLETE |
+---------------------+--------------------------------------------------------------------------+--------------------+
| exit_code           | The exit code of the completed execution.                                | EXECUTION_COMPLETE |
+---------------------+--------------------------------------------------------------------------+--------------------+
| status              | The final status of the completed execution.                             | EXECUTION_COMPLETE |
+---------------------+--------------------------------------------------------------------------+--------------------+

The following table lists all events by their `event_type` value and a brief description.

+---------------------+--------------------------------------------------------------------------+
| Event type          | Description                                                              |
+=====================+==========================================================================+
| ACTOR_READY         | The actor is ready to accept messages.                                   |
+---------------------+--------------------------------------------------------------------------+
| ACTOR_ERROR         | The actor is in error status and requires manual intervention.           |
+---------------------+--------------------------------------------------------------------------+
| EXECUTION_COMPLETE  | An actor execution has just completed.                                   |
+---------------------+--------------------------------------------------------------------------+





