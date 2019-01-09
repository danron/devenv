Netmark - a network load generator and analysis tool
====================================================

Quick overview
--------------

The server is started on one machine and the client on the other. The server will open three listening ports, one control port, one data port for TCP and one data port for UDP. The client will first open a connection to the server and send the traffic profile to be executed. The client then closes this connection before executing the traffic profile. During execution the client opens all data connections to the two data ports on the server. After execution a report is exchanged between the client and server.


Handshake between client and server
-----------------------------------

When the client starts up it first reads its traffic profile and figure out how many TCP and UDP connections that will be needed during the execution of the trafic profile. It then creates a socket for each one and binds it to a **random** local port. This will be the source port of the outgoing connection to the server. The traffic profile is then updated with these source ports so that the server will know which connection is which (it has to know this because sometimes the server should send data before the client on a new data connection).

The profile should contain some addition information like a client identifier so that the server can log more than just the source ip and port of the client.

The profile is then serialized and sent to the server which replies with an ACK message. The ACK message contains a unique execution identifier which should be used by the client when it reconnects after the traffic profile is completed. This will let the server know what reports to send to the client.

The client starts execution (opening data connections) as soon as it receives the ACK.

When the execution starts the client will see if it's supposed to send any data first, and if it should it will pick the right socket and connect it to the server and send the data. The same socket can also be supposed to receive data so the server will need to know which one of the incoming connections to send data on by looking at the source port of the connection. This is important when there are multiple connections in the first step (see the example in this document).

**IMPORTANT**  
Note is that when the profile has overlapping actions of the same type (TCP/UDP) multiple connections are needed. This can be derived by how the sequence of actions are asynchroneously branched.


Example of usage
----------------

### Start the server

    netmark -s 4000

Netmark will start to listen for incoming control connections on TCP port 4000. The following port will be used for data for both TCP and UDP. A specific UDP and TCP data port can be specified with switches -t and -u.

### Start the client with a simple traffic profile

    netmark serverhost.se:4000 -- TCP/300/0 (TCP/500/400 \
    (UDP/50/0/-/- UDP/0/120/-/-)) TCP/0/800

Note that the backslash is used to divide up the one line expression into multiple lines.

Netmark connects to the server control port and sends it's traffic profile. It then closes the control connection and executes the specified traffic profile. After the profile is completed (or something timed out) the control connection is again opened and reports containing metrics for each of the actions are exchanged.

Here is a description including a timeline of a traffic profile:

    The traffic profile syntax:
    TCP/300/0 (TCP/500/400 (UDP/50/0/-/- UDP/0/120/-/-)) TCP/0/800
       (1)         (2)         (3)            (4)           (5)

    Description of each object in the profile:
    1) Send 300 bytes over TCP
    2) Send 500 and receive 400 bytes over TCP
    3) Send 50 bytes of UDP with default packet size and packet rate
    4) Receive 120 bytes of UDP with default packet size and packet rate
    5) Receive 800 bytes over TCP

    Timeline with each row as a separate connection:
    -1->    -5->
    -2->
    -3->-4->

The traffic profile actions
---------------------------

A traffic profile is made up of actions that are specified in a sequence. The sequence can be split up in asynchronously executing branches by grouping each asynchronous branch in parentheses.

### TCP

TCP/outbytes/inbytes

### UDP

UDP/outbytes/inbytes

UDP packets must contain a packet number in their payload for the packet ordering checks to work. So there is a lower limit on packet size.


### NOP

NOP/milliseconds

Do nothing and just idle for the specified number of milliseconds.

General parameters
------------------

General parameters that is valid for all actions are configurable as a parameter in the action syntax. Add the following after the normal action syntax:

    #param1=value1;param2=value2

    Example:
    TCP/500/0#Bps=5000;randomness=45

### Randomness of payload

The name of the parameters for this function will depend on the implementation of the random algorithm.

#### UDP Packet size

    OutPktSize (Size in bytes of outgoing packets)
    InPktSize (Size in bytes of incoming packets)
    PktSize (Size in bytes of all packets)

These values are only valid for the UDP action and defaults to MTU - UDP header size. They can be specified in either or both directions.

### Bandwidth limit

    OutBps (Bytes per second of outgoing traffic)
    InBps (Bytes per second of incoming traffic)
    Bps (Bytes per second of all traffic)

This can be used to limit the bandwidth for both TCP and UDP connections in either or both directions.

When using UDP connections the packetrate is calculated as:

    Packetrate = Bytes per second / Packetsize

    Example:

    UDP/500/0#PktSize=10;Bps=50

    Packetsize = 10
    Bps = 50
    Packetrate = 50 / 10 = 5 packets per second


Traffic profile report details
------------------------------

The report is sent both ways and both ends will print or log the complete result. The report contains the following metrics for each of the actions in the profile:

TCP:
* Bytes out
* Bytes in
* Duration
* Result (OK or an error)

UDP:
* Bytes out
* Bytes in
* Duration
* Packet loss
* Packet wrong order
* Packet jitter
* Result (OK or an error)

NOP:
* Result (OK or an error, for example if a connection died during this action)
