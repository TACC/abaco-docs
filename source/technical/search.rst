.. _search:

======
Search
======
With the release of Abaco 1.6.0, a search capability has been introduced using
the Mongo aggregation system, full-text searching, and indexing. Searching can be
done on actor, worker, execution, and log collections. This feature allows users
to search based on any attribute associated with resources that they have permission
to view. For example, using search, a user could retrieve all viewable executions with status
"ERROR" in one API call. The search currently makes use of logical operators and
datetime to allow for easy searching of any object based on any specific field.

.. attention::
    Search in Abaco was implemented in version 1.6.0.

Search has been implemented via query parameters on a new ``/search/<collection>`` endpoint. To use it, specify the collection to be
searched (``actors``, ``workers``, ``executions``, or ``logs``) in the URL. With no query arguments Abaco will return
all entries in the collection that you have permission to view. To specify query arguments, add a ``?`` to the end of
the url and specify the parameters in form ``<attribute_name>.<operator>=<param_value>`` separated by ``&``. If not
specified, the search defaults the operator to the equality operator (i.e., the ``eq`` operator). The general form for
requests to the search endpoint looks like:

.. code-block:: bash

  GET /actors/v2/search/<collection>?<attr_1>.<op_1>=<value_1>&<attr_2>.<op_2>=<value_2>&...

where ``<attr_1>``, ``<attr_2>``, etc. are valid attributes on an instance of ``<collection>`` (for example, ``image``
for the ``actors`` collection), ``<op_1>``, ``<op_2>``,
etc. are valid Abaco search operators (see table below), and ``<value_1>``, ``<value_2>``, etc. are values for the
attribute type. The response from a
search consists of the list of objects of type ``<collection>`` that meet the search criteria and that the caller has
view access to.

The same query parameters can also be used on the following existing endpoints:

+-------------------------------------+-------------------------------------------------+
| Endpoint                            | Description                                     |
+=====================================+=================================================+
| /actors                             | Search all actors; equivalent to /search/actors |
+-------------------------------------+-------------------------------------------------+
| /actors/{aid}/executions            | Search all executions for a fixed actor.        |
+-------------------------------------+-------------------------------------------------+
| /actors/{aid}/executions/{eid}/logs | Search logs for a specific execution.           |
+-------------------------------------+-------------------------------------------------+
| /actors/{aid}/workers               | Search all workers for a fixed actor.           |
+-------------------------------------+-------------------------------------------------+

When applied to one of the existing endpoints above, the query parameters can be thought of as filters,
refining the set of objects that would have been returned by the listing.

A table of valid operator parameters, their function, and examples are below.

+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| Operator    | Function                                                                                                                           | Examples                                |
+=============+====================================================================================================================================+=========================================+
| eq          | Checks if given value is equal to db value matching given key. This is the default operator.                                       | ?id.eq=AKY5o4Z847lB3                    |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| neq         | Checks if given value is not equal to db value matching given key.                                                                 | ?id.neq=AKY5o4Z847lB3                   |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| gt          | Checks if given value is greater than db value matching given key.                                                                 | ?start_time.gt=2020-04-29+06:00         |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| gte         | Checks if given value is greater than or equal to db value matching given key.                                                     | ?runtime.gte=423                        |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| lt          | Checks if given value is less than db value matching given key.                                                                    | ?message_received_time.lt=2020          |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| lte         | Checks if given value is less than or equal to db value matching given key.                                                        | ?final_state.FinishedAt.lte=2020-04-29  |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| in          | Checks if db value matching given key match any values in the given list of values.                                                | ?status.in=["BUSY","REQUESTED","READY"] |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| nin         | Checks if db value matching given key does not match any values in the given list of values.                                       | ?status.nin=["COMPLETED", "READY"]      |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| like        | Checks if given value in (through regex) db value matching given key.                                                              | ?image.like=abaco_docker_username       |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| nlike       | Checks if given value not in (through regex) db value matching given key.                                                          | ?image.nlike=abaco_test                 |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| between     | Checks if db value matching given key is greater than or equal to first given value, and less than or equal to second given value. | ?start_time.between=                    |
|             |                                                                                                                                    | 2020-04-29T20:15:52:246Z,               |
|             |                                                                                                                                    | 2021-06-24-05:00                        |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| limit       | Sets a limit on total amount of results returned. Defaults to 10 results.                                                          | ?limit=20                               |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| skip        | Skips a specified amount of results when returning.                                                                                | ?skip=4                                 |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+

