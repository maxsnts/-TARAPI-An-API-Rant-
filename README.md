# “TARAPI” An API Rant! 

I have to call it something… we always do!

Yeah Yeah it stands for "The-Anti-Rest-API" <br/>
Lost 50% of readers right there... 45% will complain... 10% will give this some thought! <br/>
Wait, what? 105%? Ohh 5% will complain without reading... right! <br/>

Disclaimer: I will use a lot of "**you** do this... **you** did that..." <br/>
It is deliberate because i want to strike a nerve, to feel personal, uncomfortable even. <br/>
However it is not, and never was, meant to be offensive. <br/>
I have done some of the things that i complain here, so i'm not excluding myself from it. <br/>

Let me put it out there, I think a REST API are crap. <br/>
Sure… this will ruffle some feathers! <br/>
Its just something that became popular and spread like wildfire. <br/>

At some point, someone proposed this new way to build an API, one that “_took advantage_” of the underlying structure of the http protocol. <br/>
People liked it without question the biggest mistake in this assumption: <br/>
“Why is the protocol being used to indicate what must be done or has been done? Just because http works that way? Is it reason enough?”

Its bad for one main reason: **It conflates transport with execution**<br/>
There are plenty to dislike about REST APIs but they all have the same root.

This promiscuity between transport and execution was created without thinking that an API could and should be build in a way that allow its use over multiple different transport methods, however, and REST API relies on the use of Http Status codes, Verbs, and Urls… none of them can be replicated in other transports.

With a REST API you are locked not being able to use it over web sockets for instance.<br/>
What about TCP sockets… good old sockets?<br/>
Message queue systems (MQTT)?<br/>
Or… even if File Systems? (sure i would not do it either!)<br/>
Homing Pigeons?<br/>

The point is that an API is a method of transferring information and executing commands, preferably using any method of transportation that we have available. On the other hand an REST API allows for only one way of transferring the data.

![](transport.png) 

Authentication is outside of the scope of this rant as it pertains to the transport and would need to be handled differently in all of them.<br/>
The API layer itself would not do authentication.



Lets take it one step at a time, starting by the logical last one... the response:

### Http Status Codes

Do the codes mean anything? No, they do not. The same code can be used by the API or the server to indicate massively different things. We must almost always look at the payload to see what was done.

From somewere on the interwebs...<br/>
_“A 403 error response indicates that the client’s request is formed correctly, but the REST API refuses to honor it i.e. the user does not have the necessary permissions for the resource.”_

Notice the error in the assumtion: _"request is formed correctly but the REST API refuses to honor it "_! <br/>
No, sorry, no. When you get a 403 form any API you can not assume this, ever.<br/>
The request may not have gotten to the API.<br/>

How do i know the difference between that and the SysAdmin messing up and removing all access to the API? Do I have to go and look at the payload? See if its valid?  <br/>
Why the hell do we need the status codes for then?<br/>



And don’t get me started on the 500… be honest, how many times did you see APIs responding with a 500 to indicate something went wrong with the stuff you sent?

Some times you get a 500 and you have to go and see the payload to get what happened, but some times you get a 500 because the server craped itself and there is not even a valid payload. <br/>
Its nonsense, all of it.

Oh... and what about the mixed response where some responses come in html or plain text instead of valid json payloads?<br/>
Well, yes you can say that is a bad implementation. Granted, but that exists only because transport/protocol and execution got mixed.
Remove the assumption that there is an http protocol available and the html responses will most probable go away to.

We should not use status codes at all as those indicate the status of the transport and not the execution.

When using http/https it should be like this:

200 = All requests got to the API… ALL OF THEM no matter what happened inside the API. You want so say that the user has no access to the resource? Do it in the response payload!

Anything else… its just the protocol talking not the API<br/>
Anything else and you would not be required to look at the payload because you knew from the start that it was not the API responding.

401 = API needs authentication (outside of the scope of this rant)<br/>
500 = “Crap the server is not good”<br/>
403 = “Someone messed up”<br/>
429 = “Hold your horses”<br/>

None of this has to do with the Execution and should not be treated the same way.<br/>
If we try to use the same _"TARAPI"_ with any other transport it would still work.<br/>

TCP/Sockets:

- Client open connection<br/>
- Provides authentication according to whatever was specified for that transport<br/>
- If accepted, it starts sending payloads and receiving payloads… no status codes, no verbs, no urls.

Files (huum, its just an example):

- Provides authentication according to whatever was specified for that transport<br/>
- Client saves a file with a payload on a folder accessible to both the server and client (nfs, smb, ftp, ssh… go wild, it just an example)<br/>
- Server gets the file, writes a file with response<br/>
- Clients reads response files from folder<br/>

Pigeons:

- Client prints payload on paper<br/>
- Monkey attaches payload to pigeon leg<br/>
- Monkey gets payload from pigeon leg... OCR's it<br/>
- ...<br/>

... the point is: Why build an API that has no versatility at all by locking it to the http protocol?<br/>

While we are in the topic of responses, make them consistent.<br/>
Make the response always the same format even if coming from different places in the API.<br/>
Even if you need to do things asynchronously, CallBacks, AsyncCalls, WebHooks, whatever you want to call them.<br/>

