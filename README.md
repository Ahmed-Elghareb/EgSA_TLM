# EgSA_TLM

## Introduction

The telemetry Control System is one of the most important systems in the ground control station. It is based on the telemetry subsystem that is responsible for sending commands from the ground segment and receiving and monitoring the satellite’s telemetry, like pressure or temperature, the data of the task that the satellite is responsible for like the Imaging command, and satellite states in the space. By the end of this project, we managed to implement a software program that is successfully functioning and integrated with another group that represents the space side of the process which receives commands and sends back telemetry and data. 
<img src="https://user-images.githubusercontent.com/61229902/168183671-74229098-4e03-425b-be46-478872e5b632.png" width="400" height="300" />


The system contains two main subsystems:
1.	TLM server: Which receive the commands to the satellite
2.	TLM Client: Which shows the data from the satellite





## Overview 
The system flow is as follows:
<p align="center">
<img src="https://user-images.githubusercontent.com/61229902/168145900-86af7846-a8b4-4466-a8f6-301d023a9ec5.jpeg" width="600" height="200" />
</p>


## Work Requirements 
<div class="flex-container">
  <div >
    1. Python as front-end and back-end programming language. </br>
    2. SQL to design and program database.
  </div>
  
  <div >
  <img src="https://user-images.githubusercontent.com/61229902/168183668-d19e8b93-944f-4cd0-ac09-37e5a0eef853.png" width="100" height="220" />
  </div>
  
</div>


## Tasks
First, choose a place to photograph from the payload, the satellite control search for a target to photograph, if there is one in the database then it sends the photograph order to the satellite. You can also send other commands to the satellite (get_telemetry, open, close,..etc). Satellite control sends the command to DRTF, DRTF sends the commands to the satellite. Then when the satellite responds, we show the received data whether its image or sensor readings, and show them on the telemetry screen.
The orbit subsystem is responsible for detecting the satellite place at instance time, it's our initial data as we use the satellite place to determine when to image.
We will show the front end of each part and explain how it works.

a)	Payload
As our mission is imaging a specific place, it all starts with a payload subsystem. In this section of our ground station system, the authorized user selects a certain place and sends the order to the image. The payload consists of 2 parts, the first is the graphical user interface (GUI) and the second is the backend connected to a database which consists of two tables.
The first table role comes when a specific place is selected. This table contains all the available places with their coordinates (X-axis and Y-axis) so when a place is selected from the list, we can fetch its coordinates from this premade data. This table is called main and contains four columns (id, name, x_axis, y_axis).
After selecting a place and getting its coordinates, the role of the second table shows up. The targets table saves the selected record with its coordinates, and it will contain one record by max because the table is cleared first before adding any new data.
Now when the satellite control section starts it will look at the targets table, if there is one record it will take a photo of this place, else no photos will be taken.
As a conclusion of the payload objective, it prepares the exact location of the desired place to take a photo of and sends that to satellite control.
<p align="center">
<img src="https://user-images.githubusercontent.com/61229902/168183656-f54b184e-e3df-40a6-a731-77f334a0b1c2.png" width="600" height="200" />
</p>

b)	Satellite Control
Satellite control is a system that is responsible for monitoring and encoding the plan of commands in the ground station. Each command is encoded in a packet format called Space Packet according to the European standard CCSDS as we see in Figure(1). The space packet for each command is consists of a primary header and packet data field. The packet data field consists of a secondary header and user data field, which is consisted of, the opcode (Operation Code) for the command and the stream of data needed to be sent to the satellite. In our implementation, we did not use the secondary header. As a result, our Space Packet consists of the Primary header, opcode, and the user data field. 

The primary header length is 6 bytes (octets) as shown in Figure(2). The first 2 bytes are for packet version, packet type, secondary header flag, and the application process identifier (APID). The APID consists of 11 bits, the first 2 bits refer to the origin of the packet, from the satellite or the ground station. The second 2 bits refers to the number of the satellite, then the next 7 bits refers to the number of the subsystem. Then the next 2 bytes in the primary header show the packet sequence which consists of the sequence flags and the packet number. The last 2 bytes in the primary header refer to the length of the Packet data field. 

As we see in Figure(3), the GUI for the satellite control. As we see, the list of the commands is found at the top of the screen, we could choose any commands and then click on the “Choose commands” button. The chosen commands will appear in the small box at the bottom of the screen. When choosing the getTLM command we need to specify the targeted subsystem to get the information from, as a result, there is an arrow point towards a drop-down list that allows you to choose the subsystem you need. After finishing the plan of commands, you choose which satellite will receive the packets, then click the “Send Commands” button, it encodes all the chosen commands in the format of the space packet and sends them to the DRTF. The DRTF then sends the packets to the chosen satellite.
<p align="center">
<img src="https://user-images.githubusercontent.com/61229902/168183640-6f8b52a0-a005-4ade-9ac5-35508c0b94d4.png" width="600" height="200" />
</p>


c)	DRTF Subsystem
1.	Send
The left half of the DRTF window which is responsible for viewing the commands coming from the satellite control which they are encoded in it then sent to the DRTF to collect the commands and the encoded binary text file to send it using socket programming over to the raspberry pi board in which is either satellite one or satellite two based on the selection in the satellite control window, the commands are presented in the order to be sent in and executed.
There is also a ‘Manual Update’ button which in case of a failure in the reading of the encoded file and the file of the subsystems then by clicking on it the process of fetching the data from the files will repeat then send the commands and get the acknowledgments from the satellite.
2.	Receive 
The right half of the DRTF window is responsible for viewing the incoming commands or packet information coming from the satellite, this window doesn’t show all the information in the packet but can show the type of command received from the satellite that will be executed in the ground station.
This part of the window has its socket opened separate from the send part of the subsystem and this separation is to make the sending and receiving parts not interfere with each other and to minimize the connection errors that might happen due to socket connections with the satellite.
It will also show information about which system the packet will be redirected to (Telemetry, Satellite Control).
<p align="center">
<img src="https://user-images.githubusercontent.com/61229902/168183510-2ac11cc2-75cc-4808-b85a-708a6e18f2e1.png" width="600" height="200" />
</p>


d)	Telemetry

In order to keep track of the going events at the space segments, the ground station holds the mission of monitoring all the readings of the sensors installed on the satellites to detect any abnormalities occurring and analyze the situations the satellites underwent in order to come up with the best plan to execute and solve the issues if there are any. The telemetry subsystem takes the responsibility of receiving and monitoring the status of the two satellites launched into space whenever a communication session is held. In addition, the subsystem receives and views the payload data, like captured photos if there are any. In general, telemetry tries to keep track of space segment subsystems like ADCS, Power, OBC, Communication, and Payload. In the Figure below we show how we designed the telemetry subsystem to function efficiently in real-time communication with the DRTF receiving the TLM data in a communication session with the satellite. In addition to the telemetry data received for two satellites, the screen shows and increments the number of packets received for a specific subsystem for each satellite. All of which serve the purpose of monitoring and debugging the status of both the space segments. If any issues exist, the station is responsible for commanding the satellites to execute the commands which address the issue occurring. 
<p align="center">
<img src="https://user-images.githubusercontent.com/61229902/168183514-a3c8294c-181c-4cfe-aa8f-1362b29238bf.png" width="600" height="200" />
</p>

