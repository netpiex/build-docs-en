.. raw:: html

    <div align="right"><a href="https://docs.netpie.io/th/coap-netpie.html">TH</a> | <b>EN</b></div>

CoAP API
==========

|

CoAP (Constrained Application Protocol) is a type of protocol similar to HTTP. But HTTP is TCP, whereas CoAP is UDP. It is developed to reduce the size of the data packet sent. It reduces the use of memory and processing power. Therefore it is suitable for Microcontrollers that have small memory and NB-IoT that use low power. The endpoint is |coap_url|. The details are as follows:

|

Use NodeJS (install from https://nodejs.org/en/download/) for CoAP client installation. The command is:

|

``npm i coap-cli@0.5.1 -g``

|

**1. Publish Messages to Various Topics**

|

.. list-table::
	:widths: 20 80

	* - **EndPoint**
	  - |coap_url|/message/{any}/{topic}
	* - **Method**
	  - PUT
	* - **Parameter**
	  - auth=<ClientID>:<Token>
	* - **Payload**
	  - -p message
	* - **Return**
	  - The response code here is ``undefined``. Because the returned code is not configured.

Example (Command Line) 

.. code-block:: console

	coap put "coap://coap.netpie.io/message/home/bedroom?auth=6c36fdee-5273-4318-xxxx-75dfd2c513db:nzxGsGMYnFdfET6xxxxfb32U9z5kuhvx" -p "Hello from CoAP"

In the example above, it will Publish message ``Hello from CoAP``, to the Topic  ``@msg/home/bedroom``

|

**2. Reading Shadow Data of the Device**

|

.. list-table::
	:widths: 20 80

	* - **EndPoint**
	  - |coap_url|/shadow/data
	* - **Method**
	  - GET
	* - **Parameter**
	  - auth=<ClientID>:<Token>
	* - **Return**
	  - Response Object { ``deviceid`` => ClientID, ``data`` => Device (JSON) Shadow Data, ``rev`` => Shadow Revision, ``modified`` => Last Revision Timestamp}

Example (Command Line) 

.. code-block:: console

	coap get "coap://coap.netpie.io/shadow/data?auth=6c36fdee-5273-4318-xxxx-75dfd2c513db:nzxGsGMYnFdfET6xxxxfb32U9z5kuhvx"

From the example above, it is reading the shadow information of Device ID: 6c36fdee-5273-4318-xxxx-75dfd2c513db . And the return value is:

.. code-block:: json
	
	{
		"deviceid":"6c36fdee-5273-4318-xxxx-75dfd2c513db",
		"data": {
			"humid":76.2, "temp":25
		},
		"rev":3,
		"modified":1605516471534
	}

|

**3. Writing Data to Shadow Data as Merge**

|

.. list-table::
	:widths: 20 80

	* - **EndPoint**
	  - |coap_url|/shadow/data
	* - **Method**
	  - PUT
	* - **Parameter**
	  - auth=<ClientID>:<Token>
	* - **Payload**
	  - -p {data: { Shadow Data (JSON) }}
	* - **Return**
	  - Response Object { ``deviceid`` => ClientID, ``data`` => Device (JSON) Shadow Data, ``modified`` => Last modified Timestamp, ``timestamp`` => Timestamp used to mark the data point in case of Time-series data storage }

Example (Command Line)  

.. code-block:: console

	coap put "coap://coap.netpie.io/shadow/data?auth=6c36fdee-5273-4318-xxxx-75dfd2c513db:nzxGsGMYnFdfET6xxxxfb32U9z5kuhvx" -p "{data: {temp: 30.4} }"

From the example above, It is a merge shadow write of device ID: 6c36fdee-5273-4318-xxxx-75dfd2c513db. And return value is:

.. code-block:: json
	
	{
		"deviceid":"6c36fdee-5273-4318-xxxx-75dfd2c513db",
		"data": {
			"temp":30.4
		},
		"modified":1605518877506,
		"timestamp":1605518877506
	}