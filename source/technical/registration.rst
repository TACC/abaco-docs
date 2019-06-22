.. _registration:

==================
Actor Registration
==================

When registering an actor, the only required field is a reference to an image on the public Docker Hub. However,
there are several other properties that can be set. The following table provides a complete list of properties and
their descriptions.

+---------------------+----------------------------------------------------------------------------------+
| Property Name       | Description                                                                      |
+=====================+==================================================================================+
| image               | The Docker image to associate with the actor. This should be a fully qualified   |
|                     | image available on the public Docker Hub. We encourage users to use to image     |
|                     | tags to version control their actors.                                            |
+---------------------+----------------------------------------------------------------------------------+
| name                | A user defined name for the actor.                                               |
+---------------------+----------------------------------------------------------------------------------+
| description         | A user defined description for the actor.                                        |
+---------------------+----------------------------------------------------------------------------------+
| default_environment | The default environment is a set of key/value pairs to be injected into every    |
|                     | execution of the actor. The values can also be overridden when passing a         |
|                     | message to the reactor in the query parameters (see :ref:`messages`).            |
+---------------------+----------------------------------------------------------------------------------+
| stateless           | (True/False) - Whether the actor stores private state as part of its execution.  |
|                     | If True, the state API will not be available, but in a future release, the       |
|                     | Abaco service will be able to automatically scale reactor processes to execute   |
|                     | messages in parallel. The default value is False.                                |
+---------------------+----------------------------------------------------------------------------------+
| privileged          | (True/False) - Whether the actor runs in privileged mode and has access to       |
|                     | the Docker daemon. *Note*: Setting this parameter to True requires elevated      |
|                     | permissions.                                                                     |
+---------------------+----------------------------------------------------------------------------------+
| use_container_uid   | Run the actor using the UID/GID set in the Docker image. *Note*: Setting         |
|                     | this parameter to True requires elevated permissions.                            |
+---------------------+----------------------------------------------------------------------------------+

Notes
-----

- The ``default_environment`` can be used to provide sensitive information to the actor that cannot be put in the image.
- In order to execute privileged actors or to override the UID/GID used when executing an actor container,
  talk to the Abaco development team about your use case.

Examples
--------

curl
~~~~

Here is an example using curl; note that to set the default environment, we *must* pass content type ``application/json`` and
be sure to pass properly formatted JSON in the payload.

.. code-block:: bash

  $ curl -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image": "abacosamples/test", "name": "test", "description": "My test actor using the abacosamples image.", "default_environment":{"key1": "value1", "key2": "value2"} }' \
  https://api.tacc.utexas.edu/actors/v2


Python
~~~~~~

To register the same actor using the agavepy library, we use the ``actors.add()`` method and pass the same arguments
through the `body` parameter. In this case, the ``default_environment`` is just a standard Python dictionary where the
keys and values are ``str`` type. For example,

.. code-block:: bash

  >>> from agavepy.agave import Agave
  >>> ag = Agave(api_server='https://api.tacc.utexas.edu', token='<access_token>')
  >>> actor = {"image": "abacosamples/test",
               "name": "test",
               "description": "My test actor using the abacosamples image registered using agavepy.",
               "default_environment":{"key1": "value1", "key2": "value2"} }
  >>> ag.actors.add(body=actor)


