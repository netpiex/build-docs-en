.. raw:: html

    <div align="right"><a href="https://docs.netpie.io/th/device-api.html">TH</a> | <b>EN</b></div>

RESTful API
============

|

Provides a way to connect the device to the platform via a RESTful API that uses HTTP protocol. The API functionality can be tested at |swagger_part|. There are two groups of API’s that are currently available.

|

Device API
--------------------

|

It is an API related to devices. Its domain name is |rest_url|. The details are as follows:

|

**1. Publish messages to various topics, can be used in two ways:**

- Type 1: Topic is specified in the URL path.

:EndPoint: |rest_url|/message/{any}/{topic}

:Method: PUT

:Request Header: Authorization : *Device ClientID:Token*

:Request Body: Content-type : *text/plain*
	
	Message to Publish to Topic

:Return: *Response Object*

	- ``status`` => Response Code (HTTP Response Code)

	- ``message`` => Reply message

:Example: 
	
	PUT /device/message/mythings/bedroom/light HTTP/1.1

	Host: |rest_url2|

	Authorization: Device 777d5c96-4c83-4aa6-a273-5ee7c5f453b1:abcduKh8r2tP1zVc1W1nG8YWZeu21234

	ON

|

- Type 2: Specify Topic through parameter (Query string)

:EndPoint: |rest_url|/message

:Method: PUT

:Request Header: Authorization : *Device ClientID:Token*

:Parameter: ``topic`` : string is the Topic you want to publish messages to {{any}/{topic}}

:Return: *Response Object*

	- ``status`` => Response Code (HTTP Response Code)

	- ``message`` => Reply message

:Example: 
	
	PUT /device/message?topic=mything/bedroom/light HTTP/1.1

	Host: |rest_url2|

	Authorization: Device 777d5c96-4c83-4aa6-a273-5ee7c5f453b1:abcduKh8r2tP1zVc1W1nG8YWZeu21234

	OFF

|

.. caution:: 

	Setting Topic in the REST API is different from that of MQTT protocol. If the message is sent via REST, you need not put “@msg” at the front in the Topic, as the system will autocomplete for you. But, if you are sending a message via MQTT, you still need to add the prefix “@msg” in the Topic.

|

**2. Reading the Shadow Data of the Device (Must be a Device belonging to the same Group)**

:EndPoint: |rest_url|/shadow/data

:Method: GET

:Request Header: Authorization : *Device ClientID:Token*

