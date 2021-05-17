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

For example, if actor A sends messages to actor B, the user can create an alias for actor B and configure A to send
messages to that alias. In the future, if changes need to be made to actor B or if messages from actor A need to be
routed to a different actor, the alias value can be updated without any code changes needed on the part of actor A.

Creating and managing aliases is done via the ``/aliases`` collection.

cURL
~~~~

To create an alias, make a POST request passing the alias and actor id. For example, suppose we have an actor that counts
the words sent in a message. We might create an alias for it with the following:

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

With the alias ``counter`` created, we can now use it in place of the actor id in any Abaco request. For example, we
can get the actor's details:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/counter

The response returned is identical to that returned when the actor id is used.


Nonces Attached to Aliases
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Important::
   Support for Nonces attached to aliases was added in version 1.1.0.


.. Important::
   The nonces attached to aliases feature was updated in version 1.5.0, so that 1) ``UPDATE`` permission on the
   underlying actor id is required and 2) It is no longer possible to create an alias nonce for permission level ``UPDATE``.


Nonces can be created for aliases in much the same way as creating nonces for a specific actor id - instead of using
the ``/nonces`` endpoint associated with the actor id, use the ``/nonces`` endpoint associated with the alias instead. The
POST message payload is the same. For example:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "maxUses=5&level=READ" \
    https://api.tacc.utexas.edu/actors/v2/aliases/counter/nonces

will create a nonce associated with the ``counter`` alias.

.. Note::
  Listing, creating and deleting nonces associated with an alias requires the analagous permission for both the alias
  **and** the associated actor.


Actor Events, Links and WebHooks
--------------------------------

.. Important::
   Support for Actor events, links and webhooks was added in version 1.2.0.

Abaco captures certain events pertaining to the evolution of the system runtime and provides mechanisms for users to
consume these events in actors as well as in external systems.

First, Abaco provides a facility to automatically send a message to a specified actor whenever certain events occur. This
mechanism is called an actor `link`: if actor A is registered with a ``link`` property specifying actor B, then Abaco will
automatically send actor B a message whenever any of the recognized events occurs.

Second, an actor can be registered with a ``webhook`` property: a single string representing a URL to send an HTTP POST
request to. The Abaco events subsystem will send a POST request **exactly once** to the specified URL whenever a
recognized event occurs.

Webhooks and event messages are guaranteed to be delivered in order relative to the order the events occurred for the
specific actor. Since there is no total ordering on events across different actors, there is no analagous order
guarantee.

Links or Webhooks - Which to use?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In both cases, the details of the event are described in a JSON message (sent to an actor in the case of a link, and
sent in the POST payload in the case of a webhook).

However, the actor link is far more general and flexible since
the user can define arbitrary logic to handle the event. Even when the ultimate goal is a webhook, the user may opt for
defining a link to an actor that performs the webhook. This approach enables users to customtize the webhook processing
in various ways, including retry logic, authentication, etc. In fact, the ``abacosamples/webhook`` image provides a
webhook dispatcher built to parse the Abaco events message with many configurable options.

Use of an actor's ``webhook`` property is really intended for simple use cases or situations missed or dropped events
will not cause a major issue.

Adding a Link
~~~~~~~~~~~~~

Registering an actor with a link (or updating an exisitng actor to add a link property) follows the same semantics as
defined in the :ref:`registration` section; simply add the ``link`` attribute in the payload. For example, the following
request creates an actor with a link to actor id ``6PlMbDLa4zlON``.

.. code-block:: bash

  $ curl -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image": "abacosamples/test", "name": "test", "link": "6PlMbDLa4zlON", "description": "My test actor using the abacosamples image.", "default_environment":{"key1": "value1", "key2": "value2"} }' \
  https://api.tacc.utexas.edu/actors/v2

It is also possible to link an actor to an alias: just pass ``link=<the_alias>`` in the registration payload.

.. note::
  Setting a link attribute requires ``EXECUTE`` permission for the associated actor.

