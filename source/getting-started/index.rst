
.. _getting-started:

===============
Getting Started
===============

This Getting Started guide will walk you through the initial steps of setting up the necessary accounts and installing
the required software before moving to the Abaco Quickstart, where you will create and execute your first Abaco actor. If
you are already using Docker Hub and the TACC Cloud APIs, feel free to jump right to the `Abaco Quickstart`_.


.. contents:: :local:

------------------------------------------
Account Creation and Software Installation
------------------------------------------

Create a TACC account
^^^^^^^^^^^^^^^^^^^^^

The main instance of the Abaco platform is hosted at the Texas Advanced Computing Center (`TACC <https://tacc.utexas.edu>`_).
TACC designs and deploys some of the world's most powerful advanced computing technologies and innovative software
solutions to enable researchers to answer complex questions. To use the TACC-hosted Abaco service, please
create a `TACC account <https://portal.tacc.utexas.edu/account-request>`__ .

Create a Docker account
^^^^^^^^^^^^^^^^^^^^^^^

`Docker <https://www.docker.com/>`__  is an open-source container runtime providing operating-system-level
virtualization. Abaco pulls images for its actors from the public Docker Hub. To register actors
you will need to publish images on Docker Hub, which requires a `Docker account <https://hub.docker.com/>`__ .


Install the TACC Cloud Python SDK
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To interact with the TACC-hosted Abaco platform in Python, we will leverage the TACC Cloud Python SDK. To install it,
simply run:

.. code-block:: bash

  $ pip3 install agavepy

.. attention::
    While ``agavepy`` works with both Python 2 and 3 we strongly recommend using Python 3.


-----------------------
Working with TACC OAuth
-----------------------

Authentication and authorization to the TACC Cloud APIs uses `OAuth2 <https://oauth.net/2/>`_, a widely-adopted web standard.
Our implementation of OAuth2 is designed to give you the flexibility you need to script and automate use of TACC
Cloud while keeping your access credentials and digital assets secure. This is covered in great detail in our
Developer Documentation but some key concepts will be highlighted here, interleaved with Python code.


Create an OAuth Client
^^^^^^^^^^^^^^^^^^^^^^

The first step is to create an OAuth client. This is a one-time set up step, much like creating a TACC account. To do
it, we will use the TACC Cloud API Python SDK. First, import the ``Agave`` class and create python object called ``ag``
that points to the TACC Cloud API server using your TACC username and password. Do so by typing the following in
a Python shell:

.. code-block:: bash

  >>> from agavepy.agave import Agave
  >>> ag = Agave(api_server='https://api.tacc.utexas.edu',
  ...            username='your username',
  ...            password='your password')

Once the object is instantiated, interact with it according to the API documentation and your specific usage needs.
For example, to create a new OAuth client we type the following:

.. code-block:: bash

  >>> ag.clients.create(body={'clientName': 'enter a client name'})

You should see a response like:

.. code-block:: bash

  {'_links': {'self': {'href': 'https://api.tacc.utexas.edu/clients/v2/abaco_quickstart'},
   'subscriber': {'href': 'https://api.tacc.utexas.edu/profiles/v2/apitest'},
   'subscriptions': {'href': 'https://api.tacc.utexas.edu/clients/v2/abaco_quickstart/subscriptions/'}},
   'callbackUrl': '',
   'consumerKey': 'pYV81QNBxkqeC6Nms3XBzk9UJuca',
   'consumerSecret': 'Oug0gdLa3a_Xt37_fwxO6ZGNffUa',
   'description': '',
   'name': 'abaco_quickstart',
   'tier': 'Unlimited'}


Record the consumerKey and consumerSecret in a secure place; you will use them over and over to generate Oauth tokens,
which are temporary credentials that you can use in place of putting your real credentials into code that
is scripting against the TACC APIs.


Reuse an Existing Oauth Client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once you generate an OAuth client, you can re-use its key and secret:

.. code-block:: bash

  >>> from agavepy.agave import Agave
  >>> ag = Agave(api_server='https://api.tacc.utexas.edu',
  ...            username='your username', password='your password',
  ...            client_name='my_client',
  ...            api_key='pYV81QNBxkqeC6Nms3XBzk9UJuca',
  ...            api_secret='Oug0gdLa3a_Xt37_fwxO6ZGNffUa')


Generate a Token
^^^^^^^^^^^^^^^^

With the ``ag`` object instantiated and an OAuth client created, we are ready to generate an OAuth token:

 .. code-block:: bash

  >>> ag.token.create()
  Out[1]: 'c21199177da6dd4d14d659399a933f5'

Note that the token is automatically stored on the ``ag`` object for you. You are now ready to check your access to the
TACC Cloud APIs.

Check Access to the TACC Cloud APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The Agave object ``ag`` should now be configured to talk to all TACC Cloud APIs on your behalf. We can check that
our client is configured properly by making any API call. Here's an example: Let's retrieve the current
user's **profile**.

