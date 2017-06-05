# _Making A Bandwidth App (node)_

## Prerequisites for creating an app 
1. Make a Bandwidth account. 
	1. Visit http://dev.bandwidth.com/
	2. Choose the `Sign Up` button on the top right hand side of the page. 
	3. Fill out the requested information. The platform will call the number provided and will say a confirmation number. **NOTE:** This is a phone call not a text. 
	4. Login 
2. Install an editor. There are many to chose from but the following applications were made using **sublime** https://www.sublimetext.com/
3. Install **ngrok**. This will allow the program to be live on the internet: https://ngrok.com/
4. Install a program to create a local database if needed. These applications use **Postgres.app** http://postgresapp.com/documentation/install.html
5. Configure the **zsh file**. **Oh my zsh** is a great link for managing the zsh file. https://github.com/robbyrussell/oh-my-zsh
6. Install **nvm**. For a comprehensive setup, follow this link: https://github.com/lukechilds/zsh-nvm
7. Setup environment variables in zshrc file. To find the **ID**, **Token**, and **Secret**, login to http://dev.bandwidth.com/. Select the `Account` tag and scroll down until **API Information** appears. The user ID is displayed in the **User ID** box. Choose the `Credentials` button to retrieve the Token and Secret. Add the following to the zshrc file: 
export CATAPULT_USER_ID=
export CATAPULT_API_TOKEN=
export CATAPULT_API_SECRET=
export BANDWIDTH_USER_ID=
export BANDWIDTH_API_TOKEN=
export BANDWIDTH_API_SECRET=
 
## Testing Application
To test the application, launch Ngrok and Postgres. To launch ngrok, type `./ngrok http 3000` in the terminal on a new tab. To view the webpage, copy the first forwarding link on the ngrok terminal page. Paste this link in a web browser. If everything is connected, when the program is launched, the website will be live and show no errors. Note: If given the error **Failed to complete tunnel connection**, this means that the program is not live or is not setup correctly. 


## Setup to create an app 
1. Make directory: `mkdir name`
2. Change into that directory: `cd name`
3. Use npm to create a .json file: `npm init` and follow the instructions 
	* Name: name of project 
	* Version: version 
	* Description: description of the app 
	* Main (fills out as index.js) 
	* scripts : echo ‘no test yet’ 
	* Keywords: helps npm find your package when people search for the keywords 
	* Author: your name 
	* License: MIT 
4. Open the text editor: `stt`
5. Create index page: index.js
6. In package.json add start to script: `"start": "node index.js"`
7. Add the dependencies to index.js: 

```js
var Bandwidth = require("node-bandwidth");
var express = require("express");
var app = express();
var bodyParser = require("body-parser");
var http = require("http").Server(app);
```

8. Install the dependencies: 
```js
npm install --save node-bandwidth
npm install --save express
npm install --save body-parser
```

9. Run index, download dependencies: `npm start` 
10. Open back to home directory: `cd`
11. Open ngrok: `./ngrok http 3000`
12. Create a new tab in terminal
13. Return to project folder: `cd -`
14. Open index.js file: `stt` (if not already open) 
15. Create new client using Bandwidth interface 
	* Add user ID 
	* API Token 
	* API Secret 
 
```js
var client = new Bandwidth({
    // uses my environment variables. Go to dev.bandwidth.com, look under Account -> API Information -> Credentials OR .zsrh file
    userId    : process.env.BANDWIDTH_USER_ID, // <-- note, this is not the same as the username you used to login to the portal
    apiToken  : process.env.BANDWIDTH_API_TOKEN,
    apiSecret : process.env.BANDWIDTH_API_SECRET
});
```

16. Add necessary apps: 
```js
app.use(bodyParser.json());
//use a json body parser
app.set('port', (process.env.PORT || 3000));
//set port to an environment variable port or 3000
```

# Messaging
## Send a Message 
_Your Bandwidth number sends a text/media message to your phone._
 
1. Create methods 

_Method 1: **sendMessage** sends a message_