.. note::
  Defining a link property that would result in a cycle of linked actors is not permitted, as this would result in
  infinite messages. In particular, an actor cannot link to itself.

Adding a WebHook
~~~~~~~~~~~~~~~~
Registering an actor with a webhook is accomplished similarly by setting the ``webhook`` property in the actor
registration (POST) or update (PUT) payload. For example, the following request creates an actor with a webhook
set to the requestbin at ``https://eniih104j4tan.x.pipedream.net``.

.. code-block:: bash

  $ curl -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image": "abacosamples/test", "name": "test", "webhook": "https://eniih104j4tan.x.pipedream.net", }' \
  https://api.tacc.utexas.edu/actors/v2


Events and Event Message Format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whenever a supported event occurs, Abaco sends a JSON message to the linked actor or webhook with data about the event.
The included data depends on the event type, as documented below.

In the case of a linked actor, all the typical context variables, as
documented in :ref:`context`, will be injected as usual, excepted where noted below. In this case, note that there are
details about two actors: the actor for which the event occurred and the linked actor itself (which are always different,
as self-links are not permitted).
The former is described in the message itself with variables such as ``actor_id``, ``tenant_id``, etc., while the
latter is described using the special reserved Abaco variables, e.g., ``_abaco_actor_id``, etc.

+---------------------+--------------------------------------------------------------------------+--------------------+
| Variable Name       | Description                                                              | Event Type         |
+=====================+==========================================================================+====================+
| actor_id            | The id of the actor for which the event occurred.                        | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| tenant_id           | The id of the tenant of the actor for which the event occurred.          | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| actor_dbid          | The internal id of the actor for which the event occurred.               | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| event_type          | The event type associated with the event. (see table below)              | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| event_time_utc      | The time of the event, in UTC, as a float.                               | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| event_time_display  | The time of the event, as a string, formatted for display.               | all types          |
+---------------------+--------------------------------------------------------------------------+--------------------+
| _abaco_link         | The actor id of the linked actor (the actor receiving the event message) | all types          |
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

The following table lists all events by their ``event_type`` value and a brief description. Additional event types
may be added in subsequent releases.

+---------------------+--------------------------------------------------------------------------+
| Event type          | Description                                                              |
+=====================+==========================================================================+
| ACTOR_READY         | The actor is ready to accept messages.                                   |
+---------------------+--------------------------------------------------------------------------+
| ACTOR_ERROR         | The actor is in error status and requires manual intervention.           |
+---------------------+--------------------------------------------------------------------------+
| EXECUTION_COMPLETE  | An actor execution has just completed.                                   |
+---------------------+--------------------------------------------------------------------------+


Actor Configs
-------------
.. Important::
   Support for Actor configs was added in version 1.9.0.

The actor configs feature allows users to manage a set of conigurations shared by multiple actors all in one place.
Configs can include both standard configuration as well as "secrets" such as database passwords and API keys. With
actor config secrets, Abaco encrypts the config data before saving it in its database.

Actor configs are managed via new endpoint, ``/actors/v2/configs``. Each config object has the following properties:

  * ``name`` - The name of the config. This attribute must be unique within the tenant.
  * ``value`` - The content of the config to be shared with actors. The value must be JSON-serializable.
  * ``actors`` - A comma-separated string of actors to share the config data with. The list can include both actor
    id's and aliases. The user creating the config must have UPDATE access to all actors in the list, as sharing a
    config with an actor is equivalent to updating the actor's default environment.
  * ``isSecret`` (True/False) - Whether the config data should be considered security sensitive. If true, Abaco will
    encrypt the config data (i.e., the contents of ``value``) in the database and decrypt it right before injecting it
    into the actor container. Additionally, when retrieving the config object using Abaco's REST API, Abaco will
    display the encrypted version of the secret data.

Creating Actor Configs
~~~~~~~~~~~~~~~~~~~~~~
Here is an example of creating a simple config using curl:

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs \
  -H "content-type: application/json" \
  -d '{"name": "config_name", "value": "123", "actors": "JBExVooD31rko", "is_secret": false }'

