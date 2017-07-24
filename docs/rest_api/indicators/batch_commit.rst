Batch Upload: Indicators
------------------------

Sample Batch Create request

.. code::

     POST /v2/batch/                 
     {                               
     "haltOnError": "false",         
     "attributeWriteType": "Replace", 
     "action": "Create",             
     "owner": "Common Community"     
     }                               

Server Response on Success

.. code::

     HTTP/1.1 201 Created                       
     {                                          
     batchId: "123"                             
     }                                          

Server Response on Insufficient Privileges

.. code::

     HTTP/1.1 403 Forbidden                     
     {                                          
     status: "Not Authorized",                  
     description: "Organization not authorized  
     for batch"                                 
     }                                          

Server Response on Incorrect Settings

.. code::

     HTTP/1.1 403 Forbidden                     
     {                                          
     status: "Not Authorized",                  
     description: "Document storage not enabled 
     for this instance"                         
     }                                          

The following is an example of a Batch Indicator Input file:

.. code:: json

    {
      
      "ownerName": "<String>", 
      "type": "<String>",
      "rating": "<BigDecimal>",
      "confidence": "<Short>",
      "description": "<String>",
      "summary": "<String>",
      
      "tag": [
        {
          "name": "<String>"
        }
            ]
    }

Sample Batch Upload Input File request

.. code::

     POST /v2/batch/123                                               
                                                                      
     Content-Type: application/octet-stream; boundary=[boundary-text] 
     Content-Length: <data_size>                                         
     Content-Encoding: gzip       
     [boundary-text]                                                  
     <uploaded_data>                                              

Server Response on Success

.. code::

     HTTP/1.1 202 Accepted                
     {                                    
     status: "Queued"    
     }                                    

Server Response on Overlarge Input File

.. code::

     HTTP/1.1 400 Bad Request             
     {                                    
     status: "Invalid",                   
     description: "File size greater than 
     allowable limit of 2000000"          
     }                                    

Sample Batch Status Check request

.. code::

    GET /v2/batch/123


Server Response on Success (job still running)

.. code::

    HTTP/1.1 200 OK
    {
    status: "Running",
    }

Server Response on Success (job finished)

.. code::

    HTTP/1.1 200 OK
    {
    status: "Completed",
    errorCount: 3420,
    successCount: 405432
    unprocessCount: 0
    }

Sample Batch Error Message request

.. code::

    GET /v2/batch/123/errors

Server Response on Success (job still running)

.. code::

    HTTP/1.1 400 Bad Request
    {
    status: "Invalid",
    description: "Batch still in Running state" }

Server Response on Success (job finished):

.. code::

    HTTP/1.1 200 OK
    Content-Type: application/octet-stream ; boundary=
    Content-Length: 
    Content-Encoding: gzip

Create Batch Endpoint

The Batch API allows bulk Indicator creation and deletion via the HTTP
POST method. After creating a batch, an Indicator file is uploaded. The
content of the file must be valid JSON, with content and format
mimicking the data structure of the Bulk JSON file download. A file
upload instantly triggers a batch job to begin processing the data. The
Batch API is restricted to Indicators and will improve performance when
importing large amounts of data.

.. note:: Document Storage is required to use the Batch API.

The Batch Create resource creates a batch entry in the system. No batch processing is triggered until the batch input file is uploaded. The table below displays the fields required for the Batch Create message.

+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
| BatchConfig Message | Values          | Description                                                                                                       |
+=====================+=================+===================================================================================================================+
| haltOnError         | true            | The batch process will stop processing the entire batch the first time it reaches an error during processing.     |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
|                     | false (default) | If this field is not provided, the default behavior is to continue processing further entities in the input file. |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
| attributeWriteType  | Append          | Append: Add attributes (allow duplicates)                                                                         |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
|                     | Replace         | Replace: Delete current attributes; Add/Validate new                                                              |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
| action              | Create          | Create: Create Indicator                                                                                          |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+
|                     | Delete          | Delete: Delete Indicator (only the ‘summary’ and ‘type’ field are required)                                       |
+---------------------+-----------------+-------------------------------------------------------------------------------------------------------------------+

Batch Indicator Input File Format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: json

    [
      {
            "rating": 3,
            "confidence": 60,
            "description": "a malicious domain",
            "summary": "super-malicious.ru",
            "type": "Host", 
            "associatedGroup": [12345, 54321],
            "attribute": [
                {
                    "type": "AttributeName",
                    "value": "MyAttribute"
                }
                ],
            "tag": [
             {
               "name": "MyTag"
             }
            ]
      }
    ]

The batch upload feature expects to ingest a JSON file consisting of a
list of dictionaries

+---------------------+----------------------+-----------+
| Field               | Data type            | Required? |
+=====================+======================+===========+
| ``rating``          | integer              | Required  |
+---------------------+----------------------+-----------+
| ``confidence``      | float                | Required  |
+---------------------+----------------------+-----------+
| ``description``     | string               | Required  |
+---------------------+----------------------+-----------+
| ``summary``         | string               | Required  |
+---------------------+----------------------+-----------+
| ``type``            | string               | Required  |
+---------------------+----------------------+-----------+
| ``tag``             | list of dictionaries | Optional  |
+---------------------+----------------------+-----------+
| ``attribute``       | list of dictionaries | Optional  |
+---------------------+----------------------+-----------+
| ``associatedGroup`` | list of integers     | Optional  |
+---------------------+----------------------+-----------+

Supported ``type`` values for Indicators:

-  Host
-  Address
-  EmailAddress
-  URL
-  File

.. note:: Exporting indicators via the ``JSON Export`` feature in ThreatConnect will create a file in this format

Upload Batch Input File Endpoint

Batch files should be sent as HTTP POST data to a REST endpoint,
including the relevant ``batchId``

Check Batch Status Endpoint

You may also check the status of a running batch upload job

The server can be configured to restrict the file size. Clients can
submit multiple batches for larger files.

Possible GET response status includes:

-  Created
-  Queued
-  Running
-  Completed

If ``haltOnError`` is set to ‘true’ and an error occurs, then the status
will be set to ‘Completed’, and ‘errorCount’ will be greater than zero.
The ‘unprocessedCount’ field will be greater than zero, unless the
uploaded file did not contain valid JSON.

Partial failures will have an error file with a response having a
‘reason text’, which includes Tag, Attribute, or Indicator errors (fail
on first).