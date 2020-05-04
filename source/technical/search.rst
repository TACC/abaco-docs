.. _search:

======
Search
======
With the release of Abaco 1.6.0, a search capability has been introduced using
the Mongo aggregation system, full-text searching, and indexing. Searching can be
done on actor, worker, execution, and log collections. This feature allows users
to search based on any attribute associated with resources that they have permission
to view. For example, search would allow retrieval of all viewable executions with status
"ERROR" in one API call. The search currently makes use of logical operators and
datetime to allow for easy searching of any object based on any specific field.

.. attention::
    Search in Abaco was implemented in version 1.6.0.

Search has been implemented via query parameters on a new ``/search`` endpoint and has the following general form:

.. code-block:: bash

  GET /actors/v2/search/{collection}?<attr_1>.<op_1>=<value_1>&<attr_2>.<op_2>=<value_2>&...

where ``<attr_1>``, ``<attr_2>``, etc. are valid attributes on an instance of ``<collection>``.

The same query parameters can also be used on the following existing endpoints:

+-------------------------------------+-------------------------------------------+
| Endpoint                            | Description                               |                                                                                         | Examples                                |
+=====================================+===========================================+
| /actors                             | Search all actors, same as /search/actors |
+-------------------------------------+-------------------------------------------+
| /actors/{aid}/executions            | Search all executions for a fixed actor.  |
+-------------------------------------+-------------------------------------------+
| /actors/{aid}/executions/{eid}/logs | Search logs for a specific execution.     |
+-------------------------------------+-------------------------------------------+
| /actors/{aid}/workers               | Search all workers for a fixed actor.     |
+-------------------------------------+-------------------------------------------+

When applied to one of the existing endpoints above, the query parameters can be thought of as filters,
refining the set of objects that would have been returned by the listing.

To use search on the ``/actors/search/{collection}`` endpoint, specify the collection to be searched
(``actors``, ``workers``, ``executions``, or ``logs``) in the URL.
With no query arguments Abaco will return all entries in the collection that you have
permission to view. To specify query arguments, add a ``?`` to the end of the url and specify the parameters
in form ``<param_name>=<param_value>`` separated by ``&``.

A table of search parameters, their function, and examples are below.

+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| Parameter   | Function                                                                                                                           | Examples                                |
+=============+====================================================================================================================================+=========================================+
| search      | Completes a fuzzy full-text search based on inputs. Returns results by best accuracy/score.                                        | ?search=stringToSearchFor               |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| exactsearch | Completes a full-text search and looks for exact matches with inputs.                                                              | ?exactsearch=stringToMatchExactly       |
+-------------+------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------+
| eq          | Checks if given value is equal to db value matching given key.                                                                     | ?id.eq=AKY5o4Z847lB3                    |
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

You may use as many parameters as you want in one query sans ``limit`` and ``skip``, where each may only be used once.

--------

--------
Metadata
--------
Abaco search slightly alters the expected result of a request in the fact that the returned
result from a search now returns two objects, the expected result, ``search``, and ``_metadata``.

This new ``_metadata`` object returns pertinent information about the amount of records returned,
the amount of records the return is limited to, the amount of records skipped (specified in query),
and the total amount of records that match the query searched for. This is useful to implement paging
or to only receive a set amount of records.

.. important::
    A new ``_metadata`` object is now returned alongside the usual result in ``result.``

Example of result with new ``result``
-----------------------------------------

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
All inputs are given to the search function as query parameters and thus are converted
to strings. It is then up to Abaco's side to convert these inputs back to the intended
formats. Strings are left untouched. Booleans are expected to be "False" or "false" and
"True" or "true" to be converted. Numbers are converted all to floats, these are still
comparable to database instances of int, so there should be no issue. Lists are parsed
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
    Comparison with datetime rounds to the minimum time possible. For instance if you want
    to see if 2020-12-30 is greater than 2020, you would receive True as 2020 is rounded
    to `2020-01-01T00:00:00Z`. This holds true until you reach millisecond accurate time.

Creating ISO 8601 formatted strings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

---------
Searching
---------

Like mentioned above, a search may contain as many parameters as a user wants sans for
``limit`` and ``skip``, where each may only be used once. Search on the new
``{base}/actors/search/{collection}`` always takes place and when given no parameters returns
any information the user has access to. To activate on search on the other endpoints, at
least one query parameter must be declared.

.. important::
    ``x-nonce`` queries will still work as expected and do not need any modification.



Performing searches on different endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

{base}/actors/search/actors
^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use ``actors``, ``workers``, ``executions``, or ``logs`` as database inputs
for the endpoints. Each queries the specified database.

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

{base}/actors/joBjeDkWyBwLx/executions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For a search from an endpoint like this the actor_id will already be in the query,
so for this example you would only receive executions with the actor_id of ``joBjeDkWyBwLx``.
``{base}/actors/joBjeDkWyBwLx/workers`` would result in the same behaviour, but for workers.
This usage means that performing a search on ``{base}/actors/joBjeDkWyBwLx/executions/1JKkQwX75vE56/logs``
would always result in one result. Only search on the ``{base}/actors`` thus is the only full
search available that does not use the ``{base}/actors/search/{database}`` endpoint.

.. attention::
    Use the ``{base}/actors/search/{database}`` endpoint for a full search of the specified
    database.

cURL
----

.. code-block:: cURL

    $ curl -H "Authorization: Bearer $TOKEN" \
    https://api.tacc.utexas.edu/actors/v2/search/actors/joBjeDkWyBwLx/executions?status=COMPLETE&start_time.gt=2019

Result
------

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