Additionally, the Abaco search supports the following special parameters

+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| Parameter   | Function                                                                                                                           | Examples                                |
+=============+====================================================================================================================================+=========================================+
| search      | Completes a fuzzy full-text search based on inputs. Returns results by best accuracy/score.                                        | ?search=stringToSearchFor               |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| exactsearch | Completes a full-text search and looks for exact matches with inputs.                                                              | ?exactsearch=stringToMatchExactly       |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| limit       | Limit the number of results to return.                                                                                             | ?limit=5                                |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| skip        | The number of results to skip.                                                                                                     | ?skip=10                                |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+

You may use as many search parameters as you want in one query sans ``limit`` and ``skip``, where each may only be used once.

--------

--------
Metadata
--------
Abaco formats the responses to searches slightly differently from a typical response in the fact that the response
to a search returns two objects within the ``result`` object: a  ``search`` object, containing the actual results, and
a ``_metadata`` object. The ``_metadata`` object returns pertinent information about the amount of records returned,
the amount of records the return is limited to, the amount of records skipped (specified in query),
and the total amount of records that match the query searched for. This is useful to implement paging
or to only receive a set amount of records.

.. important::
    A search ``result`` object contains a ``_metadata`` object and a ``search`` object, the latter is a JSON list
    containing the actual search results.

Example of a Search Response
----------------------------

.. code-block:: bash

    {'message': 'Executions search completed successfully.',
     'result': {'_metadata': {'countReturned': 1,
                             'recordLimit': 10,
                             'recordsSkipped': 0,
                             'totalCount': 1},
                 'search': [{'_links': {'logs': 'https://dev.tenants.aloedev.tacc.cloud/actors/v2/joBjeDkWyBwLx/logs',
                                     'owner': 'https://dev.tenants.aloedev.tacc.cloud/profiles/v2/testuser',
                                     'self': 'https://dev.tenants.aloedev.tacc.cloud/actors/v2/joBjeDkWyBwLx/executions/1JKkQwX75vE56'},
                             'actorId': 'joBjeDkWyBwLx',
                             'cpu': 444097006,
                             'executor': 'testuser',
                             'exitCode': 0,
                             'finalState': {'Dead': False,
                                         'Error': '',
                                         'ExitCode': 0,
                                         'FinishedAt': '2020-04-29T21:47:21.385Z',
                                         'OOMKilled': False,
                                         'Paused': False,
                                         'Pid': 0,
                                         'Restarting': False,
                                         'Running': False,
                                         'StartedAt': '2020-04-29T21:47:19.382Z',
                                         'Status': 'exited'},
                             'id': '1JKkQwX75vE56',
                             'io': 716,
                             'messageReceivedTime': '2020-04-29T21:47:18.7Z00',
                             'runtime': 2,
                             'startTime': '2020-04-29T21:47:18.954Z',
                             'status': 'COMPLETE',
                             'workerId': '7kvAAKYKB6Qk6'}]},
     'status': 'success',
     'version': ':dev'}

--------

------
Inputs
------
All inputs are given to the search function as query parameters and thus are transmitted
as strings. Abaco attempts to convert these inputs to the native type associated with the
attribute. Strings are left untouched. Booleans are expected to be "False" or "false" and
"True" or "true" to be converted. Numbers are all converted to floats. Lists are parsed
with ``json.loads`` and will accept either ``["test"]`` or ``['test']`` with post-processing
on Abaco's end to convert to lists.

The last consumed input type is datetime objects. Abaco accepts a broad range of ISO 8601
like strings. An example of the most detailed string accepted is ``2020-04-29T20:15:52:246252-06:00``.
``2020-04-29T20:15:52:246Z``, ``2020-04-29T20:15:52-06:00``, ``2020-04-29T20:15-06:00``,
``2020-04-29T20-06:00``, ``2020-04-29-06:00``, ``2020-04Z``, and ``2020`` are also acceptable.

.. attention::
    Abaco stores all times in UTC, so addition of your timezone or conversion to UTC is
    important. If no timezone information is given (``-06:00`` or ``Z`` (to signal UTC))
    the datetime is assumed to be in UTC.

.. important::
    For the purposes of comparison, unspecified elements of a datetime string are set to the minimal possible value. For example,
    the string "2020-12-30" is greater than "2020-12" which in turn is greater than "2020". As datetime objects, these
    are converted to ``2020-12-30T00:00:00Z``, ``2020-12-01T00:00:00Z`` and ``2020-01-01T00:00:00Z``, respectively. This
    holds true until you reach millisecond accurate time.

