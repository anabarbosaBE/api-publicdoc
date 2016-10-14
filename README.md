# How to use BinaryEdge’s API

<p align="center"><img src ="https://dl.dropboxusercontent.com/s/mcu6bftqbbzrf7l/how_to_use_api_2.png?dl=0: 200px;" /></p>

Note: all requests are identified by Job ID and are shown in the stream window.


|   | Input                                                                                                                                                                                                                                                                                                   | Output                                                    |
|---|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| 1 | `curl https://stream.api.binaryedge.io/v1/stream -H "X-Token:InsertYourClientToken" `                                                                                                                                                                                                                                     | (data stream)                                             |
| 2 | `curl https://api.binaryedge.io/v1/tasks -d '{"type":"scan", "description": "InsertYourDescriptionHere", "options":[{"targets":["InsertAnIPAddress/IPNetwork"], "ports":[{"port":InsertPort, "protocol": "tcp or udp", "modules": ["InsertModule"]}]}]}' -v -H "X-Token:InsertYourClientToken"` | {"stream_url":"stream URL","job_id":"Job ID"}             |


### Data Stream

The data stream will send data to you as it's generated by our platform.

```
Important: The Stream might disconnect sometimes, as such, it is the client's responsibility to reconnect, as it might miss results while disconnected.
```

There are 2 types of data streams available that can be consumed:

##### 1. firehose

_Endpoint_: https://stream.api.binaryedge.io/v1/firehose

_Description_: This stream contains all data generated by Binaryedge own scans, these include WorldWide Scans. This stream is restricted, based on your client account. This stream does not contain data generated from jobs requested by clients.

##### 2. stream

_Endpoint_: https://stream.api.binaryedge.io/v1/stream

_Description_: This stream contains all data generated by your requested Jobs.

### Supported Types

There are 2 types of requests.

##### 1. scan
The scan type will request a scan on the targets and will launch the modules against the detected open ports. It should be used against a large number of targets and its function is to filter responding IPs.

##### 2. grab
The grab type will try to gather information directly from the targets.
Should be used against a small number of targets.


### Supported Modules

##### 1. ssh

_Description_: Extract SSH details, e.g. key and algorithms for SSH servers

_Detailed documentation_: [ssh module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/ssh.md "ssh")

##### 2. ssl
_Description_: Extract SSL details e.g. type of encryption

_Detailed documentation_: [ssl module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/ssl.md "ssl")

##### 3. vnc
_Description_: Extract VNC details and screenshot

_Detailed documentation_: [vnc module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/vnc.md "vnc")

##### 4. service
_Description_: Extract detailed product specific information, e.g. product name, version, headers, scripts. If you just want product name and version, consider using the faster "service-simple".

_Detailed documentation_: [service module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/service.md "service")

##### 5. service-simple (beta)
_Description_: Extract basic product specific information, e.g. product name, version. This module is much faster than "service", since it returns less information.

_Detailed documentation_: [service-simple module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/service-simple.md "service-simple")

##### 6. http & https
_Description_: Extract HTTP/HTTPS information, e.g. HTTP headers, HTTP status codes, HTTP body, and redirects information. Follows up to 5 redirects.

_Detailed documentation_: [http & https module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/http.md "http")

##### 7. telnet
_Description_: Extract Telnet information, e.g. Will, Do, Don't Won't commands.

_Detailed documentation_: [telnet module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/telnet.md "telnet")

##### 8. rdp
_Description_: Extract RDP details and screenshot

_Detailed documentation_: [rdp module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/rdp.md "rdp")

##### 9. x11
_Description_: Extract x11 screenshot

