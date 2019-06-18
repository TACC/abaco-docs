.. _sharing:

========================
Actor Sharing and Nonces
========================

Abaco provides a basic permissions system for securing actors. An actor registered with Abaco starts out as private
and only accessible to the API user who registered it. This API user is referred to as the "owner" of the actor.
By making a POST request to the permissions endpoint for an actor, a user can manage the list of API users who have
access to the actor.

-----------------
Permission Levels
-----------------

Abaco supports sharing an actor at three different permission levels; in increasing order, they are: `READ`,
`EXECUTE` and `UPDATE`. Higher permission imply lower permissions, so a user with `EXECUTE` also has `READ` while a
user with `UPDATE` has `EXECUTE` and `READ`. The permission levels provide the followig accesses:

  * `READ` - ability to list the actor to see it's details, list executions and retrieve execution logs.
  * `EXECUTE` - ability to send an actor a message.
  * `UPDATE` - ability to change the actor's definition.


cURL
----

To share an actor with another API user, make a POST request to the `/permissions` endpoint; the following example
uses curl to grant READ permission to API user `jdoe`.

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "user=jdoe&level=READ" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/permissions

Example response:

.. code-block:: bash

    {
      "message": "Permission added successfully.",
      "result": {
        "jdoe": "READ",
        "testuser": "UPDATE"
      },
      "status": "success",
      "version": "1.0.0"
    }


We can list all permissions associated with an actor at any time using a GET request:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/permissions

Example response:

.. code-block:: bash

    {
      "message": "Permissions retrieved successfully.",
      "result": {
        "jdoe": "READ",
        "jsmith": "EXECUTE",
        "testuser": "UPDATE"
      },
      "status": "success",
      "version": "1.0.0"
    }

.. Note::
  To remove a user's permission, POST to the permission endpoint and set `level=NONE`


-------------
Public Actors
-------------

At times, it can be useful to grant **all** API users access to an actor. To enable this, Abaco recognizes the special
ABACO_WORLD user. Granting a permission to the ABACO_WORLD user will effectively grant the permission to all API users.


cURL
----

The following grants `READ` permission to all API users:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "user=ABACO_WORLD&level=READ" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/permissions

------
Nonces
------

Abaco provides a capability referred to as actor *nonces* to ease integration with third-party systems leveraging
different authentication mechanisms. An actor `nonce` can be used in place of the typical TACC API access token
(bearer token). However, unlike an access token which can be used for any actor the user has access, a nonce can only be
used for a specific actor.

Creating Nonces
---------------

API users create nonces using the nonces endpoint associated with an actor. Nonces can be limited to a specific
permission level (e.g., `READ` only), and can have a finite number of uses or an unlimited number.

The following example uses curl to create a nonce with `READ` level permission and with 5 uses.

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "maxUses=5&level=READ" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/nonces

A typical response:

.. code-block:: bash

    {
      "message": "Actor nonce created successfully.",
      "result": {
        "_links": {
          "actor": "https://api.tacc.utexas.edu/actors/v2/rNjQG5BBJoxO1",
          "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
          "self": "https://api.tacc.utexas.edu/actors/v2/rNjQG5BBJoxO1/nonces/DEV_qBMrvO6Zy0yQz"
        },
        "actorId": "rNjQG5BBJoxO1",
        "apiServer": "http://172.17.0.1:8000",
        "createTime": "2019-06-18 12:20:53.087704",
        "currentUses": 0,
        "description": "",
        "id": "TACC_qBMrvO6Zy0yQz",
        "lastUseTime": "None",
        "level": "READ",
        "maxUses": 5,
        "owner": "testuser",
        "remainingUses": 5,
        "roles": [
          "Internal/everyone",
          "Internal/AGAVEDEV_testuser_postman-test-client-1497902074_PRODUCTION",
          "Internal/AGAVEDEV_testuser_postman-test-client-1494517466_PRODUCTION",
       ]
      },
      "status": "success",
      "version": "1.0.0"
    }


The `id` of the nonce (in the above example, `TACC_qBMrvO6Zy0yQz`) can be used to access the actor in place of the
access token.

.. Note::
  Roles are used throughout the TACC API's to grant users with specific privileges (e.g., administrative access to certain
  APIs). The roles of the API user generating the nonce are captured at the time the nonce is created; when using a nonce,
  a request will have permissions granted via those roles. Most users will not need to worry about TACC API roles.

To create a nonce with unlimited uses, set `maxUses=-1`.


Redeeming Nonces
----------------

To use a nonce in place of an access token, simply form the request as normal and add the query paramter `x-nonce=<nonce_id>`.

For example

.. code-block:: bash

    $ curl -X POST -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages?x-nonce=TACC_vr9rMO6Zy0yHz

The response will be exactly the same as if issuing the request with an access token.