.. code-block:: bash

  >>> ag.profiles.get()
  Out[1]:
  {'email': 'aci-cic@tacc.utexas.edu',
   'first_name': 'API',
   'full_name': 'API Test',
   'last_name': 'Test',
   'mobile_phone': '',
   'phone': '',
   'status': '',
   'uid': 834517,
   'username': 'apitest'}


----------------
Abaco Quickstart
----------------

In this Quickstart, we will create an Abaco actor from a basic Python function. Then we will execute our actor on the
Abaco cloud and get the execution results.

A Basic Python Function
^^^^^^^^^^^^^^^^^^^^^^^

Suppose we want to write a Python function that counts words in a string. We might write something like this:

.. code-block:: bash

  def string_count(message):
      words = message.split(' ')
      word_count = len(words)
      print('Number of words is: ' + str(word_count))

In order to process a message sent to an actor, we use the ``raw_message`` attribute of the ``context`` dictionary.
We can access it by using the ``get_context`` method from the ``actors`` module in ``agavepy``.

For this example, create a new local directory to hold your work. Then, create a new file in this directory called
``example.py``. Add the following to this file:

.. code-block:: bash

  # example.py

  from agavepy.actors import get_context

  def string_count(message):
      words = message.split(' ')
      word_count = len(words)
      print('Number of words is: ' + str(word_count))

  context = get_context()
  message = context['raw_message']
  string_count(message)


Building Images From a Dockerfile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To register this function as an Abaco actor, we create a docker image that contains the python function and
execute it as part of the default command.

We can build a Docker image from a text file called a Dockerfile. You can think of a Dockerfile as a recipe for
creating images. The instructions within a Dockerfile either add files/folders to the image, add metadata to the
image, or both.


The FROM Instruction
~~~~~~~~~~~~~~~~~~~~

Create a new file called ``Dockerfile`` in the same directory as your ``example.py`` file.

We can use the ``FROM`` instruction to start our new image from a known image. This should be the first line of our
Dockerfile. We will start an official Python image:

.. code-block:: bash

  FROM python:3.6

The RUN, ADD and CMD Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can run arbitrary Linux commands to add files to our image. We'll run the ``pip`` command to install the ``agavepy``
library in our image:

.. code-block:: bash

  RUN pip install --no-cache-dir agavepy

(note: there is a ``abacosample`` image that contains Python and the agavepy library; see :ref:`samples` for more
details)

We can also add local files to our image using the ``ADD`` instruction. To add the ``example.py`` file from our local
directory, we use the following instruction:

.. code-block:: bash

  ADD example.py /example.py

The last step is to write the command from running the application, which is simply ``python /example.py``. We use
the ``CMD`` instruction to do that:

.. code-block:: bash

  CMD ["python", "/example.py"]

With that, our ``Dockerfile`` is now ready. This is what is looks like:

.. code-block:: bash

  FROM python:3.6

  RUN pip install --no-cache-dir agavepy
  ADD example.py /example.py

  CMD ["python", "/example.py"]


Now that we have our ``Dockerfile``, we can build our image and push it to Docker Hub. To do so, we use the
``docker build`` and ``docker push`` commands [note: user is your user on Docker, you must also $ docker login] :

.. code-block:: bash

  $ docker build -t user/my_actor .
  $ docker push user/my_actor

Registering an Actor
^^^^^^^^^^^^^^^^^^^^

Now we are going to register the Docker image we just built as an Abaco actor. To do this, we will use the ``Agave``
client object we created above (see `Working with TACC OAuth`_).

To register an actor using the agavepy library, we use the ``actors.add()`` method and pass the arguments describing
the actor we want to register through the ``body`` parameter. For example:

.. code-block:: bash

  >>> from agavepy.agave import Agave
  >>> ag = Agave(api_server='https://api.tacc.utexas.edu', token='<access_token>')
  >>> my_actor = {"image": "user/my_actor", "name": "word_counter", "description": "Actor that counts words."}
  >>> ag.actors.add(body=my_actor)

You should see a response like this:

.. code-block:: bash

    {'_links': {'executions': 'https://api.tacc.utexas.edu/actors/v2/O08Nzb3mRA7Bz/executions',
    'owner': 'https://api.tacc.utexas.edu/profiles/v2/jstubbs',
    'self': 'https://api.tacc.utexas.edu/actors/v2/O08Nzb3mRA7Bz'},
    'createTime': '2018-07-03 22:41:29.563024',
    'defaultEnvironment': {},
    'description': 'Actor that counts words.',
    'id': 'O08Nzb3mRA7Bz',
    'image': 'abacosamples/wc',
    'lastUpdateTime': '2018-07-03 22:41:29.563024',
    'mounts': [],
    'name': 'word_counter',
    'owner': 'jstubbs',
    'privileged': False,
    'state': {},
    'stateless': False,
    'status': 'SUBMITTED',
    'statusMessage': '',
    'type': 'none',
    'useContainerUid': False}

