
Every means of input/out will need three methods:
	- Directly on the SLC
	- Via a message queue, existing or new
	- Via an as yet developed API

Example: today products can be sent via the message queues, entered on the SLC screen, and tomorrow an API request.

# Apache MQ

### Queues

![[Pasted image 20260310163826.png]]

example: https://10.10.200.148:8162/
ctsNN = NN is the number of the scale.  Written to by the Inven-Host-Message
The scales consume, based on scale ID

Scale publishes to Inven queue, and consume by the Inven-Host-Message program
![[Pasted image 20260310155248.png]]
# External API Calls
 - [ ] How is user data synchronized with S9 when available.
 - [ ] How are serials synchronized, if this is even done?  Fire-forget?
 - [ ] Any other reporting or data I'm missing?


 - [ ] What is the 'Goals.sentToWPL' field indicate?
	 - DF:  This indicates a message was sent and acknowledged to a que setup for WPL communications.


On Login - call to Login Service API.
