# MQTT-Siemens-S7-300
MQTT library block written in SCL for S7-300 with *internal* (PN) or *external* (CP) Ethernet.

This started as a port of [knolleary's MQTT library](https://github.com/knolleary/pubsubclient) for Arduino & ESP8266.
The implementation is following the MQtt v3.1.1 protocol documentation (http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)

*Main functionallity complete, still work in Progress!*

# Purpose and Possibilities
MQTT is a popular communications protocol in the IoT space. Its demand on most types of networks and CPUs make it a good option for M2M applications. MQTT devices can easily be utilized to send a message anywhere in the world.

##### What does this mean for PLCs & Industrial Automation?
MQTT will enable a PLC to connect to the cloud without using proprietary hardware or protocols. PLC programmers can use it to build customized programs that send info to a web server, log plant data, and communicate with any other MQTT client device.
Developers can use it to build customized dashboards (physical, website, or mobile device), providing valuable data for analytics and business. MQTT enabled IO could serve as an inexpensive alternative for the PLC. (although not recommended for mission critical IO and for safety reasons)

### Current Test Scenario:
At this time, this is tested on
- CPU315-2 PN/DP (315-2EH14-0AB0) with internal Ethernet
- CPU314C-2 PN/DP (314-6EH04-0AB0) with CP343-1 (343-1EX30-0XE0)and .
The PLC is connected to a Mosquitto MQtt broker.
All main functionallity has been testet: connect, disconnect, subscribe, unsubscribe, ping, publish.
I am locally connecting to a Mosquitto broker.

### Currently Limitations:

- not all MQtt policies described in the MQtt v3.1.1 standard are exactly standard conform implemented
  Especially the code must be reviewed wether it conforms to all the yellowish lines in the MQtt documentation
- subscribe and unsubscribe only for one topic at a time (you can subscribe multiple times if you need several topics)

### Todo:
- state machine for tcp and mqtt should be harmonized
- MQtt policies review for the code
- implementation of internal and external ethernet usage

# Requirements
It should work with most S7-300 CPU's.
This code is written in Step7 SCL v5.3 SP1. It probably needs modification for it to compile in TIA.

# How to Use
The following needs to be setup in you project in Simatic Manager:

1. You must call the MQTT function block in your OB1 program loop.
2. You need to add the following FBs from the Standard Library:

   For internal Ethernet:
   - FB63  TSEND
   - FB64  TRCV
   - FB65  TCON
   - FB66  TDISCON
   
   For external Ethernet (CP):
   - FC5   AG_SEND
   - FC6   AG_RECV
   - FC10  AG_CTRL
   
   Additional Objects needed:
   - FC21  LEN
   - SFB4  TON
   - SFC6  RD_SINFO
   - SDV20 BLKMOV
   - SFC58 WR_REC
   - SFC59 RD_REC


# Example
Included is an example application function block (FB70) that is typically called from OB1.
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
