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
- CPU314C-2 PN/DP (314-6EH04-0AB0) with CP343-1 (343-1EX30-0XE0) external Ethernet

The PLC is connected to a Mosquitto MQtt broker.
All main functionallity has been testet: connect, disconnect, subscribe, unsubscribe, ping, publish.
I am locally connecting to a Mosquitto broker.

### Currently Limitations:

- not all MQtt policies described in the MQtt v3.1.1 standard are exactly standard conform implemented
  Especially the code must be reviewed wether it conforms to all the yellowish lines in the MQtt documentation
- subscribe and unsubscribe only for one topic at a time (you can subscribe multiple times if you need several topics)
- the Siemens PLC Ethernet adapters can send/receive *8192 bytes max.* per transmission. Please refer to the Simatic Menager help for the corresponding FBs/FCs


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

## Network Configuration
The MQTT FB can use the internal Ethernet adapter of a CPU (PN) or an external Ethernet adapter (CP).

Remarks for internal Ethernet (PN) adapter configuration:
- The IP-Address of the internal adapter has to be configured within the hardware configuration tool (HW Config), click on the PN-IO object
- The remote IP and Port parameters are configured via an UDT structure (UDT1 or whatever the UDTx number is). Create a DB out of it and modify the network parameters according to your needs:
 - connection_type: must be B#16#11
 - rem_staddr:  The IP address in bytes (6 byte array). The last 4 bytes must be B#16#0
 - rem_tsap_id: The remote Port in bytes (16 byte array). For an MQtt broker this is usually 1883 (B#16#7, B#16#D0). The last 14 bytes must be B#16#0.
 - you must pass the network parameters DB of type (UDT1 or whatever the UDTx number is) to the MQTT Functionblock parameter net_config.
 - you must set the MQTT Functionblock parameter PNorCP to 0 (PN=0, CP=1)
 - you must set the MQTT Functionblock parameter connectionID to a desired value, f.e. 1
 - you must set the MQTT Functionblock parameter is not needed for the PN configuration
Example for a MQTT FB call in OB1 configured for internal Ethernet (PN) usage:
*MQTT.DB71(net_config := DB72_NET_CONFIG, PNorCP := 0, connectionID := 1);*

 Remarks for external Ethernet (CP) adapter configuration:
 - The IP-Address of the internal adapter has to be configured within the hardware configuration tool (HW Config), click on the PN-IO object
 - The connection must be configured in Simatic Manager "Connections"
 - you must pass the network parameters DB of type (UDT1 or whatever the UDTx number is) to the MQTT Functionblock parameter net_config, however no configuration is needed because it is ignored.
 - you must set the MQTT Functionblock parameter PNorCP to 1 (PN=0, CP=1)
 - you must set the MQTT Functionblock parameter connectionID to the connection ID configured in Simatic Manager "Connections"
 - you must set the MQTT Functionblock parameter cpLADDR to the address of the CP module.
 Example for a MQTT FB call in OB1 configured for external Ethernet (CP)usage:
*MQTT.DB71(net_config := DB72_NET_CONFIG, PNorCP := 1, connectionID := 1, cpLADDR := W#16#100);*

## Setup memory footprint

### Setting up buffers
To reduce the memory usage of the MQtt DBs you can set the receive and transmit buffer sizes.

- Set tcpRecBuf array size in mqttData DB and TCP_RECVBUFFERSIZE in mqttGlobals to the same value (f.e. 8192) of your choice.
- Set TCP_MAXRECVSIZE in mqttGlobals DB to a value equal or smaller than TCP_RECVBUFFERSIZE
- Set tcpSendBuf array size in mqttData DB to a value of your choice
- Set buffer array size in mqttData DB to a value of your choice, but not smaller than the largest value of TCP_MAXRECVSIZE or tcpSendBuf, whoever is larger

Keep in mind: data from tcpRecBuf is transfered to buffer and also data from buffer is tranfered to tcpSendBuf within the Code. So match the sizes appropriately.

### Kickout CP or PN references if not used

All calls to Ethernet functions within the MQTT Functionblock code are implemented as a PN and a CP call. There is a If-Then-Else clause refering to PNorCP Input variable.
Remove the If-Then-Else clause and keep the Ethernet function call you need.

Don´t break the code :-)

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