.. warning::

  When creating actor configs be sure to use content type ``application/json``. Using url-encoded forms will lead
  to issues.

In this example, we have shared the config with exactly one actor -- the one with id ``JBExVooD31rko``. We can list or
update the config using its name; for example:

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs/config_name \

  "message": "Config retrieved successfully.",
  "result": {
    "actors": "JBExVooD31rko",
    "is_secret": false,
    "name": "config_name",
    "value": "123"
  },
  "status": "success",
  "version": "1.9.0"

Now, whenever we send actor ``JBExVooD31rko`` a message, Abaco will inject a special environment variable,
``_actor_configs``, into the container, and the value of the variable will be a JSON-serializable representation of all
configs that have been shared with the actor. To be precise, the ``_actor_configs`` variable will be a JSON object with
a key for each such config equal to the config's ``name`` and value equal to the config's ``value``.

For example, assuming this is the only config shared with this actor, the actor container would have an environment
variable ``_actor_configs`` with value:

.. code-block:: bash

  _actor_configs={'config_name': '123'}

We can put any JSON-serializable content for the ``value`` of the config. For example, we could create and share a
second, more complicated config with the same actor as follows:

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs \
  -H "content-type: application/json" \
  -d '{"name": "config2", "value": {"key": "some_key", "int_key": 12345, "a list key": ["a",4, 3.14159]}, "actors": "JBExVooD31rko", "is_secret": false }'

Now when we send a message to actor ``JBExVooD31rko`` the ``_actor_configs`` variable will have contents

.. code-block:: bash

  _actor_configs={'config_name': '123', 'config2': "{'key': 'some_key', 'int_key': 12345, 'a list key': ['a', 4, 3.14159]}"}

Updating Actor Configs
~~~~~~~~~~~~~~~~~~~~~~
Updating an actor config is done by making a PUT request to the ``/actors/v2/configs/<config_name>`` endpoint. A complete
description of the config should be given in the PUT body. For example, to add a new actor to the list of actors that
our simple config from above is shared with, we would make a PUT request like so:

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs/config_name \
  -X PUT \
  -H "content-type: application/json" \
  -d '{"name": "config_name", "value": "123", "actors": "JBExVooD31rko, mr_fixer", "is_secret": false }'

Note that in the above example we have shared the config with both an actor id (``JBExVooD31rko``) and an
alias (``mr_fixer``) which is perfectly allowable.

.. note::

  Updating the ``value`` of an actor config takes effect immediately in the sense that any new actor execution will
  start to use the new value as soon as the PUT request is processed. Thus, actor configs provide a way to update the
  configuration for a set of actors simultaneously, with one API request, instead of updating/redeploying individual
  actors one at a time.


Actor Config Permissions
~~~~~~~~~~~~~~~~~~~~~~~~
It is important to keep in mind that actor config objects have *their own* permissions, separate from the permissions
associated with the actors a config may be shared with. To see and manage the permissions associated with a config,
use the ``/actors/v2/configs/<config_name>/permissions`` endpoint. For example,

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs/config_name/permissions \

    {
      "message": "Permissions retrieved successfully.",
      "result": {
        "testuser": "UPDATE"
      },
      "status": "success",
      "version": "1.9.0"
    }

A user must have explicit access to a config object to read or update it. When a config is first created, only the owner
has access. We can give access to another user by making a POST request to the permissions endpoint, like so:

.. code-block:: bash

  curl -H "Authorization: Bearer $TOKEN" \
  https://api.tacc.utexas.edu/actors/v2/configs/config_name/permissions \
  -H "content-type: application/json" \
  -d '{"user": "testotheruser", "level": "UPDATE"}

    {
      "message": "Permission added successfully.",
      "result": {
        "testotheruser": "UPDATE",
        "testuser": "UPDATE"
      },
      "status": "success",
      "version": "1.9.0"
    }

