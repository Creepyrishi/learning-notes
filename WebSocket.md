web-sockets are a type of persistent duplex connection between client and the server. unlike HTTP or HTTPs where the server only response to the request, websocket maintains a tcp connection where any side can send message anytime.

in fastapi we can create  a websocker like 
```python
from fastapi import Websocket, Fastapi

app = Fastapi()

@app.websocket("/ws")
async def socket(ws: Websocket):
	await ws.accept() # this is must to accept the ws connection, this might be 
					  # only done after auth, etc 
		
		try:
			while True: # as this is a persistent connecion we need to make it                           # in a always true block
				
				data = await ws.receive_text()   
				await ws.send_text("this is the response")

				# there are other types of receive and send funcion in fastapi
				# currenlty await ws.rec.. blocks the code untill a data is 
				# recived from the frontend however there might be certian 
				# situation were we might need to both listen and send 
				# data at the same time in that case we will asycnio to 
				# run the both task cocurrently
				
		except fastapi.WebSocketDisconnect:
			print("ws disconnected")

```

listening and sending task both at same time
```python
from fastapi import Websocket, Fastapi
import asycnio 

app = Fastapi()

@app.websocket("/ws")
async def socket(ws: Websocket):
	await ws.accept()

	async def recevive_data():
		try:
			while True:
				data = await ws.recevive_text()
				print(data)
		except Exception as e:
			print(e)			
	
	async def send_data():
		try:
			counter = 0
			while True:
				await ws.send_text(counter)
				counter += 1
		except Exception as e:
			print(e)
			
	
	tasks = [asyncio.create_task(recevive_data()),
		asyncio.create_task(send_data())]
		
	done, pending = asyncio.wait(tasks, return_when = asyncio.FIRST_COMPLETED)
	
	for task in pending:
		task.cancel()
```



**Auth websocket**
a browser web-socket cannot set `Authorization: Bearer <token>` but other clients like mobile apps and other scripts can do it.

so if making for the browser we may send `token`  via query parameter and then verify that. or we might use  cookies  as it is always send with every request or do first message auth meaning we always accept the connection and then we assume the first message as the auth token and then process it.

if not making for browser using header is fine.


There few things that are similar to web sockets
**polling**
polling is like client making repeated calls to the back-end to get new data which is unlike the webs-ockers. 

there are 2 types of polling.
**short polling:** in this say the fronted make request in fixed interval and the back-end responds immediately  ever data it have. it will have more waste bandwidth

**long polling**: here the when client make the request to the back-end it don't response immediately rather it waits for new data and then send it to the client (meaning it holds on to the request ) or time out hits and the connection is drops. if the connection drops client again send the request.

**server-side-events**
in short fronted initiate the the connection and subscribe to any event and the backed sends the data later when it is available 