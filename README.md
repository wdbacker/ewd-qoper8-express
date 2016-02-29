# ewd-qoper8-express: Express integration module for ewd-qoper8
 
Rob Tweed <rtweed@mgateway.com>  
24 February 2016, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)


## ewd-qoper8-express

This module may be used to integrate Express with ewd-qoper8, for handling incoming HTTP and WebSocket messages within ewd-qoper8's worker processes.

## Installing

       npm install ewd-qoper8-express
	   
## Using ewd-qoper8-express with ewd-qoper8

### Setup

     var express = require('express');
     var bodyParser = require('body-parser');
     var qoper8 = require('ewd-qoper8');
     var qx = require('ewd-qoper8-express');

     var app = express();
     app.use(bodyParser.json());
     var q = new qoper8.masterProcess();
     qx.init(q);

     // define any routing here (see next sections)

     q.on('started', function() {
       // specify your worker process module for handling messages
       this.worker.module = 'ewd-qoper8/lib/test/express-module';
       // Express to listen on port 8080
       var server = app.listen(8080);
     });

### Handling messages

This example will queue an incoming POSTed JSON payload, send it to a worker process for processing, and then return the
result object as an HTTP application/json response

     app.post('/qoper8', function (req, res) {
       qx.handleMessage(req.body, function(resultObj) {
         // optionally remove the ewd-qoper8 finished property
         delete resultObj.finished;
         res.send(resultObj);
       });
     });

### Using the default router

This packages up REST/HTTP requests as messages that are then handled by the handleMessage() function

eg:

      app.use('/vista', qx.router());

The queued messages will have the following properties:

- type: combines the route and the 2nd part of the URL path.  eg /vista/login requests would have type: 'vista.login'
- method: matches the HTTP method
- headers: contains the HTTP request headers (req.headers)
- params: contains req.params
- path: the URL path that followed the main route (eg everything after /vista)
- query: contains req.query (ie any URL name/value pairs)
- content: for POST requests, the parsed JSON payload 

Two events are emitted in the worker on receipt of these messages:

- expressMessage: all express/qoper8 messages can be handled with this.  

      worker.on('expressMessage', function(messageObj, send, finished) {...});

If expressMessage events aren't handled, then each message will trigger an event based on the message type value, eg:

      worker.on('vista.login', function(messageObj, send, finished) {...});

For both events, the callback function takes 3 arguments:

- messageObj: the message object (as described above)
- send: function for sending a message back to the master process without releasing the worker process back to the available pool
- finished: function for sending a message back to the master process and releasing the worker process back to the available pool

The send and finished functions take a single argument: response object

Error responses should be defined as follows:

      finished({
        error: 'Error message here...'
      });

You can optionally specify an HTTP status code for the error response.  By default it will be 400.

     finished({
       error: 'Error message here...',
       status: {
         code: 403
       }
     });

If an Express/qoper8 message is not handled by either an 'expressMessage' or type-specific handler, then an error message will be
sent to the master process, telling it that no handler was available for the message, and the worker will be released
back to the available pool.


For more details and documentation, see:
 [http://gradvs1.mgateway.com/download/ewd-qoper8.pdf](http://gradvs1.mgateway.com/download/ewd-qoper8.pdf)


## License

 Copyright (c) 2016 M/Gateway Developments Ltd,                           
 Reigate, Surrey UK.                                                      
 All rights reserved.                                                     
                                                                           
  http://www.mgateway.com                                                  
  Email: rtweed@mgateway.com                                               
                                                                           
                                                                           
  Licensed under the Apache License, Version 2.0 (the "License");          
  you may not use this file except in compliance with the License.         
  You may obtain a copy of the License at                                  
                                                                           
      http://www.apache.org/licenses/LICENSE-2.0                           
                                                                           
  Unless required by applicable law or agreed to in writing, software      
  distributed under the License is distributed on an "AS IS" BASIS,        
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
  See the License for the specific language governing permissions and      
   limitations under the License.      