```js
var sendMessage = function(params){
    client.Message.send({
        //returns a promise 
        from : params.from, //your bandwidth number 
        to   : params.to,       //number to send to 
        text : "",
        media: "https://img.memesuper.com/ce20eb4f1da26e98771cd1c17a2a5641_who-me-who-me-memes_632-651.png"
    })
//calls back the message id number and catches any errors 
//is this important?
    .then(function(message){
        messagePrinter(message);
        return client.Message.get(message.id)
        //access ID from json can also get to and from
    })
// prints message to console 
    .then(messagePrinter)
 
// catches any errors     
.catch(function(err){
        console.log(err)
    });
}
```

_Method 2: **messagePrinter** prints message to console (helper method)_ 

```js
var messagePrinter= function (message){
    console.log('Using the message printer');
    console.log(message);
}
```
 
2. Create a number variable with the to and from numbers 
```js
var numbers = {
    to: "+1#######",	        //number to send to
    from: "+1##########" //Bandwidth number
};
```

3. Send message using the numbers 
```js
sendMessage(numbers);
```
 
## Message Call Back
_When you run the app and message the bandwidth number, it sends a text/media message to your phone._
 
1. Add all methods from Send a Message. 
2. Set up website 
```js
    app.get("/", function (req, res) {
    console.log(req); 
    res.send("Text on Website");
    //res.send(can be a website); ***Reroutes to other website?
});
``` 

3. Create callback method 
```js
app.post("/message-callback", function(req, res){
// the /message-callback is the extension used for the website (req, res) means the user sends in a request and they get back a response 
//for more about body refer to other info 
    var body= req.body; 
    //let the other know you got the request and we will process it 
    res.sendStatus(200);
    if(body.direction === "in"){
        var numbers={
            to: body.from, 
            from: body.to
        }
        sendMessage(numbers);
    }
 
});
```

4. Make the website listen 
```js
http.listen(app.get('port'), function(){
    //once done loading then do this (callback)
    console.log('listening on *:' + app.get('port'));
});

```

