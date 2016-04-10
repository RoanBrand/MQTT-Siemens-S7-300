# MQTT-Siemens-S7-300
MQTT library block written in SCL for S7-300 with CP343-1

This started as a port of [knolleary's MQTT library](https://github.com/knolleary/pubsubclient) for Arduino & ESP8266.

*Work in Progress!*

### Current State:
At this time, this is working for CPU313C-2DP with CP343-1 Lean Ethernet module.
I am locally connecting to a HiveMQ broker and controlling PLC outputs using published messages.
I can also publish a hard coded message from the PLC.

# Requirements
It should work with most S7-300 CPU's.
You must have a CP343-1 module with v2.1 or higher.
This code is written in Step7 SCL v5.3 SP1. It probably needs modification for it to compile in TIA.

# How to Use
The following needs to be setup in you project in Simatic Manager:

1. The tcp connection to the broker must be set up in NetPro (IP Address + Port: 1883)
2. You must call the MQTT function block in your OB1 program loop.
2. Your project's symbol list needs appropriate block numbers assigned to them, mine is:

- MQTT   		: FB1
- PacketReader	: FB2
- Globals  		: DB100
- Data			: DB101
- connect 		: FC50
- sendTCP		: FC51
- write 		: FC52
- writeString	: FC53
- available		: FC54
- readByte 		: FC55
- readByteToBuf	: FC56
- publish     	: FC57
- subscribe		: FC58
- connected		: FC59


# Example
Included is an example application function block (FB66) that is typically called from OB1.
Inputs for this block can trigger a MQTT broker connect, publish a message or subscribe to a MQTT channel.




### Notes
Porting C++ to Siemens SCL has taught many differences between systems. One major difference is the inability to make blocking calls in SCL.
The reason for this is a PLC is a real-time system. What this means is the entire program from the first to last line *must* complete in a certain amount of time. (typically <10ms)
The workaround I make is using a FB with a state machine that can wait for a procedure to finish before moving on.

After working with SCL for a while it feels that the language is primitive in many ways in comparison with most PC languages.
I realise that some of it is because of program limitation for safety and resource conservation, but general program principles like DRY, OOP, etc. are sometimes difficult to achieve if not impossible.

Another annoying difference is the idea of having symbol entries for symbolic block names. On one side I want to refactor and break behaviour up in more functions like in general programming.
But in Siemens I don't want to create a new symbol entry for each FC that I make. Also, this is probably not encouraged in PLC programming anyway as it increases the maximum size of the call stack in every scan cycle and increases the scan time.
This makes it difficult to make small sized program code that is also performant.

Furthermore, the low-level programming and memory limitation reminds of microcontroller programming.