Notes:

- Abaco assigned an id to the actor (in this case ``O08Nzb3mRA7Bz``) and associated it with the image (in this case,
  ``abacosamples/wc``) which it began pulling from the public Docker Hub.
- Abaco returned a status of ``SUBMITTED`` for the actor; behind the scenes, Abaco is starting a worker container to
  handle messages passed to this actor. The worker must initialize itself (download the image, etc) before the
  actor is ready.
- When the actor's worker is initialized, the status will change to ``READY``.

At any point we can check the details of our actor, including its status, with the following:

.. code-block:: bash

  >>> ag.actors.get(actorId='O08Nzb3mRA7Bz')

The response format is identical to that returned from the ``.add()`` method.


Executing an Actor
^^^^^^^^^^^^^^^^^^

We are now ready to execute our actor by sending it a message. We built our actor to process a raw message string, so
that is what we will send, but there other options, including JSON and binary data. For more details, see the
:ref:`messages` section.

We send our actor a message using the ``sendMessage()`` method:

.. code-block:: bash

  >>> ag.actors.sendMessage(actorId='O08Nzb3mRA7Bz',
                            body={'message': 'Actor, please count these words.'})

Abaco queues up an execution for our actor and then responds with JSON, including an id for the execution contained in
the ``executionId``:

.. code-block:: bash

    {'_links': {'messages': 'https://api.tacc.utexas.edu/actors/v2/O08Nzb3mRA7Bz/messages',
      'owner': 'https://api.tacc.utexas.edu/profiles/v2/jstubbs',
      'self': 'https://api.tacc.utexas.edu/actors/v2/O08Nzb3mRA7Bz/executions/kA1P1m8NkkolK'},
     'executionId': 'kA1P1m8NkkolK',
     'msg': 'Actor, please count these words.'}

In general, an execution does not start immediately but is instead queued until a future time when a worker for the
actor can take the message and start an actor container with the message. We can retrieve the details about an
execution, including its status, using the ``getExecution()`` method:

.. code-block:: bash

  >>> ag.actors.getExecution(actorId='O08Nzb3mRA7Bz', executionId='kA1P1m8NkkolK')

The response will be similar to the following:

.. code-block:: bash

    {'_links': {'logs': 'https://api.tacc.utexas.edu/actors/v2/TACC-PROD_O08Nzb3mRA7Bz/executions/kA1P1m8NkkolK/logs',
      'owner': 'https://api.tacc.utexas.edu/profiles/v2/jstubbs',
      'self': 'https://api.tacc.utexas.edu/actors/v2/TACC-PROD_O08Nzb3mRA7Bz/executions/kA1P1m8NkkolK'},
     'actorId': 'O08Nzb3mRA7Bz',
     'apiServer': 'https://api.tacc.utexas.edu',
     'cpu': 0,
     'executor': 'jstubbs',
     'exitCode': 1,
     'finalState': {'Dead': False,
      'Error': '',
      'ExitCode': 1,
      'FinishedAt': '2018-07-03T22:56:30.605256563Z',
      'OOMKilled': False,
      'Paused': False,
      'Pid': 0,
      'Restarting': False,
      'Running': False,
      'StartedAt': '2018-07-03T22:56:30.474917256Z',
      'Status': 'exited'},
     'id': 'kA1P1m8NkkolK',
     'io': 0,
     'messageReceivedTime': '2018-07-03 22:56:29.075122',
     'runtime': 1,
     'startTime': '2018-07-03 22:56:29.558470',
     'status': 'COMPLETE',
     'workerId': 'e7B3JXDNxM6M0'}

Note that a status of ``COMPLETE`` indicates that the execution has finished and we are ready to retrieve our results.


Retrieving the Logs
^^^^^^^^^^^^^^^^^^^

The Abaco system collects all standard out from an actor execution and makes it available via the ``logs`` endpoint.
Let's retrieve the logs from the execution we just made. We use the ``getExecutionLogs()``
method, passing out ``actorId`` and our ``executionId``:

.. code-block:: bash

  >>> ag.actors.getExecutionLogs(actorId='O08Nzb3mRA7Bz', executionId='kA1P1m8NkkolK')

The response should be similar to the following:

.. code-block:: bash

    {'_links': {'execution': 'https://api.tacc.utexas.edu/actors/v2/6PlMbDLa4zlON/executions/kGQk6RRJQBL3',
      'owner': 'https://api.tacc.utexas.edu/profiles/v2/jstubbs',
      'self': 'https://api.tacc.utexas.edu/actors/v2/6PlMbDLa4zlON/executions/kGQk6RRJQBL3/logs'},
     'logs': 'Number of words is: 5\n'}

We see our actor output `Number of words is: 5`, which is the expected result!


Conclusion
^^^^^^^^^^

Congratulations! At this point you have created, registered and executed your first actor, but there is a lot more you
can do with the Abaco system. To learn more about the additional capabilities, please continue on to the Technical Guide.