5. Login to dev.bandwidth.com
6. Choose the My Apps tab 
7. Select create new 
8. Choose post 
9. Choose message 
10. In the message callback box, enter the web address /message-callback (ex. http://2ab58988.ngrok.io/message-callback)
11. Save 
12. Add number by checking the box next to the number you want to use. 
13. Run using `npm start`. Test by texting the ‘from’ number, and you should get your automated response message back.
 
# Calling
## Create Outbound Call 
_Bandwidth number calls your number_
1. Follow the setup instructions 
2. Create a call method
 
```js
var createCall = function(toNumber, fromNumber){
	console.log("to: " + toNumber);
	console.log("from: " + fromNumber);
	return client.Call.create({
		from: fromNumber,
		to  : toNumber
		})
};
```

3. Create call entry point 

```js
app.post("/calls", function (req, res){
	var callbackUrl = getBaseUrl(req);
	var body = req.body;
	console.log(body);
	var phoneNumber = body.phoneNumber;
	console.log(phoneNumber);
	createCall(phoneNumber, myBWNumber);
	.then(function(call){
		console.log(call);
		res.send(call).status(201);
	})
 
	.catch(function(err){
		console.log("ERROR CREATING CALL");
	})
});
 
var getBaseUrl = function (req) {
	return 'http://' + req.hostname;
};
``` 
 
## Add Callback Listener to Outbound Call
_Each time the call status changes (answered, hungup, declined, etc.) the program will be notified._
1. Follow the instructions to create a call.
2. In the create call method, add callbackUrl as a parameter 

```js
var createCallWithCallback = function(toNumber, fromNumber, callbackUrl){
	console.log("to: " + toNumber);
	console.log("from: " + fromNumber);
	return client.Call.create({
		from: fromNumber,
		to  : toNumber,
		callbackUrl: callbackUrl
	})
};
```
 
3. Create outbound callback entry point. This is the url that will recieve the information about the call 
 
4. Add the callback url to the call entry point 

```js
app.post("/calls", function (req, res){
	var callbackUrl = getBaseUrl(req) + "/outbound-callbacks";
	var body = req.body;
	console.log(body);
	var phoneNumber = body.phoneNumber;
	console.log(phoneNumber);
	createCallWithCallback(phoneNumber, myBWNumber, callbackUrl)
	.then(function(call){
		console.log(call);
		res.send(call).status(201);
	})
 
	.catch(function(err){
		console.log("ERROR CREATING CALL");
	})
});
 
var getBaseUrl = function (req) {
	return 'http://' + req.hostname;
};

``` 

## Example: Speak Audio from Outbound Call 

1. Create an outgoing call with a callback listener
2. Check if the call is answered (this holds the program from speaking if the call is not answered or is still ringing) 

**Helper Method:**

```js
var checkIfAnswer = function(eventType){
	return (eventType === "answer");
}
 ```

3. Speak the sentence 

**Helper Method:**

```js
var speakSentenceInCall = function (callId, sentence){
	return client.Call.speakSentence(callId, sentence);
}
```

4. Once the sentence is spoken, hang up the call. 

**Helper Method:**

```js
var isSpeakingDone = function (callBackEvent){
	return (callBackEvent.eventType === "speak" && callBackEvent.state === "PLAYBACK_STOP");
}
 
app.post("/outbound-callbacks", function (req, res){
	var body = req.body;
	console.log(body);
	if(checkIfAnswer(body.eventType)){
		speakSentenceInCall(body.callId, "Hello from Bandwidth")
		.then(function(response){
			console.log(response);
		})
		.catch(function(error){
			console.log(error);
		})
	}
	else if (isSpeakingDone(body)){
		client.Call.hangup(body.callId)
		.then (function(){
			console.log ("Hanging up call");
		})
		.catch (function(err){
			console.log("Error hanging up the call, it was probably already over")
			console.log("err")
		});
	}
});
``` 
 
## Call Call-Back
_The user can call the Bandwidth number and the program will answer. It then speaks a sentence and hangs up when done._
 
1. Prepare by following the **Setup** instructions.
2. Set up website. refer to **Message Call Back** Step 2.
3. Create listener. refer to **Message Call Back** Step 4.
4. Create callback method:

```js
app.post("/call-callback", function (req, res){
	var body = req.body;
	res.sendStatus(200);
//If the number ‘answers’, speak sentence
	if (body.eventType === "answer"){
		client.Call.speakSentence(body.callId, "Voice message here.")
		.then(function (res) {
			console.log("speakSentence sent");
		})
		.catch(function (err){
			console.log(err);
		});
	}
	//If message has been spoken, and the playback has stopped, hang up call
	else if (body.eventType === "speak" && body.state === "PLAYBACK_STOP"){
		client.Call.hangup(body.callId)
		.then(function (){
			console.log("Hanging up call");
		})
		.catch(function (err){
			console.log("Error hanging up the call, it was probably already over.");
			console.log(err);
		});
	}
	else{
		console.log(body);
	}
	//Default: print out body
});
```

5. Login to dev.bandwidth.com
6. Choose the My Apps tab 
7. Select create new 
8. Choose post 
9. Choose voice 
10. In the voice callback box, enter the web address /call-callback (ex. http://2ab58988.ngrok.io/call-callback) 
11. Check the “Automatically answer incoming calls” box 
12. Save 
13. Add number by checking the box next to the number you want to use. 
14. Run using npm start. Test by calling the bandwidth number chosen in step 12. If it works, it should speak a sentence and hang up.
 
## Combining Voice and Messaging Callbacks
Under My Applications on dev.bandwidth.com, choose the BOTH option to link the two types of callbacks to one Bandwidth number. Also only one website and one listener must be created to run the application. 
 
# Other info 
* Messages are sent back using JSON. It comes back in a header and body format. The header contains authentication while the body has all the content. Bandwidth bodies come back as: 
 
```js 
{ direction: 'out',
  from: '+17204407441',
  id: 'm-ouyf7kktx3wggehh6iggp6y',
  messageId: 'm-ouyf7kktx3wggehh6iggp6y',
  state: 'sending',
  text: 'ILY',
  media: [],
  time: '2017-05-23T20:48:02Z',
  to: '+13035659555',
  skipMMSCarrierValidation: true }
```
 
_Any of these fields can be referenced by `body.’field’` (ex. body.direction or body.from)_
 
* If the program just wants the to and from numbers to be hard coded in (no flexibility), the programmer can also create a message like this: 

```js
client.Message.send({ 
        from: "+17204407441",
        to: "+13035659555",,
        text : "Text here",
        //media: "https://imageurl.jpg"
    })
```

_They can also create a call like this:_ 

```js
client.Call.create({
    from: "+17204407441",
    to: "+13035659555",
    callbackUrl: "http://2ab58988.ngrok.io"
})
```
 
 
 