Example: (and only that)<br/>
```
{
	"ResponseFrom": "MethodSomething",
	"Status": "OK", //or NOK codes 
	"Errors": [ ... ], //Describing catastrophic errors
	"Warnings": [ ... ], //Something to pay attention to but not catastrophic
	"Info": [ { "All fine skipper!" } ], 
	"Payload": 
	{
		"ObjectID": "123456",
		...
	}
}
```

Think of responses in a _parsable_ way. They will most definitely not be read by a Human so they need to be consistent.<br/>
Now, the payload would obviously be different from method to method, but anything outside of the _Payload_ json node should be the same between all methods.

Consistency will even allow you to have some users working assyncronasly and some users, perhaps less technically proficient, working synchronously by sacrificing performance. All you need is a user setting or who knows the user could request sync or async when doing the request. (Assuming the inner workings of the API allow for it) This gives you versatility.

### Verbs and URLs

This is just more of the same, mixing transport with execution.<br/>
Just do it all with the payload.<br/>
If you do, it will allow anything of the above, and you gain a few things even if you use it in http.<br/>

In rest, We need payload + Url + verb.<br/>
In here... the payload itself describes all of the operation instead of only part of it.<br/>
Logging becomes simpler, and if needed, your internal tools can even reprocess those payloads later (redo actions, solve problems, run them in Dev) because they are complete, all the information is there.

### IDs

For the love of God don’t force your clients to keep record of your internal Ids!<br/>
The API should work with the ID of the original object.<br/>
Now its fine (read much better) for your internal system to use internal ids, but have the API match object using the client own IDs... not yours.

Lets see, there is mainly 3 types of APIs:<br/>
1 - Clients create objects on the API system and deal only with those remote objects<br/>
2 - Clients create objects on their own system and then sync them to the API<br/>
3 - Hybrids, where the client can do both.<br/>

Most APIs I have worked with are of the 2 and 3 type, but they always force the client to keep a record of the remote API id? Why?

Don’t you like to sleep well at night? So, instead of solving the problem on your side ONCE, you force each and EVERY client to solve the same problem over and over and over again.<br/>
This will without a doubt be a source of problems, bad implementations, errors, support tickets… and ultimately management steaming from the ears like a cartoon character!

![](steam.jpg)

If a client needs to send his objects to multiple APIs, he will have to create a way to keep a record of multiple API ids for that same object. A 1-to-N table is the most common.

If you on the other hand keep the record, you only need a simple extra column on the object table. **Nothing more**.

“Hei what about conflict?” you ask

ClientA has an Object with ID 1<br/>
ClientB has an Object with ID 1

So what, you identify the object by ClientID+ObjectID right? Where is the problem?<br/>
The ClientID you already know from the authentication process, the ObjectID is sent to you on the request payload.

Lets call it SyncID and see how it works:

Example 1:

- ClientA creates an Object on his own system with ID “1”<br/>
- ClientA send you a request where it asks the API to save ObjectID1 into you DB<br/>
- You save it and reply with a payload containing the status of the operation and  ObjectID1


Even if the API works asynchronously, the payload is the same.

Example 2:

- ClientA send you a request where it asks the API to save Object but sends no ID<br/>
- You save it and reply with a payload containing the status of the operation and  your own ID. (you can just use your primary key if you want, but copy it to the SyncID column)

If you don’t want to support clients creating object directly in you system you can just reject payloads without and ID. Thats it. And you can even control this on a by client basis.<br/>
Some clients can, some can’t.<br/>

No matter the case your API can work the same way and support both scenarios.<br/>
It comes at no cost at all when compared with the problems you can avoid by eliminating the need for all your clients to keep a record of your IDs.

- You simplify their implementation. Thats a big one even if you ignore it.

- If the clients looses that record, he can just resend the Objects again and no duplicates will be created on your side.

- If they can get a list o items from the API, they can easyly check if there is anithing in there that should not be (by checking theur ID) and delete it.

Look, like it or not we are in the business of “support ticket elimination/prevention”.<br/>
And I am sure that solving the problem of the SyncID once on the API side is far better than forcing your clients, all of them, to do it.
Help your users, don't force them to keep you IDs. They may not have a way to do it.<br/>
Do you want to loose users just because you don't want a extra column to hold theur ID on your DB?

### Content manipulation

While we are in the topic of taking work from your users.<br/>
Anything you feel is particular to you... you should do it! Period. Sorry.<br/>
Before you feel like forcing your users to do something, ask if any of your compatitours have no such requisite.

Example:<br/>
Why ask that all users remove \r\n from texts (!?) and send "BR" instead?<br/>
Do it on the API!<br/>
Well, this is a deeper problem...<br/>
I personaly thing that saving "BR" in you database on a piece of text that is not Html is just dumb! <br/>
And yet... I can seam fo format this rant without them beacuse github markup sucks!  

### Summary

What does the user want to do?<br/>
What is the minimum necessary for the user to do what he wants?<br/>
(usually: AUTH, LIST, GET, POST, DELETE)<br/>

Everything else: Optional!<br/>
Everything else: Is your job to deal with it!<br/>

(I'm sure this rant is not over)





