
.. _messages:

==============================
Messages, Executions, and Logs
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

**Note:** Due to each message being tied to a specific execution, each execution
will have exactly one message that can be processed.

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

Sending a message
^^^^^^^^^^^^^^^^^

cURL
----

To send a message to the ``messages`` endpoint with cURL, you would do the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages

Python
------

To send a message to the ``messages`` endpoint with ``AgavePy`` and Python, you would do the following:

.. code-block:: python

    ag.actors.sendMessage(actorId='<actor_id>',
                          body={'message':'<your content here>'})

Results
-------

These calls result in a JSON list similar to the following:

.. code-block:: bash

    {'message': 'The request was successful',
     'result': {'_links': {'messages': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/messages',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/apitest',
       'self': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/executions/00wLaDX53WBAr'},
      'executionId': '00wLaDX53WBAr',
      'msg': '<your content here>'},
     'status': 'success',
     'version': '0.11.0'}

Get message count
^^^^^^^^^^^^^^^^^

It is possible to retrieve the current number of messages an actor has with the
``messages`` end point.

cURL
----

The following retrieves the current number of messages an actor has:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages

Python
------

To retrieve the current number of messages with ``AgavePy`` the following is done:

.. code-block:: python

    ag.actors.getMessages(actorId='<actor_id>')

Results
-------

The result of getting the ``messages`` endpoint should be similar to:

.. code-block:: bash

    {'message': 'The request was successful',
     'result': {'_links': {'owner': 'https://api.tacc.utexas.edu/profiles/v2/cgarcia',
       'self': 'https://api.tacc.utexas.edu/actors/v2/R4OR3KzGbRQmW/messages'},
      'messages': 12},
     'status': 'success',
     'version': '0.11.0'}

Binary Messages
^^^^^^^^^^^^^^^

An additional feature of the Abaco message system is the ability to post binary
data. This data, unlike raw string data, is sent through a Unix Named Pipe
(FIFO), stored at /_abaco_binary_data, and can be retrieved from within the
execution using a FIFO message reading function. The ability to read binary
data like this allows our end users to do numerous tasks such as reading in
photos, reading in code to be ran, and much more.

The following is an example of sending a JPEG as a binary message in order to
be read in by a TensorFlow image classifier and being returned predicted image
labels. For example, sending a photo of a golden retriever might yield, 80%
golden retriever, 12% labrador, and 8% clock.

This example uses Python and ``AgavePy`` in order to keep code in one script.

Python with AgavePy
-------------------

Setting up an ``AgavePy`` object with token and API address information:

.. code-block:: python

    from agavepy.agave import Agave
    ag = Agave(api_server='https://api.tacc.utexas.edu',
               username='<username>', password='<password>',
               client_name='JPEG_classifier',
               api_key='<api_key>',
               api_secret='<api_secret>')

    ag.get_access_token()
    ag = Agave(api_server='https://api.tacc.utexas.edu/', token=ag.token)

Creating actor with the TensorFlow image classifier docker image:

.. code-block:: python

    my_actor = {'image': 'notchristiangarcia/bin_classifier',
                'name': 'JPEG_classifier',
                'description': 'Labels a read in binary image'}
    actor_data = ag.actors.add(body=my_actor)

The following creates a binary message from a JPEG image file:

.. code-block:: python
    
    with open('<path to jpeg image here>', 'rb') as file:
        binary_image = file.read()

Sending binary JPEG file to actor as message with the ``application/octet-stream`` header:

.. code-block:: python

    result = ag.actors.sendMessage(actorId=actor_data['id'],
                                   body={'binary': binary_image},
                                   headers={'Content-Type': 'application/octet-stream'})

The following returns information pertaining to the execution:

.. code-block:: python

    execution = ag.actors.getExecution(actorId=actor_data['id'],
                                       executionId = result['executionId'])

Once the execution has complete, the logs can be called with the following:

.. code-block:: python
    
    exec_info = requests.get('{}/actors/v2/{}/executions/{}'.format(url, actor_id, exec_id),
                             headers={'Authorization': 'Bearer {}'.format(token)})

Sending binary from execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Another useful feature of Abaco is the ability to write to a socket connected 
to an Abaco endpoint from within an execution. This Unix Domain (Datagram)
socker is mounted in the actor container at /_abaco_results.sock.

In order to write binary data this socket you can use ``AgavePy`` functions,
in particular the ``send_bytes_result()`` function that sends bytes as single
result to the socket. Another useful function is the ``send_python_result()``
function that allows you to send any Python object that can be pickled with
``cloudpickle``.

In order to retrieve these results from Abaco you can get the 
``/actor/<actor_id>/executions/<execution_id>/results`` endpoint. Each get of
the endpoint will result in exactly one result being popped and retrieved. An
empty result with be returned if the results queue is empty.

As a socket, the maximum size of a result is 131072 bytes. An execution can
send multiple results to the socket and said results will be added to a queue.
It is recommended to to return a reference to a file or object store.

As well, results are sent to the socket and available immediately, an execution
does not have to complete to pop a result. Results are given an expiry time of
60 minutes from creation.

cURL
----

To retrieve a result with cURL you would do the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>/results

----

Synchronous Messaging
^^^^^^^^^^^^^^^^^^^^^

.. Important::
   Support for Synchronous Messaging was added in version 1.1.0.

Starting with `1.1.0`, Abaco provides support for sending a synchronous message to an actor; that is, the client
sends the actor a message and the request blocks until the execution completes. The result of the execution is returned
as an HTTP response to the original message request.

Synchronous messaging prevents the client from needing to poll the executions endpoint to determine when an execution
completes. By eliminating this polling and returning the response as soon as it is ready, the overall latency
is minimized.

While synchronous messaging can simplify client code and improve performance, it introduces some additional challenges.
Primarily, if the execution cannot be completed within the HTTP request/response window, the request will time out.
This window is usually about 30 seconds.

.. Warning::
  Abaco strictly adheres to message ordering and, in particular, synchronous messages do not skip to the front of the
  actor's message queue. Therefore, a synchronous message *and all queued messages* must be processed within the HTTP
  timeout window. To avoid excessive synchronous message requests, Abaco will return a 400 level request if the actor
  already has more than 3 queued messages at the time of the synchronous message request.

To send a synchronous message, the client appends `_abaco_synchronous=true` query parameter to the request; the rest of
the messaging semantics follows the rules and conventions of asynchronous messages.

cURL
----

The following example uses the curl command line client to send a synchronous message:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -d "message=<your content here>" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/messages?_abaco_synchronous=true

As stated above, the request blocks until the execution (and all previous executions queued for the actor) completes.
To make the response to a synchronous message request, Abaco uses the following rules:

  1. If a (binary) result is registered by the actor for the execution, that result is returned with along with a content-type `application/octet-stream`.
  2. If no result is available when the execution completes, the logs associated with the execution are returned with content-type `text/html` (charset utf8 is assumed).


----------
Executions
----------

Once you send a message to an actor, that actor will create an execution for the actor
with the inputted data. This execution will be queued waiting for a worker to spool up
or waiting for a worker to be freed. When the execution is initially created it is
given an execution_id so that you can access information about it using the execution_id endpoint.

Access execution data
^^^^^^^^^^^^^^^^^^^^^

cURL
----

You can access the ``execution_id`` endpoint using cURL with the following:

.. code-block:: bash
    
    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>

Python
------

You can access the ``execution_id`` endpoint using ``AgavePy`` and Python with the following:

.. code-block:: python

    ag.actors.getExecution(actorId='<actor_id>',
                           executionId='<execution_id>')    

Results
-------

Access the ``execution_id`` endpoint will result in something similar to the following:

.. code-block:: bash
    
    {'message': 'Actor execution retrieved successfully.',
     'result': {'_links': {'logs': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/executions/00wLaDX53WBAr/logs',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/apitest',
       'self': 'https://api.tacc.utexas.edu/actors/v2/R0y3eYbWmgEwo/executions/00wLaDX53WBAr'},
      'actorId': 'R0y3eYbWmgEwo',
      'apiServer': 'https://api.tacc.utexas.edu',
      'cpu': 7638363913,
      'executor': 'apitest',
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
     
List executions
^^^^^^^^^^^^^^^

Abaco allows users to retrieve all executions tied to an actor with the
``executions`` endpoint.

cURL
----

List executions with cURL by getting the ``executions endpoint``

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions

Python
------

To list executions with ``AgavePy`` the following is done:

.. code-block:: python

    ag.actors.listExecutions(actorId='<actor_id>')

Results
-------

Calling the list of executions should result in something similar to:

.. code-block:: bash

    {'message': 'Actor execution retrieved successfully.',
     'result': {'_links': {'logs': 'https://api.tacc.utexas.edu/actors/v2/R4OR3KzGbRQmW/executions/YqM3RPRoWqz3g/logs',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/apitest',
       'self': 'https://api.tacc.utexas.edu/actors/v2/R4OR3KzGbRQmW/executions/YqM3RPRoWqz3g'},
      'actorId': 'R4OR3KzGbRQmW',
      'apiServer': 'https://api.tacc.utexas.edu',
      'cpu': 0,
      'executor': 'apitest',
      'id': 'YqM3RPRoWqz3g',
      'io': 0,
      'messageReceivedTime': '2019-02-22 01:01:50.546993',
      'runtime': 0,
      'startTime': 'None',
      'status': 'SUBMITTED'},
     'status': 'success',
     'version': '0.11.0'}

Reading message in execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

One of the most important parts of using data in an execution is reading said
data. Retrieving sent data depends on the data type sent.

Python - Reading in raw string data or JSON
-------------------------------------------

To retrieve JSON or raw data from inside of an execution using Python and 
``AgavePy``, you would get the message context from within the actor and then
get it's ``raw_message`` field. 

.. code-block:: python
	
	from agavepy.actors import get_context

	context = get_context()
	message = context['raw_message']

Python - Reading in binary
--------------------------

Binary data is transmitted to an execution through a FIFO pipe located at
/_abaco_binary_data. Reading from a pipe is similar to reading from a regular
file, however ``AgavePy`` comes with an easy to use ``get_binary_message()``
function to retrieve the binary data.

**Note:** Each Abaco execution processes one message, binary or not. This means
that reading from the FIFO pipe will result with exactly the entire sent
message.

.. code-block:: python

	from agavepy.actors import get_binary_message

	bin_message = get_binary_message()

Cron Schedule
^^^^^^^^^^^^^^^

.. Note::
  The Abaco Cron Schedule feature was implemented in version 1.7.0.

Abaco's cron schedule is a tool to automatically execute your actor based on a schedule.
Each actor has two user-defined parameters associated with the cron execution: 
``cronSchedule`` and ``cronOn``. The scheduler has another variable, ``cronNextEx``,
which holds the next execution time of the actor. This is an internal variable 
and cannot be edited by users. 
To create a schedule, set the ``cronSchedule`` parameter when registering a new actor or updating an
existing actor.
The value of ``cronSchedule`` should be a string in the following format:

.. code-block:: bash

  yyyy-mm-dd hh + <number> <unit of time>

where the first part is the datetime when the first execution will happen, and the 
second part is the time increment for each subsequent execution. Note that the spaces,
plus characters (``+``) and dash characters (``-``) in the template above are meaningful and
are a required part of
the format. Abaco's cron schedule also has an alias
called ``now``, which lets you execute the actor at the current UTC time. For example,
if an actor was registered with this schedule

.. code-block:: bash

  "cronSchedule": "now + 1 hour"

the actor would execute at the current time, and then again at the top of the hour every hour.

.. Note::
  The highest granularity is the hour, and the units of time that can be used are hour, day, week, and month.

To create an actor with a schedule, make a request like the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"image": "abacosamples/test", "cronSchedule": "2020-09-28 16 + 1 hour"}' \
    https://api.tacc.utexas.edu/actors/v2

To update the schedule, make a request like the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -X PUT \
    -H "Content-Type: application/json" \
    -d '{"image": "abacosamples/test", "cronSchedule": "2020-12-11 16 + 3 days"}' \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>

This last request above will update the cron schedule for the actor with id ``<actor_id>`` as
follows: the actor will be scheduled to
automatically execute on December 11th, 2020 at 4 pm, UTC timezone. 
That actor will be executed again 3 days later on the 14th, 
at 4pm, and then 3 days after that, again at 4pm. This execution will 
recur every 3 days until the user changes the cron schedule,
turns off the cron schedule, or deletes the actor.

.. Note::
  The cron schedule runs on the UTC timezone. 

.. Note::
  When making requests to set the cronSchedule, be sure to pass "application/json" content to avoid
  issues requiring escaping characters inside the schedule value.


To turn off the schedule, use the `cronOn` switch like so:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    -X PUT \
    -H "Content-Type: application/json" \
    -d '{"image": "abacosamples/test", "cronOn": "False"}' \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>


By turning off the schedule, the actor will no longer execute itself at each increment. 
You can turn it on again at any time, and the actor will 
resume incrementing as before. For example, if an actor is set to 
execute every hour, and then the cron switch is turned off, the actor will
stop executing itself. After a week, the switch can be turned back on, and the
actor will resume executing on the hour. 

Cron Schedule - Error Messages
------------------------------
If users supply a value for `cronSchedule` in an incorrect format, they will receive an error
letting them know to check the format. The API also checks that the schedule
sent in has not already past. For example, if you pass in the year 1955, you 
will get an error message saying the cron schedule has already passed. The error 
message will also tell you the current UTC time for reference. 

Cron Message and Execution
--------------------------
When it is time to execute an actor configured with a ``cronSchedule``, Abaco's internal cron agent
simply queues a message on the actor's internal message queue, just as if a client had sent a message
to the actor using the ``/messages`` API. If the actor already has (unprocessed) messages in its queue,
these messages will be processed first before the cron message. This means that there could be some delay
between the time Abaco internally queues the message and the actor starts executing it.

Currently, the cron message sent to the actor is the static string

.. code-block:: bash

  This is your cron execution

Accordingly, the ``_abaco_Content_Type`` context variable is set to ``str``. The rest of the context
variables are set normally, as described in :ref:`context`.

----
 
----
Logs
----

At any point of an execution you are also able to access the execution logs
using the ``logs`` endpoint. This returns information
about the log along with the log itself. If the execution is still in the
submitted phase, then the log will be an empty string, but once the execution
is in the completed phase the log would contain all outputted command line data.

Retrieving an executions logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

cURL
----

To call the ``log`` endpoint using cURL, do the following:

.. code-block:: bash

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/<actor_id>/executions/<execution_id>/logs

Python
------

To call the ``log`` endpoint using ``AgavePy`` and Python, do the following:

.. code-block:: python

    ag.actors.getExecutionLogs(actorId='<actor_id>',
                               executionId='<executionId>')

Results
-------

This would result in data similar to the following:

.. code-block:: bash

    {'message': 'Logs retrieved successfully.',
     'result': {'_links': {'execution': 'https://api.tacc.utexas.edu/actors/v2/qgKRpNKxg0DME/executions/qgmq08wKARlg3',
       'owner': 'https://api.tacc.utexas.edu/profiles/v2/apitest',
       'self': 'https://api.tacc.utexas.edu/actors/v2/qgKRpNKxg0DME/executions/qgmq08wKARlg3/logs'},
      'logs': '<command line output here>'},
     'status': 'success',
     'version': '0.11.0'}
