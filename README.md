# Making-A-Bandwidth-App

## Setup to create an app 
1. Make directory: mkdir name
2. Change into that directory: cd name
3. Use npm to create a .json file: npm init and follow the instructions 
	a. Name: name of project 
	b. Version: version 
Description: description of the app 
Main (fills out as index.js) 
scripts : echo ‘no test yet’ 
Keywords: helps npm find your package when people search for the keywords 
Author: your name 
License: MIT 
Open the text editor: stt
Create index page: index.js
In package.json add start to script: "start": "node index.js"
Add the dependencies to index.js: 
var Bandwidth = require("node-bandwidth");
var express = require("express");
var app = express();
var bodyParser = require("body-parser");
var http = require("http").Server(app);
 
Install the dependencies: 
npm install --save node-bandwidth
npm install --save express
npm install --save body-parser
 
Run index, download dependencies: npm start 
Open back to home directory: cd
Open ngrok: ./ngrok http 3000
Create a new tab in terminal
Return to project folder: cd -
Open index.js file: stt (if not already open) 
Create new client using Bandwidth interface 
Add user ID 
API Token 
API Secret 
 
var client = new Bandwidth({
    // uses my environment variables. Go to dev.bandwidth.com, look under Account -> API Information -> Credentials OR .zsrh file
    userId    : process.env.BANDWIDTH_USER_ID, // <-- note, this is not the same as the username you used to login to the portal
    apiToken  : process.env.BANDWIDTH_API_TOKEN,
    apiSecret : process.env.BANDWIDTH_API_SECRET
});
 
Add necessary apps: 
app.use(bodyParser.json());
//use a json body parser
app.set('port', (process.env.PORT || 3000));
//set port to an environment variable port or 3000
 
##Send a Message 
Your Bandwidth number sends a text/media message to your phone.
 
Create methods 
Method 1: sendMessage sends a message
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
 
Method 2: messagePrinter prints message to console (helper method) 
var messagePrinter= function (message){
    console.log('Using the message printer');
    console.log(message);
}
 
 
Create a number variable with the to and from numbers 
var numbers = {
    to: "+1#######",	        //number to send to
    from: "+1##########" //Bandwidth number
};
 
Send message using the numbers 
sendMessage(numbers);
 
##Message Call Back
When you run the app and message the bandwidth number, it sends a text/media message to your phone.
 
Add all methods from Send a Message. 
Set up website 
    app.get("/", function (req, res) {
    console.log(req); 
    res.send("Text on Website");
    //res.send(can be a website); ***Reroutes to other website?
});
 
Create callback method 
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
 
Make the website listen 
http.listen(app.get('port'), function(){
    //once done loading then do this (callback)
    console.log('listening on *:' + app.get('port'));
});
 
Login to dev.bandwidth.com
Choose the My Apps tab 
Select create new 
Choose post 
Choose message 
 In the message callback box, enter the web address /message-callback (ex. http://2ab58988.ngrok.io/message-callback)
Save 
Add number by checking the box next to the number you want to use. 
Run using npm start. Test by texting the ‘from’ number, and you should get your automated response message back.
 
##Create Outbound Call 
Bandwidth number calls your number
Follow the setup instructions 
Create a call method
 
var createCall = function(toNumber, fromNumber){
	console.log("to: " + toNumber);
	console.log("from: " + fromNumber);
	return client.Call.create({
		from: fromNumber,
		to  : toNumber
		})
};
 
Create call entry point 
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
 
 
##Add Callback Listener to Outbound Call
Each time the call status changes (answered, hungup, declined, etc.) the program will be notified. 
Follow the instructions to create a call. 
In the create call method, add callbackUrl as a parameter 
var createCallWithCallback = function(toNumber, fromNumber, callbackUrl){
	console.log("to: " + toNumber);
	console.log("from: " + fromNumber);
	return client.Call.create({
		from: fromNumber,
		to  : toNumber,
		callbackUrl: callbackUrl
	})
};
 
 
Create outbound callback entry point. This is the url that will recieve the information about the call 
 
Add the callback url to the call entry point 
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
 
##Example: Speak Audio from Outbound Call 
Create an outgoing call with a callback listener
Check if the call is answered (this holds the program from speaking if the call is not answered or is still ringing) 
Helper Method: 
var checkIfAnswer = function(eventType){
	return (eventType === "answer");
}
 
Speak the sentence 
Helper Method:
var speakSentenceInCall = function (callId, sentence){
	return client.Call.speakSentence(callId, sentence);
}
 
Once the sentence is spoken, hang up the call. 
Helper Method:
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
 
 
##Call Call-Back
The user can call the Bandwidth number and the program will answer. It then speaks a sentence and hangs up when done. 
 
Prepare by following the Setup instructions.
Set up website. refer to Message Call Back Step 2.
Create listener. refer to Message Call Back Step 4.
Create callback method:
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
Login to dev.bandwidth.com
Choose the My Apps tab 
Select create new 
Choose post 
Choose voice 
 In the voice callback box, enter the web address /call-callback (ex. http://2ab58988.ngrok.io/call-callback) 
Check the “Automatically answer incoming calls” box 
Save 
Add number by checking the box next to the number you want to use. 
Run using npm start. Test by calling the bandwidth number chosen in step 12. If it works, it should speak a sentence and hang up.
 
##Combining Voice and Messaging Callbacks
Under My Applications on dev.bandwidth.com, choose the BOTH option to link the two types of callbacks to one Bandwidth number. Also only one website and one listener must be created to run the application. 
 
##Other info 
Messages are sent back using JSON. It comes back in a header and body format. The header contains authentication while the body has all the content. Bandwidth bodies come back as: 
 
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
 
Any of these fields can be referenced by body.’field’ (ex. body.direction or body.from) 
 
If the program just wants the to and from numbers to be hard coded in (no flexibility), the programmer can also create a message like this: 
client.Message.send({ 
        from: "+17204407441",
        to: "+13035659555",,
        text : "Text here",
        //media: "https://imageurl.jpg"
    })
They can also create a call like this: 
client.Call.create({
    from: "+17204407441",
    to: "+13035659555",
    callbackUrl: "http://2ab58988.ngrok.io"
})
 
 
 
