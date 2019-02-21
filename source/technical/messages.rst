
.. _messages:

==============================
Executions, Messages, and Logs
==============================

Once you have an Abaco actor created the next logical step is to send this actor
some type of job or message detailing what the actor should do. The act of sending
an actor information to execute a job is called sending a message. This sent
message can be raw string data, JSON data, or a binary message.

Once a message is sent to an Abaco actor, the actor will create an execution with
a unique ``execution_id`` tied to it that will show results, time running, and other
stats which will be listed below. Executions also have logs, and when the log are 
called for, you'll receive the command line logs of your running execution.
Akin to what you'd see if you and outputted a script to the command line.
Details on messages, executions, and logs are below.

----

--------
Messages
--------

A message is simply the message given to an actor with data that can be used to run
the actor. This data can be in the form of a raw message string, JSON, or binary.
Once this message is sent, the messaged Abaco actor will queue an execution of 
the actor's specified image.

Once off the queue, if your specified image has inputs for the messaged data, 
then that messaged data will be visible to your program. Allowing you to set
custom parameters or inputs for your executions.

Message Properties
^^^^^^^^^^^^^^^^^^

+---------+----------------------------+
| Results | Descriptions               |
+---------+----------------------------+
| message | Message from abaco         |
+---------+----------------------------+
| result  | Some content, shown below. |
+---------+----------------------------+
| status  | Don't know                 |
+---------+----------------------------+
| version | Don't know                 |
+---------+----------------------------+

Examples
^^^^^^^^

cURL
----

To message an actor with the use of cURL you would do the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages

Python
------

To message an actor with Agave in Python you would do the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages

Results
-------

These calls result in a json list similar to the following:

.. code-block:: bash

    {'message': 'The request was successful',
     'result': {'_links': {'messages': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/messages',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/cgarcia',
       'self': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/executions/00wLaDX53WBAr'},
      'executionId': '00wLaDX53WBAr',
      'msg': '<your content here>'},
     'status': 'success',
     'version': '0.11.0'}

----

----------
Executions
----------

Once you send a message to an actor, that actor will create an execution for the actor
with the inputted data. This execution will be queued waiting for a worker to spool up
or waiting for a worker to be freed. When the execution is initially created it is
given an execution_id so that you can access information about it using the execution_id endpoint.

Examples
^^^^^^^^

cURL
----

You can access the ``execution_id`` endpoint with cURL with the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>

Python
------

You can access the ``execution_id`` endpoint with ``AgavePy`` and Python with the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>


Results
-------

These executions will result with something similar to the following:

.. code-block:: bash
    
    {'message': 'Actor execution retrieved successfully.',
     'result': {'_links': {'logs': 'https://api.tacc.utexas.edu/actors/v2/TACC-PROD_R0y3eYbWmgEwo/executions/00wLaDX53WBAr/logs',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/cgarcia',
       'self': 'https://api.tacc.utexas.edu/actors/v2/TACC-PROD_R0y3eYbWmgEwo/executions/00wLaDX53WBAr'},
      'actorId': 'R0y3eYbWmgEwo',
      'apiServer': 'https://api.tacc.utexas.edu',
      'cpu': 7638363913,
      'executor': 'cgarcia',
      'exitCode': 1,
      'finalState': {'Dead': False,
       'Error': '',
       'ExitCode': 1,
       'FinishedAt': '2019-02-21T17:32:18.56680737Z',
       'OOMKilled': False,
       'Paused': False,
       'Pid': 0,
       'Restarting': False,
       'Running': False,
       'StartedAt': '2019-02-21T17:32:14.893485694Z',
       'Status': 'exited'},
      'id': '00wLaDX53WBAr',
      'io': 124776656,
      'messageReceivedTime': '2019-02-21 17:31:24.300900',
      'runtime': 11,
      'startTime': '2019-02-21 17:32:12.798836',
      'status': 'COMPLETE',
      'workerId': 'oQpeybmGRVNyB'},
     'status': 'success',
     'version': '0.11.0'}

----
 
----
Logs
----

At any point of an execution you are also able to access the execution logs
using the ``logs`` endpoint. This returns information
about the log along with the log itself. If the execution is still in the
submitted phase, then the log will be an empty string, but once the execution
is in the completed phase the log would contain all outputted command line data.

Examples
^^^^^^^^

cURL
----

To retrieve the log using cURL, do the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>

Python
------

To retrieve the log using cURL, do the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>

Results
-------

This request would result in the following returned data:

.. code-block:: bash

    {'message': 'Logs retrieved successfully.',
     'result': {'_links': {'execution': 'https://api.tacc.utexas.edu/actors/v2/qgKRpNKxg0DME/executions/qgmq08wKARlg3',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/cgarcia',
       'self': 'https://api.tacc.utexas.edu/actors/v2/qgKRpNKxg0DME/executions/qgmq08wKARlg3/logs'},
      'logs': ''},
     'status': 'success',
     'version': '0.11.0'}

----

--------------------
Reading data from Code
--------------------

One of the most important parts of using data in an execution is reading said
data. Retrieving sent data depends on the data sent.

Python - Reading in Raw String Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To get a raw message from inside of a Python using ``Agavepy`` you would get the
message context from within the actor and then get it's ``raw_message`` field. 

.. code-block:: bash
	
	from agavepy.actors import get_context

	context = get_context()
	message = context['raw_message']

Python - Reading in JSON
^^^^^^^^^^^^^^^^^^^^^^^^

Python - Reading in Binary
^^^^^^^^^^^^^^^^^^^^^^^^^^

As binary data transmitted to a execution is streamed in through a FIFO pipe,
you'll need to read the pipe using the ``AgavePy`` ``get_binary_message()``
function.

.. code-block:: bash

	from agavepy.actors import get_binary_message

	bin_message = get_binary_message()

----

---------------
Binary Messages
---------------

An additional feature of the Abaco message system is the ability to post binary
data. This data, unlike a the raw string data is sent through a ... and is got
from within the execution using a FIFO message reading function. The ability to
read binary data like this allows our end users to do numerous task, read in
photos, read in code to be ran, and much more.

The following is an example of sending a jpeg as a binary message in order to
be read in by a TensorFlow image classifier and being returned possible image
labels. For example, sending a photo of a golden retriever might yield, 80%
golden retriever, 12% labrador, and 8% clock.

Example
^^^^^^^

Python
------

The following is how to create a binary message from a .jpeg image file

.. code-block:: bash

    with open("<path to jpeg image here>", 'rb') as file:
        raw_binary_data = file.read()

    message = raw_binary_data

The following example on sending the binary image uses Python's requests library.

.. code-block:: bash

    import requests

    url = "https://api.tacc.utexas.edu"
    token = <set your api token here>

The following example on sending the binary image uses Python's requests library.

.. code-block:: bash

    actor = requests.post("{}/actors/v2".format(url),
                          headers={'Authorization':'Bearer {}'.format(token)},
                          data={'image':'notchristiangarcia/bin_classifier'})
    actor_id = actor.json()['result']['id']

The following example on sending the binary image uses Python's requests library.

.. code-block:: bash

    bin_exec = requests.post("{}/actors/v2/{}/messages".format(url, actor_id), 
                             headers={'Authorization': 'Bearer {}'.format(token),
                                      'Content-Type': 'application/octet-stream'},
                             data=message)
    exec_id = bin_exec.json()['result']['executionId']

The following example on sending the binary image uses Python's requests library.

.. code-block:: bash

    exec_info = requests.get("{}/actors/v2/{}/executions/{}".format(url, actor_id, exec_id),
                             headers={'Authorization': 'Bearer {}'.format(token)})

The following example on sending the binary image uses Python's requests library.

.. code-block:: bash
    exec_logs = requests.get("{}/actors/v2/{}/executions/{}/logs".format(url, actor_id, exec_id),
                             headers={'Authorization': 'Bearer {}'.format(token)})
    exec_logs.json()['result']['logs']

----

----
Misc
----

In order to lists all executions for the given actor id you would make a call
to an actors executions endpoint:

cURL
^^^^

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    https://api.tacc.utexas.edu/actors/<actor_id>/executions

To get execution information you would GET the executions ID:

cURL
^^^^

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    https://api.tacc.utexas.edu/actors/<actor_id>/executions/<executionId>

To receive the logs of an execution you would GET the logs endpoint of an execution ID:

cURL
^^^^

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    https://api.tacc.utexas.edu/actors/<actor_id>/executions/<executionId>/logs

Agave
^^^^^

.. code-block:: bash

    ag.actors.sendMessage(actorId='O08Nzb3mRA7Bz',
                          body={'message': 'Actor, please count these words.'})





ACTOR STUFF

cURL
^^^^

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    https://api.tacc.utexas.edu/actors

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    https://api.tacc.utexas.edu/actors

Creating an actor.

cURL
^^^^

.. code-blocl:: bash

    curl -H "X-Jwt-Assertion-TEST: $jwt" localhost:8000/actors -d 'image=abacosamples/test' -d 'description="Add your description here"'

    .. attention::
    While ``agavepy`` works with both Python 2 and 3 we strongly recommend using Python 3.