Note that use of nonces is limited to searching within the actor or alias the nonce was created for. Abaco does not allow
the use of nonce on the global search enpoint.

.. important::
    Nonces can be used along side search query parameters by setting the ``x-nonce`` parameter as usual; queries should
    still work as expected and do not need any additional modification. Searches using a nonce count as a nonce "use" as
    with any other API call using a nonce.


Creating ISO 8601 formatted strings
-----------------------------------

The following examples may be helpful for working with datetime objects in Python.

Python - String with Timezone
------

The following gets the current time as an ISO 8601 formatted string with timezone:

.. code-block:: python
    
    import datetime
    import pytz

    austin_time_zone = pytz.timezone("America/Chicago")
    isoString = datetime.datetime.now(tz=austin_time_zone).isoformat()
    print(isoString)

This prints ``2020-04-29T16:21:34.602078-05:00``.


Python - UTC String
------

The following gets the current UTC time as an ISO 8601 formatted string:

.. code-block:: python
    
    import datetime

    isoString = datetime.datetime.utcnow().isoformat()
    print(isoString)

This prints ``2020-04-29T21:21:34.602078``. Feel free to add the Z or leave it absent.


--------

---------------
Search Examples
---------------

In this section we provide some example searches using the new search endpoint as well as query parameters applied to
some existing endpoints.

Using the Global Search Endpoint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Use the new search endpoint to search for all actors defined with image "abacosamples/test", created since 4/29/2020
and in either "READY" or "BUSY" status:

cURL
----

.. code-block:: cURL

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/search/actors?image=abacosamples/test&create_time.gt=2020-04-29&status.in=["READY", "BUSY"]

Result
------

.. code-block:: bash

    {'message': 'Search completed successfully.',
    'result': {'_metadata': {'countReturned': 1,
                            'recordLimit': 10,
                            'recordsSkipped': 0,
                            'totalCount': 1},
                'search': [{'_links': {'executions': 'https://dev.tenants.aloedev.tacc.cloud/actors/v2/joBjeDkWyBwLx/executions',
                                    'owner': 'https://dev.tenants.aloedev.tacc.cloud/profiles/v2/testuser',
                                    'self': 'https://dev.tenants.aloedev.tacc.cloud/actors/v2/joBjeDkWyBwLx'},
                            'createTime': '2020-04-29T21:46:53.393Z',
                            'defaultEnvironment': {'default_env_key1': 'default_env_value1',
                                                'default_env_key2': 'default_env_value2'},
                            'description': '',
                            'gid': None,
                            'hints': [],
                            'id': 'joBjeDkWyBwLx',
                            'image': 'abacosamples/test',
                            'lastUpdateTime': '2020-04-29T21:46:53.393Z',
                            'link': '',
                            'maxCpus': None,
                            'maxWorkers': None,
                            'memLimit': None,
                            'mounts': [{'container_path': '/_abaco_data1',
                                        'host_path': '/data1',
                                        'mode': 'ro'}],
                            'name': 'abaco_test_suite_default_env',
                            'owner': 'testuser',
                            'privileged': False,
                            'queue': 'default',
                            'state': {},
                            'stateless': True,
                            'status': 'READY',
                            'statusMessage': ' ',
                            'tasdir': None,
                            'token': 'false',
                            'type': 'none',
                            'uid': None,
                            'useContainerUid': False,
                            'webhook': ''}]},
    'status': 'success',
    'version': ':dev'}


2. Use the global search endpoint to look for all executions that are not in COMPLETE status across all actors the
user has access to.

cURL
----

.. code-block:: cURL

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/search/executions?status.neq="COMPLETE"

Result
------