_Detailed documentation_: [x11 module documentation](https://github.com/binaryedge/api-publicdoc/blob/master/modules/x11.md "x11")


##### Custom Modules
Note: If you want a custom-made module, please contact BinaryEdge.

### Configurations
It is possible to set module specific configurations on the job requests. For example, the HTTP module allows the configuration of the Host and the User Agent HTTP headers.
The configuration should be set in the "config" key at the same json level of the requested module.

Example:

```
{
   "type": "scan",
   "description": "test a bunch of networks",
   "options": [
       {
         "targets": ["xxx.xxx.x.x/xx","xxx.xxx.x.x/xx"],
         "ports": [
           {
            "port": 80,
            "modules": ["http"],
            "config":
            	{
            		"user_agent":"Test user Agent", 
            		"host_header":"google.com"
            	}
           }]
       }
     ]
 }
 ```
 
#### Available configurations

- **Host header** - Change HTTP Host header. Available for **HTTP** and **HTTPS** module.
- **User Agent** - Change HTTP User Agent. Available for **HTTP** and **HTTPS** and **Service** modules.
- **SNI** - Set HTTPS Server Name Indication. Available for **HTTPS** and **SSL** module.
 

### GET /v1/replay/job_id - Job Replay

To retrieve the results from a previously requested job, you can replay the stream with this endpoint.

```
curl https://stream.api.binaryedge.io/v1/replay/JOB_ID -H "X-Token:InsertYourClientToken"

HTTP/1.1 200 OK
<Stream results from request job>
```

### POST /v1/tasks/job_id/revoke - Job Revoke

To cancel a requested job:

```
curl -XPOST https://api.binaryedge.io/v1/tasks/JOB_ID/revoke -H  "X-Token:InsertYourClientToken"

HTTP/1.1 200 OK
{"message": "Job revoked"}
```

### Job Status (Beta)

In order for you to know the status of your jobs we provide information in 2 distinct ways:


#### 1. GET /v1/tasks/job_id/status - Status Endpoint

To check the current status of a Requested job:

```
curl https://api.binaryedge.io/v1/tasks/<job_id>/status -H "X-Token:InsertYourClientToken"

HTTP/1.1 200 OK
{"message":"<STATUS>"}
```

Where Status can be:

  * "Requested": Job was requested successfully;
  * "Revoked": Job was revoked by user;
  * "Success": Job completed successfully;
  * "Failed": Job completed, but did not finish.


#### 2. Status Messages

In your stream you will find messages providing insight on the current status of your jobs:

***When a Job is created:***

```
{
	"origin": {
		"job_id": "c4773cb-aa1e-4356eac1ad08",
		"type": "job_status",
    ...
	},
	"status": {
		"success": null,
		"started": null,
		"completed": null,
		"revoked": null
	}
}
```

***Job is completed***

```
{
	"origin": {
		"job_id": "c4773cb-aa1e-4356eac1ad08",
		"type": "job_status",
    ...
	},
	"status": {
		"success": true,
		"started": true,
		"completed": true,
		"revoked": false
	}
}
```

Meaning of the status fields:

  * "success": If the job was completed successfully, with no problems.
  * "started": When a job is requested, it's put in to a Queue;
  * "completed": If the job is completed, no more results will be sent;
  * "revoked": If the job was canceled by the user or not.



### Query Endpoints

In order for you get historical information on certain ips
These endpoints allow the following parameter:
* page The number of the page of results, example: curl https://api.binaryedge.io/query/image?ip=8.8.8.8&page=2
* ip The ip getting information from, example: curl https://api.binaryedge.io/query/image?ip=8.8.8.8
* ipRange A range of ips for querying scanned information, example: curl https://api.binaryedge.io/query/image?ipRange=8.8.8.4-8.8.8.8
* cidr A cidr representing a class of ips, example: curl https://api.binaryedge.io/query/image?cidr=61.0.0.0/8

#### GET /v1/query/image - VNC Endpoint
### Error Messages

Querying for an IP that is not on record:

```
HTTP/1.1 404 Not Found
{"message": "No record found."}
```

Sending invalid Token:

```
HTTP/1.1 400 Bad Request
{"message":"Unauthorized"}
```

### Response
```
curl -v https://api.binaryedge.io/v1/query/image?ip=XXX.XXX.XXX.XXX -H 'X-Token:'
```
```
{
  "origin": {
    "country": "us",
    "type": "vnc",
    "ts": 1460035078342,
    "module": "grabber"
  },
  "target": {
    "ip": "61.252.162.16",
    "port": 5900
  },
  "result": {
    "data": {
      "version": "3.889"
    }
  }
}
```
#### GET /v1/query/raw - Raw IP Data Endpoint
### Error Messages


Querying for an IP that is not on record:

```
HTTP/1.1 404 Not Found
{"message": "No record found."}
```

Sending invalid Token:

```
HTTP/1.1 400 Bad Request
{"message":"Unauthorized"}
```

### Response

```
curl -v https://api.binaryedge.io/v1/query/raw?ip=XXX.XXX.XXX.XXX -H 'X-Token:'
```
```
{
  "origin": {
    "country": "uk",
    "module": "grabber",
    "ts": 1464558594512,
    "type": "service-simple"
  },
  "target": {
    "ip": "222.208.183.136",
    "protocol": "tcp",
    "port": 992
  },
  "result": {
    "data": {
      "state": {
        "state": "open|filtered"
      },
      "service": {
        "name": "telnets",
        "method": "table_default"
      },
      "total_delta": 4.775846004486084
    }
  }
}

```

### FAQ

**Q: What is the sample parameter?**

**A:** The Sample parameter is used to define how many open ports the platform needs to find before stopping the scan. It is useful to test modules and different configurations for each module (that we are adding in the future). This parameter is optional - by default the scan stops only after scanning the entire list of IP addresses and ports.


**Q: How can I consume the stream?**

**A:** The stream outputs to STDOUT, allowing you to consume it in different ways. For example:

- Direct the stream to a file:
    - `curl https://stream.api.binaryedge.io/v1/stream -H "X-Token:InsertYourClientToken" > file.txt`
- Pipe the stream to a custom application you developed to process it:
    - `curl https://stream.api.binaryedge.io/v1/stream -H "X-Token:InsertYourClientToken" | application_name `


**Q: What should I do if I get a error 500?**

**A:** In this case, you should contact support@binaryedge.io



**Q:** How do I scan multiple hosts with one request?

**A:**

```
options: [{
   "targets": [array of cidrs (string)],
   "ports": [{
       "port": int,
       "modules": [array of module names (string)],
       "sample": int
   }]
}]
```

for example:

```
{
   "type": "scan",
   "description": "test a bunch of networks",
   "options": [
       {
         "targets": ["xxx.xxx.x.x/xx","xxx.xxx.x.x/xx"],
         "ports": [{
            "port": 995,
            "modules": ["service"]
           },
           {
            "port": 22,
            "modules": ["ssh"]
           }]
       }, {
         "targets": ["xxx.xxx.x.x/xx"],
         "ports": [{
            "port": 5900,
            "modules": ["vnc"]
         }]
       }
     ]
 }
 ```
