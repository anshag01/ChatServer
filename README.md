# ChatServer
A chat server implemented in C

## Definitions and Notes

	1) A chat message is the text sent from one user to another.

	2) A message, we mean a message sent from client to server or server
	   to client. A message may include a chat message. A message 
	   has exactly one \n appearing at the message end.

	Example: 

		list\n 

	is a message of length 5 in cluding the \n. 
	There is no chat message here.

		message:ansh:sam:Hey sam how are you doing?\n

	is a message which contains a chat message "Hey sam how are you doing?"

	3) Each client is associated with a single user. 
           A user is an alpha numeric string
	   consisting of at most MAX_USER_LEN characters.

	4) A client sends the following message to the server to identify 
	   the user associated with this client.

		register:USER\n

	5) Bounds on messages and message parts are as follows: 

		MAX_USER_LEN 64
		MAX_CHAT_MESSAGE_LEN (1024)
		MAX_MESSAGE_LEN (7+1+MAX_USER_LEN+1+MAX_USER_LEN+1+MAX_CHAT_MESSAGE_LEN+1)

		Note: The +1 at the end of MAX_MESSAGE_LEN is for the \n message termination

	6) CONVENTION: Messages in transit will be \n terminated. Messages at rest
	   will have \n replaced by \0.

	7) If the server receives a message that is too long, > MAX_MESSAGE_LEN
           then the server will close the connection. 

	8) The server always replies to each message it receives. If it receives
	   an invalid message (other than a very long message) it responds with 

	   ERROR

## Messages
Below is a list of all possible messages to the server and the 
servers response. The interaction between the client and server consists
of a client initiated request/response cycle. 

-------------------------------------------------------------------------------

	REQUEST: (connect to server)
	RESPONSE: 

-------------------------------------------------------------------------------
	REQUEST: 
		register:USER\n

	RESPONSE: (server associates USER with this client)
		registered:USER\n  

	RESPONSE: (if USER is already registered, then USER can not be 
		associated with this client)

		userAlreadyRegistered\n  
-------------------------------------------------------------------------------
	REQUEST: 
		getMessage\n
	
	RESPONSE: (return the first message on the queue to TO_USER)
		message:FROM_USER:TO_USER:CHAT_MESSAGE\n
	
	RESPONSE: (if there are no messages for this user)
		noMessage\n

-------------------------------------------------------------------------------
	REQUEST: 
		list\n
	
	RESPONSE: 
		users:USER1 USER2 USER3 ... USERn\n
	
	NOTE: At most 10 users are sent back.
	
-------------------------------------------------------------------------------
	REQUEST: 
		message:FROM_USER:TO_USER:CHAT_MESSAGE\n
	
	RESPONSE: (the above message is queued to send to TO_USER)
		messageQueued\n
	
	RESPONSE: (if there are too many messages queued for TO_USER)
		messageNotQueued\n
	
	RESPONSE: (if TO_USER is not found in the list of conected clients)
		noToUser:TO_USER\n

	RESPONSE: (in case FROM_USER does not match the registered:USER )
		invalidFromUser:TO_USER\n

-------------------------------------------------------------------------------
	REQUEST: (long message, longer than MAX_MESSAGE_LEN)
	RESPONSE: 
		connection closed
-------------------------------------------------------------------------------
	REQUEST: 
		(anything not recognizable)\n
	RESPONSE: 
		ERROR
-------------------------------------------------------------------------------
	REQUEST: 
		quit\n
	RESPONSE: 
		closingConnection\n
-------------------------------------------------------------------------------

Sample transcript: C indicates client, S indicates server

The following is a conversation I am having with myself using 
chatServer.

In a terminal:

	$ q0/chatServer 7009   # use your port number!

In another terminal:

	$ nc 127.0.0.1 7009               # same port as above

I am chatting with myself.

	C register:ansh
	S registered
	C list
	S users:ansh
	C message:ansh:ansh:this is message 1
	S messageQueued
	C message:ansh:ansh:this is message 2
	S messageQueued
	C invalid stuff
	S ERROR
	C register:ansh
	S userAlreadyRegistered
	C getMessage
	S message:ansh:ansh:this is message 1
	C getMessage
	S message:ansh:ansh:this is message 2
	C getMessage
	S noMessage
	C message:ansh:sam:hey sam, whats up?
	S invalidToUser:sam
	C message:imposter:ansh:Hey ansh!
	S invalidFromUser:imposter
	C quit
	S closing

