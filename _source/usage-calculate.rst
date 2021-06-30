.. raw:: html

    <div align="right"><a href="https://docs.netpie.io/th/usage-calculate.html">TH</a> | <b>EN</b></div>

Service Usage Calculation
==========================

|

The calculation of amount of usage of NETPIE services is classified into following  categories:

|

API Call
--------------------

All REST API traffic on |platform_name| platform is counted in this operation. The actions that are part of this usage are:

|

API Request 
	: The number of requests made through REST API. The size of the payload sent in each block is up to 4 kilobytes.

|

API Response 
	: The number of requests made through REST API. The size of each block in the response does not exceed 4 kilobytes.

|

*Example*
````````````
	Take the case of reading Time-Series database data through REST API. The request sent is 71 bytes and response is of 10 kilobytes. So, the count is as follows:

	|

	Request sends a payload of 71 bytes = 71/1024*4 = 1 Operation (Request). Fractions that are not divisible by 4KB, must be rounded to 1 block.

	|

	Response payload is 10KB = 10/4=3 Operations (Response). Fractions that are not divisible by 4KB, must be rounded to 1 block.

	|

	API Call quota that was used = 1+3 = 4 Operations.

|

Real-time Messages
----------------------------

The service traffic related to MQTT is considered as a message. The actions that will be counted as this type of usage are:

|

MQTT Publish
	: Number of messages published. Based on the size of the message, each block does not exceed 4 kilobytes.

|

MQTT Subscribe
	: The number of times each subscription requested is counted as one message.

|

MQTT Deliver
	: Number of messages sent to the device. Based on the size of the message, each block does not exceed 4 kilobytes.

|

MQTT Connect
	: Number of requests sent to connect to the NETPIE platform. Each request is counted as one message.

|

*Example*
````````````
	There are 5 devices connected to the NETPIE platform. Device 1 publishes 6 kilobytes of messages to the Topic myDevice. Device 2, Device 3, Device 4, and Device 5 subscribes to the Topic myDevice. So the count is as follows:
	
	|

	5 devices connected to the |platform_name| = 5 * 1 = 5 messages (MQTT CONNECT).
	
	|

	Device 1 publishes messages of size 6KB = 6/4 = 2 messages (MQTT PUBLISH). Fractions that are not divisible by 4 should be rounded to 1 block.
	
	|

	Device 2-5 (4 devices) request to subscribe Topic = 4 * 1 = 4 messages (MQTT SUBSCRIBE).
	
	|

	Device 2-5 (4 devices) receive message = 4 * 2 = 8 messages (MQTT DELIVER) from subscription
	
	|

	Total message quota = 5 + 2 + 4 + 8 = 19 messages.

|

Shadow Read/Write
--------------------

The shadow traffic is considered as a message. The actions that will be counted as this type of usage are:

|

Shadow Read
	: The count is based on the size of the message in shadow being read. Each block does not exceed 1 kilobyte.

|

Shadow Write
	: The count is based on the number of requests sent to the shadow to write. Each block does not exceed 1 kilobyte.

|

Shadow Expression
	: The number of times the shadow is run. The expression is used for data transformation

|

*Example*
````````````
	Device which is in online will read all of its shadow (shadow size is 2 kilobytes) by default. Next, the device sends its current temperature to update shadow (data size is 20 bytes). The value sent is Fahrenheit. Next, expressions are used to convert the units to Celsius. The shadow operation quota is defined as:

	|

	2 Operations (Shadow Read) + 1 Operation (Shadow Write) + 1 Operation (Shadow Expression) = 4 Operations.

Time Series Data Store
-----------------------

This service deals with the amount of data and how long to keep that data. It is counted as Point-Day, Point-Month, or Point-Year, which means the data sent to collect one data-point (data size not more than 1 kilobyte) with a storage time (TTL) of 1 day, 1 month, or 1 year. The number of data-points is inversely proportional to the holding period. (If the number of data-points stored are high, then the holding period is small)

|

*Example*
````````````
	In a period of 30 days, a device collects the temperature and humidity data every 1 hour and stores it in the database for the past 7 days. The data store quota can be calculated as follows: 

	|

	2 (point-data) * (24 (hours/day) * 30 (days)) *  7 (days) = 10080 Point-Day.

	|

	Or

	|

	2 (point-data) * (24 (hours/day) * 30 (days)) * (7 (days) / 30 (days/month)) = 336 Point-Month

	|	

	Or

	|

	2 (point-data) * (24 (hours/day) * 30 (days)) * (7 (days) / 365 (days/year)) = 27.62 Point-Year

|

Trigger & Action
--------------------

|

Trigger related service is counted as operation. The actions that will be counted as this type of usage are as follows:

|

Device Trigger
	: The trigger is caused by device status on the platform. Like, device status changed from connected (Online) to disconnected (Offline) and vice-versa. Set event trigger to DEVICE.STATUSCHANGED. See :ref:`trigger-and-action` for more details. If the trigger is set, and everytime there is a status change, it will be counted as 1 Operation/1 Trigger event set.

|

Shadow Trigger
	: Trigger caused due to change in shadow. Set event trigger to SHADOW.UPDATED. See :ref:`trigger-and-action` for more details. It counts 1 Operation/1 Trigger condition.

|

*Example*
````````````
	In the example below, one device connects to the platform and sends temperature reading 3 times with values as 1, 0, and -1 respectively. The default value in shadow is 0. After sending the reading for 3 times, the device gets disconnected from the platform. The Trigger and Action usage is calculated as:

	|

	The device which is in Online, has two actions to do, “LINENOTIFY” and “myApp” = 2 Operations.

	|

	Send temperature reading for the first time, value is 1. Perform “checkTemp” condition, that returns TRUE = 1 Operation.
	Send temperature reading for the second time, value is 0. Perform “checkTemp” condition, that returns FALSE = 0 Operation.
	Send temperature reading for the third time, value is -1. Perform “checkTemp” condition, that returns FALSE = 0 Operation.

	|

	Device offline (DEVICE.STATECHANGED) does an action, “LINENOTIFY” and “myApp” = 2 Operations.

	|

	Total quota = 2 + 1 + 0 + 0 + 2 = 5 Operations.
	
.. code-block:: json

	{
		"enabled": true,
		"trigger": [{
			"action": "LINENOTIFY",
			"event": "DEVICE.STATECHANGED",
			"msg": "My Device {{$NEW.statustext}}, statuscode: {{$NEW.status}}",
			"option": {
				"url": "https://notify-api.line.me/api/notify",
				"linetoken": "HBfiJA309FWFouCPzK5WhGUvJT1RvN3xb6hGxnIqAAA"
			}
		},
		{
			"action": "myApp",
			"event": "DEVICE.STATECHANGED",
			"msg": "{{$NEW.statustext}}",
			"option": {
				"deviceid": "155941ce-1f4a-4e57-1864-1759af4f872c"
			}
		},
		{
			"action": "checkTemp",
			"event": "SHADOW.UPDATED",
			"condition": "$NEW.bedroom.temp > 0",
			"msg": "My temperature was change from {{$PREV.bedroom.temp}} to {{$NEW.bedroom.temp}}",
			"option": {
				"url": "https://mywebhook/devicetemp"
			}
		}]
	}