:Parameter: ``alias`` : Name of the device of the shadow you want to read. (if it reads your shadow, don't send this parameter)

:Return: *Response Object*

	- ``status`` => Response Code (HTTP Response Code)

	- ``data`` => Device Shadow Data (JSON)

:Example: 

	GET /device/shadow/data?alias=sensor HTTP/1.1

	Host: |rest_url2|

	Authorization: Device 777d5c96-4c83-4aa6-a273-5ee7c5f453b1:abcduKh8r2tP1zVc1W1nG8YWZeu21234

**3. Writing Data to Shadow Data as Merge**

:EndPoint: |rest_url|/shadow/data

:Method: PUT

:Request Header: Authorization : *Device ClientID:Token*

:Parameter: ``alias`` : Device name of the shadow that you want to write. (If writing to your own shadow, don't send this parameter)

:Request Body: 
	
	The data that you want to write to the Shadow Data will be in json format as follows: 

	``{data: {field name 1: value1, field name 2: value2, ..., field name n: value n}}``

:Return: *Response Object*

	- ``status`` => Response Code (HTTP Response Code)

	- ``data`` => Device Shadow Data (JSON) update information

:Example: 
	
	PUT /device/shadow/data?alias=test HTTP/1.1
	
	Host: |rest_url2|

	Authorization: Device 777d5c96-4c83-4aa6-a273-5ee7c5f453b1:abcduKh8r2tP1zVc1W1nG8YWZeu21234

	{data:{temperature:33.7, config: {item1: a, item2: b}, note: test case}}

**4. Writing Data to Shadow Data as to Overwrite (Overwrite)**

:EndPoint: |rest_url|/shadow/data

:Method: POST

:Request Header: Authorization : *Device ClientID:Token*

:Parameter: ``alias`` : Device name of the shadow that you want to overwrite. (If writing to your own shadow, don't send this parameter)

:Request Body: 
	
	The data that you want to write to Shadow Data will be in JSON format as follows: 

	``{data: {field name 1: value1, field name 2: value2, ..., field name n: value n}}``

:Return: *Response Object*

	- ``status`` => Response Code (HTTP Response Code)

	- ``data`` => Device Shadow Data (JSON) update information

:Example: 

	POST /device/shadow/data?alias=test HTTP/1.1
	
	Host: |rest_url2|

	Authorization: Device 777d5c96-4c83-4aa6-a273-5ee7c5f453b1:abcduKh8r2tP1zVc1W1nG8YWZeu21234

	{data:{temperature:33.7, config: {item1: a, item2: b}, note: test case}}

|

Data Store API
--------------------

|

It is an API used to retrieve data stored in the Time-Series database. The API’s domain name is |feed_url|. The database used is KairosDB. The parameters will be sent in the same format as KairosDB

:EndPoint: |feed_url|/api/v1/datapoints/query

:Method: POST

:Request Header: Authorization : *Bearer UserToken* หรือ Authorization : *Device ClientID:DeviceToken* 

	Content-Type : *application/json*

:Request Body: The terms used in the query are JSON format and can be classified into two types:

	*1. Query Properties* include:

	- ``start_absolute`` => Start time in milliseconds.

	- ``start_relative`` => Start time relative to current time. The units can be milliseconds, seconds, minutes, hours, days, weeks, months, and years. For example, if the start time is 5 minutes, the data points returned will be in the last 5 minutes.

	- ``end_absolute`` => End time in milliseconds and must be greater than ``start_absolute``.

	- ``end_relative`` => Specify end time relative to the current time. Units can be milliseconds, seconds, minutes, hours, days, weeks, months, and years. For example, if the start time is 30 minutes and the end time is 10 minutes, then the data points returned will be in between the last 30 minutes, to last 10 minutes. If the end time is not specified, then it will take the current time and date.

	- ``time_zone`` => Time zone for the data query period. If not specified, UTC will be used. (for time zone, NETPIE is set to GMT)

	** Note ** : You need to mention values for ``start_absolute`` and ``start_relative``. On other hand, values for ``end_absolute`` and ``end_relative`` can be optional.

	|

	*2. Metric Properties* include:

	- ``name`` => Name of the Metric to query data. Specify Device ID (ClientID) from |platform_name|, which is a required action.

	- ``aggregators`` => Aggregate or process data in various forms before returning the data point. The relevant parameters are as follows:

		- name => The type of data processing format is “avg” (Average), “dev” (Standard Deviation), “count”, “first”, “gasps”, “histogram”, “lasts”, “least_squares”, “max”, “min”, “percentile”, “sum”, “diff” (Difference), “div” (Divide), “rate”, “sampler”, “scale”, “trim”, “save_as”, “filter”, “js_function” (JS Aggregator), “js_range” (JS A ggregator). See `Kairosdb <https://kairosdb.github.io/docs/build/html/restapi/Aggregators.html>`_ for more details. 

	- ``tags`` => Used for filtering required data on |platform_name|. Format is, tags: {attr: {field 1, field 2, …, field n}}

	- ``group_by`` => Group data query points. Can be organized by Tags, Time period, Data point value, or by Data bucket on |platform_name|. 

	- ``exclude_tags`` => Will show Tag in the returned data. (true: show Tag as default, false: do not display Tag)

	- ``limit`` => This limits the number of data points to query.

	- ``order`` => Sorts the data points. (asc: ascending, des: descending). It will sort the actual data points first.

:Return: *Response Object*

	Extract Successfully (status : 200)

		Query data is in JSON format

	Data retrieval failed (status : 400 or 500)

		- 400 Bad Request => Bad request, such as incomplete or incorrect parameter sent.

		- 500 Internal Server Error => If there is an error in retrieving data from the server side.


:Example 1 Authorization with User Token: 

	POST /api/v1/datapoints/query HTTP/1.1

	Host: |feed_url2|

	Authorization: Bearer AyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.AyJjdHgiOnsib3duZXIiOiJVOTc0ODE0NzczMjA0In0sInNjb3BlIjpbXSwiaWF0Ijox

	NTcxMzc1ODk4LCJuYmYiOjE1NzEzNzU4OTgsImV4cCI6MTU3MTQ2MjI5OCwiZXhwaXJlSW4iOjg2NDAwLCJqdGkiOiIzRk50VkVmVCIsImlzcyI6I

	mNlcjp1c2VydG9rZW4ifQ.AtbhSRgGXCjiQk4wENMD4KQ3ufDof7HnzHY5Rcli0y0LpTJEDLklM-AmsAVzBnPBnJh9L3LvSGODc9xrYWotcA

	Content-Type: application/json

	{ "start_relative": { "value":1, "unit":"days" }, "metrics":[{ "name":"Aaa5d93b-Ae16-455f-A854-335AAAA16256", "tags":{"attr":["temp", "humit"]}, "limit":50, "group_by":[{ "name":"tag", "tags":["attr"] }], "aggregators":[{ "name":"avg", "sampling":{ "value":"1", "unit":"minutes" } }] }] }

:Example 2 Authorization with ClientID and Device Token: 

	POST /api/v1/datapoints/query HTTP/1.1

	Host: |feed_url2|

	Authorization: Device Aaa5d93b-Ae16-455f-A854-335AAAA16256:TuZfsgosxxxxx3br4Qt1Do9jvzLM5hZQ

	Content-Type: application/json

	{ "start_relative": { "value":1, "unit":"days" }, "metrics":[{ "name":"Aaa5d93b-Ae16-455f-A854-335AAAA16256", "tags":{"attr":["temp", "humit"]}, "limit":50, "group_by":[{ "name":"tag", "tags":["attr"] }], "aggregators":[{ "name":"avg", "sampling":{ "value":"1", "unit":"minutes" } }] }] }
