# MQTT-Siemens-S7-300
MQTT library block written in SCL for S7-300 with CP343-1

This started as a port of [knolleary's MQTT library](https://github.com/knolleary/pubsubclient) for Arduino & ESP8266.

*Work in Progress!*

# Purpose and Possibilities
MQTT is a popular communications protocol in the IoT space. Its demand on most types of networks and CPUs make it a good option for M2M applications. MQTT devices can easily be utilized to send a message anywhere in the world.

##### What does this mean for PLCs & Industrial Automation?
MQTT will enable a PLC to connect to the cloud without using proprietary hardware or protocols. PLC programmers can use it to build customized programs that send info to a web server, log plant data, and communicate with any other MQTT client device.
Developers can use it to build customized dashboards (physical, website, or mobile device), providing valuable data for analytics and business. MQTT enabled IO could serve as an inexpensive alternative for the PLC. (although not recommended for mission critical IO and for safety reasons)

### Current State:
At this time, this is working for CPU313C-2DP with CP343-1 Lean Ethernet module.
I am locally connecting to a HiveMQ broker and controlling PLC outputs using published messages.
I can also publish an array of bytes from the PLC.
##### Currently Unsupported:

- QoS 1 & 2
- MQTT Username & Password
- Will
- Unsubscribe
- ...


# Requirements
It should work with most S7-300 CPU's.
You must have a CP343-1 module with v2.1 or higher.
This code is written in Step7 SCL v5.3 SP1. It probably needs modification for it to compile in TIA.

# How to Use
The following needs to be setup in you project in Simatic Manager:

1. Setup your CP unit in HW Config with a client IP.
2. The tcp connection to the broker must be set up in NetPro. (IP Address & Port: 1883)
3. You must call the MQTT function block in your OB1 program loop.
4. In Symbols, assign the following symbolic block names with appropriate block numbers assigned to them, mine is:

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
Porting C++ to Siemens SCL has taught many differences between systems. Differences include:

- The inability to make blocking calls in SCL.
- No dynamically allocated memory.

One reason for this is a PLC is a real-time system. A consequence is that the entire program code, from the first to last line, *must* complete in a certain amount of time. (Typically <10ms)
The workaround I make is using a FB with a state machine that can wait for a procedure to finish before moving on.

After working with SCL for a while it feels that the language is primitive in many ways in comparison with most PC languages.
I realise that some of it is because of program limitation for safety and resource conservation, but general program principles like DRY, OOP, etc. are sometimes difficult to achieve if not impossible.

Another annoying difference is the idea of having symbol entries for symbolic block names. On one side I want to refactor and break behaviour up in more functions like in general programming.
But in Siemens I don't want to create a new symbol entry for each FC that I make. Also, this is probably not encouraged in PLC programming anyway as it increases the maximum size of the call stack in every scan cycle and increases the scan time.
This makes it difficult to make small sized program code that is also performant.

Furthermore, the low-level programming and memory limitation reminds of microcontroller programming.
