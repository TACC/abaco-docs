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
mechanism is called an actor `link`: if actor A is registered with a `link` property specifing actor B, then Abaco will
automatically send actor B a message whenever any of the following events occur:

  * Actor A's status changes (for example, from SUBMITTED to READY or from READY to ERROR).
  * An execution for actor A completes.