.. code-block:: bash

    {
      "message": "Search completed successfully.",
      "result": {
        "_metadata": {
          "countReturned": 4,
          "recordLimit": 100,
          "recordsSkipped": 0,
          "totalCount": 4
        },
        "search": [
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/7zxX7DlZ0eeZY/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/7zxX7DlZ0eeZY/executions/8mzXG1vxDaZZ1"
            },
            "actorId": "7zxX7DlZ0eeZY",
            "cpu": 0,
            "executor": "testuser",
            "exitCode": null,
            "finalState": null,
            "id": "8mzXG1vxDaZZ1",
            "io": 0,
            "messageReceivedTime": "2020-05-05T19:50:23.813Z",
            "runtime": 0,
            "startTime": null,
            "status": "SUBMITTED",
            "workerId": null
          },
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/7zxX7DlZ0eeZY/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/7zxX7DlZ0eeZY/executions/7mYv7rxYNNYw1"
            },
            "actorId": "7zxX7DlZ0eeZY",
            "cpu": 0,
            "executor": "testuser",
            "exitCode": null,
            "finalState": null,
            "id": "7mYv7rxYNNYw1",
            "io": 0,
            "messageReceivedTime": "2020-05-05T19:50:23.296Z",
            "runtime": 0,
            "startTime": null,
            "status": "RUNNING",
            "workerId": "1zLYaONZxWQAX"
          },
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/executions/jg0oLKJg8VvgM"
            },
            "actorId": "jm6kjmDmW885N",
            "cpu": 0,
            "executor": "testuser",
            "exitCode": null,
            "finalState": null,
            "id": "jg0oLKJg8VvgM",
            "io": 0,
            "messageReceivedTime": "2020-05-05T19:50:20.113Z",
            "runtime": 0,
            "startTime": null,
            "status": "RUNNING",
            "workerId": "7XZN4aqvzoJ33"
          },
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/executions/jgM7JBmqDDjM5"
            },
            "actorId": "jm6kjmDmW885N",
            "cpu": 0,
            "executor": "testuser",
            "exitCode": null,
            "finalState": null,
            "id": "jgM7JBmqDDjM5",
            "io": 0,
            "messageReceivedTime": "2020-05-05T19:50:20.925Z",
            "runtime": 0,
            "startTime": null,
            "status": "SUBMITTED",
            "workerId": null
          }
        ]
      },
      "status": "success",
      "version": "1.6.0"
    }


3. Find all executions for actor ``jm6kjmDmW885N`` that completed after "2020-05-05T19:50:23.748".

cURL
----

.. code-block:: cURL

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/executions?status=COMPLETE&start_time.gt=2020-05-05T19:50:23.748

Result
------

.. code-block:: bash

    {
      "message": "Executions search completed successfully.",
      "result": {
        "_metadata": {
          "countReturned": 2,
          "recordLimit": 100,
          "recordsSkipped": 0,
          "totalCount": 2
        },
        "search": [
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/executions/jg0oLKJg8VvgM"
            },
            "actorId": "jm6kjmDmW885N",
            "cpu": 159212854,
            "executor": "testuser",
            "exitCode": 0,
            "finalState": {
              "Dead": false,
              "Error": "",
              "ExitCode": 0,
              "FinishedAt": "2020-05-05T19:50:29.038Z",
              "OOMKilled": false,
              "Paused": false,
              "Pid": 0,
              "Restarting": false,
              "Running": false,
              "StartedAt": "2020-05-05T19:50:27.003Z",
              "Status": "exited"
            },
            "id": "jg0oLKJg8VvgM",
            "io": 266,
            "messageReceivedTime": "2020-05-05T19:50:20.113Z",
            "runtime": 2,
            "startTime": "2020-05-05T19:50:26.697Z",
            "status": "COMPLETE",
            "workerId": "7XZN4aqvzoJ33"
          },
          {
            "_links": {
              "logs": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/logs",
              "owner": "https://api.tacc.utexas.edu/profiles/v2/testuser",
              "self": "https://api.tacc.utexas.edu/actors/v2/jm6kjmDmW885N/executions/jgM7JBmqDDjM5"
            },
            "actorId": "jm6kjmDmW885N",
            "cpu": 172730092,
            "executor": "testuser",
            "exitCode": 0,
            "finalState": {
              "Dead": false,
              "Error": "",
              "ExitCode": 0,
              "FinishedAt": "2020-05-05T19:50:32.123Z",
              "OOMKilled": false,
              "Paused": false,
              "Pid": 0,
              "Restarting": false,
              "Running": false,
              "StartedAt": "2020-05-05T19:50:30.085Z",
              "Status": "exited"
            },
            "id": "jgM7JBmqDDjM5",
            "io": 396,
            "messageReceivedTime": "2020-05-05T19:50:20.925Z",
            "runtime": 2,
            "startTime": "2020-05-05T19:50:29.723Z",
            "status": "COMPLETE",
            "workerId": "7XZN4aqvzoJ33"
          }
        ]
      },
      "status": "success",
      "version": "1.6.0"
    }

