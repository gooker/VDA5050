![logo](./assets/logo.png)
# Interface for the communication between automated guided vehicles (AGV) and a RCS 

## VDA 5050

## Version 2.0.0

![control system and automated guided vehicles](./assets/csagv.png) 



### Brief information

Definition of a communication interface for driverless transport systems (DTS).
This recommendation describes the communication interface for exchanging task and status data between a central RCS and automated guided vehicles (AGV) for intralogistics processes.  



### Disclaimer 

The following explanations serve as an indication for the execution of an interface for communication between automated guided vehicles (AGV) and RCS and one that is freely applicable to everyone and is non-binding.
Those who apply them must ensure that they are applied properly in the specific case.

They shall take into account the state of the art prevailing at the time of each issue.
By applying the proposals, no one is evasive of responsibility for their own actions. 
The statements do not claim to be exhaustive or to the exact interpretation of the existing legislation.
They may not replace the study of relevant policies, laws and regulations. 
Furthermore, the special features of the respective products as well as their different possible applications must be taken into account.
Everyone acts at his own risk in this regard.
Liability of the VDA and those involved in the development or application of the proposals is excluded.

If you encounter any inaccuracies in the application of the proposals or the possibility of an incorrect interpretation, please inform the VDA immediately so that any defects can be rectified.

**Publisher**
Verband der Automobilindustrie e.v. (VDA)
Behrenstrasse 35, 10117 Berlin 
www.vda.de

**Copyright**
Association of the Automotive Industry (VDA)
Reproduction and any other form of reproduction is only permitted with specification of the source.

Version 2.0



## Table of contents

[1 Foreword](#Foreword)<br>
[2 Objective of the document](#Ootd)<br>
[3 Scope](#Scope)<br>
[3.1 Other applicable documents](#Oad)<br>
[4 Requirements and protocol definition](#Rapd)<br>
[5 Process and content of communication](#Pacoc)<br>
[6 Protocol specification](#Ps)<br>
[6.1 Symbols of the tables and meaning of formatting](#Sottamof)<br>
[6.1.1 可选 fields](#Of)<br>
[6.1.2 Permitted characters and field lengths](#Pcafl)<br>
[6.1.3 Notation of enumerations](#Noe) <br>
[6.1.4 JSON Datatypes](#JD)<br>
[6.2 MQTT connection handling, security and QoS](#MchsaQ)<br>
[6.3 MQTT-Topic Levels](#MTL)<br>
[6.4 Protocol Header](#PH)<br>
[6.5 Subtopics for communication](#Sfc)<br>
[6.6 Topic: "task" (from RCS to AGV)](#TOfmctA)<br>
[6.6.1 Concept and Logic](#CaL)<br>
[6.6.2 tasks and task updates](#Oaou)<br>
[6.6.3 task Cancellation (by RCS)](#OCbMC)<br>
[6.6.3.1 Receiving a new task after cancellation](#Ranoac)<br>
[6.6.3.2 Receiving a cancelOrder action when AGV has no task](#RacawAhno)<br>
[6.6.4 task rejection](#Or)<br>
[6.6.4.1 Vehicle gets a malformed new task](#Vgamno)<br>
[6.6.4.2 Vehicle receives an task with actions it cannot perform (例如 lifting height higher than maximum lifting height, or lifting actions although no stroke is installed), or with fields that it cannot use (例如 Trajectory)](#Vraowaicpeglhhtmlholaansii)<br>
[6.6.4.3 Vehicle gets a new task with the same orderId but a lower orderUpdateId than the current orderUpdateId](#Vehiclegets)<br>
[6.6.5 Maps](#Maps)<br>
[6.7 Implementation of the task message](#Iotom)<br>
[6.8 actions](#actions)<br>
[6.8.1 Predefined action definitions, their parameters, effects and scope](#Padtpeas)<br>
[6.8.2 Predefined action definitions, their parameters, effects and scope](#Padtpeas1)<br>
[6.9 Topic: "instantactions" (from RCS to AGV)](#Tifmc)<br>
[6.10 Topic: "state" (from AGV to RCS)](#TSfAtmc)<br>
[6.10.1 Concept and Logic](#CaLe)<br>
[6.10.2 Traversal of points and entering/leaving segments, triggering of actions](#Tonaeletoa)<br>
[6.10.3 Base request](#Br)<br>
[6.10.4 Information](#Information)<br>
[6.10.5 Errors](#Errors)<br>
[6.10.6 Implementation](#Implementation)<br>
[6.11 actionStates](#actionStates)<br>
[6.12 action Blocking Types and sequence](#ABTas)<br>
[6.13 Topic "visualization"](#TV)<br>
[6.14 Topic "connection"](#Tc)<br>
[6.15 Topic "factsheet"](#Tf)<br>
[7 Best practice](#Bp)<br>
[7.1 Error reference](#Er)<br>
[7.2 Format of parameters](#Fop)<br>
[8 Glossary](#Glossary)<br>
[8.1 Definition](#Definition)<br>



# <a name="Foreword"></a> 1 Foreword 


The interface was established in cooperation between the Verband der Automobilindustrie e. V. (German abbreviation VDA) and Verband Deutscher Maschinen-und Anlagenbau e. V. (German abbreviation VDMA). 
The aim of both parties is to create an universally applicable interface. 
Proposals for changes to the interface shall be submitted to the VDA, are evaluated jointly with the VDMA and adopted into a new version status in the event of a positive decision.
The contribution to this document via GitHub is greatly appreciated.
The Repository can be found at the following link: http://github.com/vda5050/vda5050.



# <a name="Ootd"></a> 2 Objective of the document 

The objective of the recommendation is to simplify the connection of new vehicles to an existing RCS and thus to integrate into an existing automated guided vehicles (AGV) system when used in the automotive industry and to enable parallel operation with AGV from different manufacturers and conventional systems (inventory systems) in the same working environment.

Uniform interface between RCS and AGV shall be defined. 
In detail, this should be achieved by the following points: 

- Description of a standard for communication between AGV and RCS and thus a basis for the integration of transport systems into a continuous process automation using co-operating transport vehicles.
- Increase in flexibility through, among other things, increased vehicle autonomy, process modules and interface, and preferably the separation of a rigid sequence of event-controlled command chains. 
- Reduction of implementation time due to high "Plug & Play" capability, as required information (例如 task information) are provided by central services and are generally valid. Vehicles should be able to be put into operation independently of the manufacturer with the same implementation effort taking into account the requirements of occupational safety.
- Complexity reduction and increase of the "Plug & Play" capability of the systems through the use of uniform, overarching coordination with the corresponding logic for all transport vehicles, vehicle models and manufacturers.
- Increase in manufacturers independence using common interfaces between vehicle control and coordination level.
- Integration of proprietary DTS inventory systems by implementing vertical communication between the proprietary RCS and the superordinate RCS (cf.  Figure 1).

![Figure 1 Integration of DTS inventory systems](./assets/Figure1.png)
>Figure 1 Integration of DTS inventory systems

In task to implement the above-mentioned objectives, this document describes an interface for the communication of task and status information between AGV and RCS.

Other interfaces required for operation between AGV and RCS (例如, for exchanging map information, taking special skills freely into account with regard to path planning, 等等.) or for communicating with other system components (例如, external peripherals, fire protection gates, 等等.) are not initially included in this document. 



# <a name="Scope"></a> 3 Scope

This recommendation contains definitions and best practice regarding communication between automated guided vehicles (AGVs) and RCS.
The goal is to allow AGV with different characteristics (例如, underrun tractor or fork lift AGV) to communicate with RCS in uniform language. 
This creates the basis for operating any combination of AGV in a RCS.
RCS provides tasks and coordinates the AGV traffic.

The interface is based on the requirements from production and plant logistics in the automotive industry.
According to the formulated requirements, the requirements of intralogistics cover the requirements of the logistics department, i.e., the logistical processes from goods receiving to production supply to goods out, through control free navigating vehicles and guided vehicles.

In contrast to automated vehicles, autonomous vehicles solve problems that occur on the basis of the corresponding sensor system and algorithms independently and can react accordingly to changes in a dynamic environment or be adapted to them shortly afterwards. 
Autonomous properties such as the independent bypassing of obstacles can be fulfilled by free navigating vehicles as well as guided vehicles. 
However, as soon as the path planning is carried out on the vehicle itself, this document describes free navigating vehicles (see glossary).
Autonomous systems are not completely decentralized (swarm intelligence) and have defined behavior through predefined rules.

For the purpose of a sustainable solution, an interface is described below which can be expanded in its structure.
This should enable a complete coverage of RCS for vehicles that are guided. 
Vehicles that are free navigating can be integrated into the structure; a detailed specification required for this is not part of this recommendation.

For the integration of proprietary stock systems, individual definitions of the interface may be required, which are not considered as part of this recommendation.



## <a name="Oad"></a> 3.1 Other applicable documents

Document (Dokument) | Description 
----------------------------------| ----------------
VDI Guideline 2510 | Driverless transport systems (DTS)
VDI Guideline 4451 Sheet 7 | Compatibility of driverless transport systems (DTS) - DTS RCS 
DIN EN ISO 3691-4 | Industrial Trucks Safety Requirements and Verification-Part 4: Driverless trucks and their systems 



# <a name="Rapd"></a> 4 Requirements and protocol definition 

The communication interface is designed to support the following requirements: 

- Control of min. 1000 vehicles
- Enabling the integration of vehicles with different degrees of autonomy
- Enable decision, 例如, with regard to the selection of routes or the behavior at intersections 

Vehicles should transfer their status at a regular interval or when their status changes. 

Communication is done over wireless networks, taking into account the effects of connection failures and loss of messages. 

The message log is Message Queuing Telemetry Transport (MQTT), which is to be used in conjunction with a JSON structure.
MQTT 3.1.1 was tested during the development of this protocol and is the minimum required version for compatibility.
MQTT allows the distribution of messages to subchannels, which are called "topics". 
Participants in the MQTT network subscribe to these topics and receive information that concerns or interests them.

The JSON structure allows for a future extension of the protocol with additional parameters.
The parameters are described in English to ensure that the protocol is readable, comprehensible and applicable outside the German-speaking area.



# <a name="Pacoc"></a> 5 Process and content of communication

As shown in the information flow to the operation of AGV, there are at least the following participants (see Figure 2): 

- the operator provides basic information
- RCS organizes and manages the operation 
- the AGV carries out the tasks

Figure 2 describes the communication content during the operational phase.
During implementation or modification, the AGV and RCS are manually configured. 

![Figure 2 Structure of the Information Flow](./assets/Figure2.png)
>Figure 2 Structure of the Information Flow

During the implementation phase, the driverless transport systems (DTS) consisting of RCS and AGV is set up.
The necessary framework conditions are defined by the operator and the required information is either entered manually by him or stored in RCS by importing from other systems. 
Essentially, this concerns the following content:

- Definition of  routes: Using CAD import,  routes can be taken over in RCS.
Alternatively, routes can also be implemented manually in RCS by the operator.
Routes can be one-way streets,  restricted for certain vehicle groups (based on the size ratios), 等等.
- Route network configuration:
Within the routes, stations for loading and unloading, battery charging stations, peripheral environments (gates, elevators, barriers), waiting positions, buffer stations, 等等. are defined. 
- Vehicle configuration: The physical properties of an AGV (size, available load carrier mounts, 等等.) are stored by the operator.
The AGV must communicate this information via the subtopic `factsheet` in a specific way that is defined in the [AGV Factsheet section](#factsheet) of this document.

The configuration of routes and the route network described above is not part of this document.
It forms the basis for enabling task control and driving course assignment by RCS based on this information and the transport requirements to be completed. 
The resulting tasks for an AGV are then transferred to the vehicle via an MQTT message broker.
This then continuously reports its status to RCS in parallel with the execution of the job. 
This is also done using the MQTT message broker.

Functions of RCS are: 

- Assignment of tasks to the AGV
- Route calculation and guidance of the AGV (taking into account the limitations of the individual physical properties of each AGV, 例如, size, maneuverability, 等等.)
- Detection and resolution of blockages ("deadlocks")
- Energy management: Charging tasks can interrupt transfer tasks
- Traffic control: Buffer routes and waiting positions
- (temporary) changes in the environment, such as freeing certain areas or changing the maximum speed
- Communication with peripheral systems such as doors, gates, elevators, 等等. 
- Detection and resolution of communication errors 

Functions of the AGV are: 

- Localization
- Navigation along associated routes (guided or autonomous) 
- Continuous transmission of vehicle status 

In addition, the integrator must take into account the following when configuring the overall system (incomplete list): 

- Map configuration: The coordinate systems of RCS and the AGV must be matched.
- Pivot point: The use of different points of the AGV or points of charge as a pivot point leads to different envelopes of the vehicle. The reference point may vary depending on the situation, 例如, it may be different for an AGV carrying a load and for an AGV that does not carry a load.



# <a name="Ps"></a> 6 Protocol specification 

The following section describes the details of the communication protocol.
The protocol specifies the communication between RCS and the AGV.
Communication between the AGV and peripheral equipment, 例如, between the AGV and a gate, is excluded.

The different messages are presented in tables describing the contents of the fields of the JSON that is sent as an task, state, 等等.

In addition, JSON schemas are available for validation in the public Git repository (https://github.com/VDA5050/VDA5050/json_schemas).
The JSON schemas are updated with every release of the VDA5050.



## <a name="Sottamof"></a> 6.1 Symbols of the tables and meaning of formatting

The table contains the name of the identifier, its unit, its data type, and a description, if any.

Identification | Description [ENG]
---|----
standard | Variable is an elementary data type 
**bold** | Variable is a non-elementary data type (例如, JSON-object or array) and defined separately
*italic* | Variable is 可选 
[Square brackets] | Variable (here arrayName) is an array of the data type included in the square brackets (here the data type is squareBrackets)

All keywords are case sensitive.
All field names are in camelCase. 
All enumerations are in UPPERCASE.



### <a name="Of"></a> 6.1.1 可选 fields

If a variable is marked as 可选, it means that it is 可选 for the sender because the variable might not be applicable in certain cases (例如, when RCS sends an task to an AGV, some AGV plan their trajectory themselves and the field trajectory within the segment object of the task can be omitted). 

If the AGV receives a message that contains a field which is marked as 可选 in this protocol, the AGV is expected to act accordingly and cannot ignore the field. 
If the AGV cannot process the message accordingly then the expected behavior is to communicate this within an error message and to reject the task.

RCS shall only send 可选 information that the AGV supports.

Example: Trajectories are 可选. 
If an AGV cannot process trajectories, RCS shall not send a trajectory to the vehicle.

The AGV must communicate which 可选 parameters it needs via an AGV factsheet message.


### <a name="Pcafl"></a> 6.1.2 Permitted characters and field lengths

All communication is encoded in UTF-8 to enable international adaption of descriptions.
The recommendation is that IDs should only use the following characters:

A-Z a-z 0-9 _ - . :

A maximum message length is not defined. 
If an AGV memory is insufficient to process an incoming task, it is to reject the task.
The matching of maximum field lengths, string lengths or value ranges is up to the integrator.
For ease of integration, AGV vendors must supply an AGV factsheet that is detailed in [section 7 - AGV Factsheet](#factsheet).



### <a name="Noe"></a> 6.1.3 Notation of enumerations 

Enumerations must be written in uppercase. 
This includes keywords such as the states of the actions (WAITING, FINISHED, 等等...) or values of the "direction" field (LEFT, RIGHT, 443MHZ, 等等...).



### <a name="JD"></a> 6.1.4 JSON Datatypes 

Where possible, JSON data types must be used.
A Boolean value is thus encoded by "true / false", NOT with an enumeration (TRUE, FALSE) or magic numbers.



## <a name="MchsaQ"></a> 6.2 MQTT connection handling, security and QoS

The MQTT protocol provides the option of setting a last will message for a client.
If the client disconnects unexpectedly for any reason, the last will is distributed by the broker to other subscribed clients.
The use of this feature is described in section 6.14.

If the AGV disconnects from the broker, it keeps all the task information and fulfills the task up to the last released point. 

Protocol-Security needs to be taken in account by broker configuration.

To reduce the communication overhead, the MQTT QoS level 0 (Best Effort) is to be used for the topics `task`, `state`, `factsheet` and `visualization`.
The topic `connection` shall use the QoS level 1 (At Least Once).



## <a name="MTL"></a> 6.3 MQTT-Topic Levels 

The MQTT-Topic structure is not strictly defined due to the mandatory topic structure of cloud providers.
For a cloud-based MQTT-Broker the topic structure has to be adapted individually to match the topics defined in this protocol. 
This means that the topic names defined in the following sections are mandatory.

For a local broker the MQTT topic levels are suggested as followed:

**interfaceName/majorVersion/manufacturer/serialNumber/topic**

Example: uagv/v2/KIT/0001/task



MQTT Topic Level | Data type | Description 
---|-----|-----
interfaceName | string | Name of the used interface 
majorVersion | string | Major version number, preceded by "v"
manufacturer | string | Manufacturer of the AGV (例如, RobotCompany)
serialNumber | string | Unique AGV Serial Number consisting of the following characters: <br>A-Z <br>a-z <br>0-9 <br>_ <br>. <br>: <br>-
topic | string | Topic (例如 task or System State) see Cap. 6.5

Note: Since the `/` character is used to define topic hierarchies, it must not be used in any of the aforementioned fields.
The `$` character is also used in some MQTT brokers for special internal topics, so it should not be used either.

## <a name="PH"></a> 6.4 Protocol Header

Each JSON starts with a header.
In the following sections, the following fields will be referenced as header for readability. 
The header consists of the following individual elements. 
The header is not a JSON object.

Object structure/Identifier | Data type | Description 
---|---|---
headerId | uint32 | header ID of the message.<br> The headerId is defined per topic and incremented by 1 with each sent (but not necessarily received) message. 
timestamp | string | Timestamp (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”)
version | string | Version of the protocol [Major].[Minor].[Patch] (例如 1.3.2)
manufacturer | string | Manufacturer of the AGV 
serialNumber | string | Serial number of the AGV 

### Protocol version

The protocol version uses semantic versioning as versioning schema.

Examples for major version changes: 

- Breaking changes, 例如, new non-可选 fields

Examples for minor version changes: 

- New features like an additional topic for visualization 

Examples for patch version: 

- Higher available precision for a batteryCharge 



## <a name="Sfc"></a> 6.5 Subtopics for communication

The AGV protocol uses the following topics for information exchange between RCS and AGV

Subtopic name | Published by | Subscribed by | Used for | Implementation | Schema 
---|---|---|---|---|---
task | RCS | AGV | Communication of driving tasks from RCS to the AGV | mandatory | task.schema 
instantactions | RCS | AGV | Communication of the actions that are to be executed immediately | mandatory | instantactions.schema
state | AGV | RCS | Communication of the AGV state | mandatory | state.schema
visualization | AGV | Visualization systems | Higher frequency of position topic for visualization purposes only | 可选 | visualization.schema
connection | Broker/AGV | RCS | Indicates when AGV connection is lost, not to be used by RCS for checking the vehicle health, added for an MQTT protocol level check of connection | mandatory | connection.schema 
factsheet | AGV | RCS | Setup of AGV in RCS | mandatory | factsheet.schema


## <a name="TOfmctA"></a> 6.6 Topic: "task"(from RCS to AGV)

The topic "task" is the MQTT topic via which the AGV receives a JSON encapsulated task. 



### <a name="CaL"></a> 6.6.1 Concept and Logic 

The basic structure of an task is a graph of points and segments.
The AGV is expected to traverse the points and segments to fulfill the task.
The full graph of all connected points and segments is held by RCS.

The graph representation in RCS contains restrictions, 例如, which AGV is allowed to traverse which segment.
These restrictions will not be communicated to the AGV.
RCS only includes segments in an AGV task which the concerning AGV is allowed to traverse.

It is to be avoided that RCS has a separate graph representation for each type of AGV.
Whenever possible, one location, 例如, a waiting position in front of fire door, should only have one point for all types of AGV.
However, due to the different sizes and specifications of AGV, it might be necessary to deviate from this standard in certain situations.

![Figure 3 Graph representation in RCS and graph transmitted in tasks](./assets/Figure3.png) 
>Figure 3 Graph representation in RCS and graph transmitted in tasks

The points and segments are passed as two lists in the task message.
The lists task also governs in which sequence the points and segments must be traversed.

For a valid task, at least one point must be present. 
The number of acceptable segments is the number of points minus one, not more or less.

The first point of an task must be trivially reachable for the AGV. 
This means either that the AGV is already standing on the point, or that the AGV is in the points deviation range.

points and segments both have a boolean attribute "released”.
If a point or segment is released, the AGV is expected to traverse it. 
If a point or segment is not released, the AGV must not traverse it.

An segment only can be released, if both the start and end point of the segment are released.

After an unreleased segment, no released points or segments can follow in the sequence. 

The set of released points and segments are called the "base”. 
The set of unreleased points and segments are called the "horizon”.

It is valid to send an task without a horizon.

An task message does not necessarily describe the full transport task. 
For traffic control and to accommodate resource constrained vehicles, the full transport task (which might consist of many points and segments) can be split up into many sub-tasks, which are connected via their orderId and orderUpdateId. 
The process of updating an task is described in the next section.



### <a name="Oaou"></a> 6.6.2 tasks and task update 

For traffic control the task-topic includes only the path to a decision point. 
Before reaching the decision point, RCS will send an updated path with additional path segments.
To communicate to the AGV what it will most likely have to do after reaching the decision point, an task consists of two separate parts: 

- <u>Drive to the decision point "Base":</u> The "Base" is the defined route that the AGV travels. All points and segments of the "Base" route have already been approved by the control panel for the vehicle. 
- <u>Estimated journey from the decision point "Horizon":</u> The "Horizon" is the route that the AGV is likely to drive, if there is no traffic jam. The "Horizon" route has not yet been approved by the control panel. However, the AGV will initially only travel to the last junction of the "Base" route.

Since MQTT is an asynchronous protocol and transmission via wireless networks is not reliable, it is important to note, that the "base" cannot be changed. 
RCS can therefore assume that the "base" is executed by the AGV.
A later section describes a procedure for cancelling an task, but this is also considered unreliable due to the communication restrictions mentioned above.

RCS has the possibility to change the driving commands of the "Horizon" route. 
Before the AGV arrives at the decision point via the "base" route, RCS will send an updated route to the AGV, which includes the other points. 
The procedure for changing the Horizon route is shown in Figure 4.

![Figure 4 Procedure for changing the driving route "Horizon"](./assets/Figure4.png)
>Figure 4 Procedure for changing the driving route "Horizon"

In Figure 4, an initial job is first sent by the control panel at time t = 1.
Figure 5 shows the pseudocode of a possible job.
For the sake of readability, a complete JSON example has been omitted here.

```
{
	orderId: "1234"
	orderUpdateId:0,
	points: [
	 	 6 {released: True},
	 	 4 {released: True},
	 	 7 {released: True},
	 	 2 {released: False},
	 	 8 {released: False}
	],
	segments: [
		e1 {released: True},
		e3 {released: True},
		e8 {released: False},
		e9 {released: False}
	]
}
```
>Figure 5 Pseudocode of an task

At time t = 3, the task is updated by sending an extension of the task (see example in Figure 6). 
Note that the "orderUpdateId" is incremented and that the first point of the job update corresponds to the last shared base point of the previous task message.

This ensures that the AGV can also perform the job update, i.e., that the first point of the job update is reachable by executing the segments already known to the AGV.

```
}
	orderId: 1234,
	orderUpdateId: 1,
	points: [
		7 {released: True},
		2 {released: True},
		8 {released: True},
		9 {released: False}
	],
	segments: [
		e8 {released: True},
		e9 {released: True},
		e10 {released: False}
	]
}
```
>Figure 6 Pseudocode of an task update. Please look out for the change of the "orderUpdateId"

This also aids in the event that an orderUpdate goes missing (because of unreliable wireless network). 
The AGV can always check that the last known base point has the same nodeId (and nodeSequenceId, more on that later) as the first new base point.

Also note that point 7 is the only base point that is sent again.
Since the base cannot be changed, a retransmission of points 6 and 4 is not valid.

It is important, that the contents of the stitching point (point 7 in the example case) are not changed. 
For actions, deviation range, 等等. the AGV must use the instructions provided in the first task (Figure 5, orderUpdateId 0).

![Figure 7 Regular update process - task extension](./assets/Figure7.png)
>Figure 7 Regular update process - task extension

Figure 7 describes how an task should be extended.
It shows the information, that is currently available on the AGV. 
The orderId stays the same and the orderUpdateId is incremented. 

The last point of the previous base is the first base point in the updated task.
With this point the AGV can add the updated task onto the current task (stitching). 
The other points and segments from the previous base are not resent.

RCS has the option to make changes to the horizon by sending entirely different points as the new base.
The horizon can also be deleted.

To allow loops in tasks (like going from point 1 to 2 and then back to 1) a sequenceId is assigned to the point and segment objects. 
This sequenceId runs over the points and segments (first point of an task receives a 0, the first segment then gets the 1, the second point then gets the 2, and so on). 
This allows for easier tracking of the task progress.

Once a sequenceId is assigned, it does not change with task updates (see Figure 7). 
This is necessary to determine on AGV side to which point RCS refers to. 

Figure 8 describes the process of accepting an task or orderUpdate.

![Figure 8 The process of accepting an task or orderUpdate](./assets/Figure8.png)
>Figure 8 The process of accepting an task or orderUpdate



### <a name="OCbMC"></a> 6.6.3 task Cancellation (by RCS)

In the event of an unplanned change in the base points, the task must be canceled by using the instantaction cancelOrder.

After receiving the instantaction cancelOrder, the vehicle stops (based on its capabilities, 例如, right where it is or on the next point).

If there are actions scheduled, these actions must be cancelled and should report "failed” in their actionstate. 
If there are running actions, those actions should be cancelled and also be reported as failed.
If the action cannot be interrupted, the actionstate of that action should reflect that by reporting "running” while it is running, and after that the respective state ("finished”, if  successful and "failed”, if not).
While actions are running, the cancelOrder action must report "running”, until all actions are cancelled/finished. 
After all vehicle movements and all actions are stopped, the cancelOrder action status must report "finished”.

The orderId and orderUpdateId is kept. 

Figure 9 shows the expected behavior for different AGV capabilities.

![Figure 9 Expected behavior after a cancelOrder](./assets/Figure9.png)
>Figure 9 Expected behavior after a cancelOrder



#### <a name="Ranoac"></a> 6.6.3.1 Receiving a new task after cancellation

After the cancellation of an task, the vehicle must be in a state to receive a new task. 

In the case of an AGV that localizes itself on points via a tag, the new task has to begin on the point the AGV is now standing on (see also Figure 5).

In case of an AGV that can stop in-between points, the choice is up to RCS how the next task should be started. 
The AGV must accept both methods.

There are two options:

- Send an task, where the first point is a temporary point that is positioned where the AGV currently stands. The AGV must then realize that this point is trivially reachable and accept the task.
- Send an task, where the first point is the last traversed point of the previous task but set the deviation range so large that the AGV is within this range. Thus, the AGV must realize that this point must be counted as traversed and accept the task.



#### <a name="RacawAhno"></a> 6.6.3.2 Receiving a cancelOrder action when AGV has no task

If the AGV receives a cancelOrder action but the AGV currently has no task, or the previous task was cancelled, the cancelOrder action must report as failed.

The AGV must report a "noOrderToCancel” error with the errorLevel set to warning. 
The actionId of the instantaction must be passed as an errorReference.



### <a name="Or"></a> 6.6.4 task rejection

There are several scenarios, when an task must be rejected. 
These are explained in Figure 8.



#### <a name="Vgamno"></a> 6.6.4.1 Vehicle gets a malformed new task

Resolution:

1. Vehicle does NOT take over the new task in its internal buffer. 
2. The vehicle reports the warning "validationError"
3. The warning must be reported until the vehicle has accepted a new task.



#### <a name="Vraowaicpeglhhtmlholaansii"></a> 6.6.4.2 Vehicle receives an task with actions it cannot perform (例如 lifting height higher than maximum lifting height, or lifting actions although no stroke is installed), or with fields that it cannot use (例如 Trajectory)

Resolution: 

1. Vehicle does NOT take over the new task in its internal buffer 
2. Vehicle reports the warning "orderError" with the wrong fields as error references
3. The warning must not be reported until the vehicle has accepted a new task. 



#### <a name="Vehiclegets"></a> 6.6.4.3 Vehicle gets a new task with the same orderId, but a lower orderUpdateId than the current orderUpdateId

Resolution: 

1. Vehicle does NOT take over the new task in its internal buffer. 
2. Vehicle keeps the PREVIOUS task it its buffer. 
3. The vehicle reports the warning "orderUpdateError"
4. The vehicle continues with the executing the previous task. 

If the AGV receives an task with the same orderId and orderUpdateId twice, the second task will be ignored. 
This might happen, if RCS sends the task again, because the status message came too late and RCS could not verify that the first task was received.



### <a name="Maps"></a> 6.6.5 Maps

To ensure consistent navigation among different types of AGV, the position is always specified in reference to the local map coordinate system (see Figure 10).
For the differentiation between different levels a unique mapId is used.
The map coordinate system is to be specified as a right-handed coordinate system with the z-axis pointing skywards. 
A positive rotation therefore is to be understood as a counterclockwise rotation. 
The vehicle coordinate system is also specified as a right-handed coordinate system with the x-axis pointing in the forward direction of the vehicle and the z-axis pointing skywards. 
This is in accordance with chapter 2.11 in DIN ISO 8855.

![Figure 10 Coordinate system with sample AGV and orientation](./assets/Figure10.png)
>Figure 10 Coordinate system with sample AGV and orientation

The X, Y and Z coordinates must be in meters. 
The orientation must be in radians and must be within +Pi and –Pi.

![Figure 11 Coordinate systems for map and vehicle](./assets/Figure11.png)
>Figure 11 Coordinate systems for map and vehicle



## <a name="Iotom"></a> 6.7 Implementation of the task message

Object structure | Unit | Data type | Description 
---|---|---|---
headerId | | uint32 | Header ID of the message.<br> The headerId is defined per topic and incremented by 1 with each sent (but not necessarily received) message. 
timestamp | | string | Timestamp (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”)
version | | string | Version of the protocol [Major].[Minor].[Patch] (例如 1.3.2)
manufacturer | | string | Manufacturer of the AGV 
serialNumber | | string | Serial number of the AGV 
orderId |  | string | task identification.<br> This is to be used to identify multiple task messages that belong to the same task. 
orderUpdateId |  | uint32 | task update identification.<br>Is unique per orderId.<br>If an task update is rejected, this field is to be passed in the rejection message
zoneSetId |  | string | Unique identifier of the zone set, that the AGV has to use for navigation or that was used by RCS for planning. <br> <br> 可选: Some RCS systems do not use zones.<br> Some AGV do not understand zones.<br> Do not add to message, if no zones are used. 
**points [point]** |  | array | Array of points objects to be traversed for fulfilling the task. <br>One point is enough for a valid task. <br>Leave segment list empty for that case. 
**segments [segment]** |  | array | Array of segment objects to be traversed for fulfilling the task. <br>One point is enough for a valid task. <br>Leave segment list empty for that case.

Object structure | Unit | Data type | Description
---|---|---|---
**point** { |  | JSON-object|   
nodeId |   |  string | Unique point identification
sequenceId |  | uint32 | Number to track the sequence of points and segments in an task and to simplify task updates. <br>The main purpose is to distinguish between a point, which is passed more than once within one orderId. <br>The variable sequenceId runs across all points and segments of the same task and is reset when a new orderId is issued. 
*nodeDescription* |  | string | Additional information on the point 
released |  | boolean | "true" indicates that the point is part of the base. <br> "false" indicates that the point is part of the horizon. 
***nodePosition*** |  | JSON-object | point position. <br>可选 for vehicle-types that do not require the point position (例如, line-guided vehicles).
**actions [action]** <br> } |  | array | Array of actions to be executed on a point. <br>Empty array, if no actions required. 

Object structure | Unit | Data type | Description 
---| --- |--- | ---
**nodePosition** { |  | JSON-object | Defines the position on a map in a global project specific world coordinate system. <br>Each floor has its own map. <br>All maps must use the same project specific global origin. 
x | m | float64 | X-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation. 
y | m | float64 | Y-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation. 
*theta* | rad | float64 | Range: [-Pi ... Pi] <br><br>Absolute orientation of the AGV on the point.<br> 可选: vehicle can plan the path by itself.<br>If defined, the AGV has to assume the theta angle on this point.<br>If previous segment disallows rotation, the AGV must rotate on the point.<br>If following segment has a differing orientation defined but disallows rotation, the AGV is to rotate on the point to the segments desired rotation before entering the segment.
*allowedDeviationXY* |  | float64 | Indicates how exact an AGV has to drive over a point in task for it to count as traversed. <br><br> If = 0: no deviation is allowed (no deviation means within the normal tolerance of the AGV manufacturer). <br><br> If > 0: allowed deviation-radius in meters. <br>If the AGV passes a point within the deviation-radius, the point is considered to have been traversed.
*allowedDeviationTheta* |  | float64 | Range: [0 ... Pi] <br><br> Indicates how big the deviation of theta angle can be. <br>The lowest acceptable angle is theta - allowedDeviationTheta and the highest acceptable angle is theta + allowedDeviationTheta.
mapId |  | string | Unique identification of the map in which the position is referenced. <br> Each map has the same project specific global origin of coordinates. <br>When an AGV uses an elevator, 例如, leading from a departure floor to a target floor, it will disappear off the map of the departure floor and spawn in the related lift point on the map of the target floor.
*mapDescription* <br> } |  | string | Additional information on the map.

Object structure | Unit | Data type | Description 
---|---|---|---
**action** { |  | JSON-object | Describes an action that the AGV can perform. 
actionType |  | string | Name of action as described in the first column of "actions and Parameters”. <br> Identifies the function of the action. 
actionId |  | string | Unique ID to identify the action and map them to the actionstate in the state. <br>Suggestion: Use UUIDs.
*actionDescription* |  | string | Additional information on the action
blockingType |  | string | Enum {NOTE, SOFT, HARD}: <br> "NONE"- allows driving and other actions;<br>"SOFT"- allows other actions, but not driving;<br>"HARD"- is the only allowed action at that time.
***actionParameters [actionParameter]*** <br><br> } |  | array | Array of actionParameter-objects for the indicated action, 例如, deviceId, loadId, external Triggers. <br><br> See "actions and Parameters"

Object structure | Unit | Data type | Description 
---|---|---|---
**segment** { |  | JSON-object | Directional connection between two points.
edgeId |  | string | Unique segment identification.
sequenceId |  | Integer | Number to track the sequence of points and segments in an task and to simplify task updates. <br>The variable sequenceId runs across all points and segments of the same task and is reset when a new orderId is issued.
*edgeDescription* |  | string | Additional information on the segment.
released |  | boolean | "true" indicates that the segment is part of the base.<br>"false" indicates that the segment is part of the horizon. 
startNodeId |  | string | nodeId of startNode.
endNodeId |  | string | nodeId of endNode.
*maxSpeed* | m/s | float64 | Permitted maximum speed on the segment. <br>Speed is defined by the fastest measurement of the vehicle.
*maxHeight* | m | float64 | Permitted maximum height of the vehicle, including the load, on segment.
*minHeight* | m | float64 | Permitted minimal height of the load handling device on the segment.
*orientation* | rad | float64 | Orientation of the AGV on the segment. The value *orientationType* defines if it has to be interpreted relative to the global project specific map coordinate system or tangential to the segment. In case of interpreted tangential to the segment 0.0 = forwards and PI = backwards. <br>Example: orientation Pi/2 rad will lead to a rotation of 90 degrees.<br><br>If AGV starts in different orientation, rotate the vehicle on the segment to the desired orientation if rotationAllowed is set to "true”.<br>If rotationAllowed is "false", rotate before entering the segment.<br>If that is not possible, reject the task.<br><br>If no trajectory is defined, apply the rotation to the direct path between the two connecting points of the segment.<br>If a trajectory is defined for the segment, apply the orientation to the trajectory. 
*orientationType* |  | string | Enum {`GLOBAL`, `TANGENTIAL`}: <br>"GLOBAL"- relative to the global project specific map coordinate system;<br>"TANGENTIAL"- tangential to the segment.<br><br>If not defined, the default value is "TANGENTIAL".
*direction* |  | string | Sets direction at junctions for line-guided or wire-guided vehicles, to be defined initially (vehicle-individual).<br> Examples: left,  right, straight, 433MHz.
*rotationAllowed* |  | boolean | "true”: rotation is allowed on the segment.<br>"false”: rotation is not allowed on the segment.<br><br>可选:<br>No limit, if not set.
*maxRotationSpeed* | rad/s | float64| Maximum rotation speed<br><br>可选:<br>No limit, if not set.
***trajectory*** |  | JSON-object | Trajectory JSON-object for this segment as a NURBS. <br>Defines the curve, on which the AGV should move between startNode and endNode.<br><br>可选:<br>Can be omitted, if AGV cannot process trajectories or if AGV plans its own trajectory.
*length* | m | float64 | Length of the path from startNode to endNode<br><br>可选:<br>This value is used by line-guided AGVs to decrease their speed before reaching a stop position. 
**action [action]**<br><br><br> } |  | array | Array of actionIds to be executed on the segment. <br>Empty array, if no actions required. <br>An action triggered by an segment will only be active for the time that the AGV is traversing the segment which triggered the action. <br>When the AGV leaves the segment, the action will stop and the state before entering the segment will be restored.

Object structure | Unit | Data type | Description 
---|---|---|---
**trajectory** { |  | JSON-object |  
degree |  | float64 | Range: [1 ... infinity]<br><br>Defines the number of control points that influence any given point on the curve. Increasing the degree increases continuity.<br><br>If not defined, the default value is 1.
**knotVector [float64]** |  | array | Range: [ 0.0 ... 1.0]<br><br>Sequence of parameter values that determines where and how the control points affect the NURBS curve.<br><br>knotVector has size of number of control points + degree + 1.
**controlPoints [controlPoint]**<br><br> } |  | array | List of JSON controlPoint objects defining the control points of the NURBS, which includes the beginning and end point.

Object structure | Unit | Data type | Description 
---|---|---|---
**controlPoint** { |  | JSON-object |  
x |  | float64 | X coordinate described in the world coordinate system. 
y |  | float64 | Y coordinate described in the world coordinate system.actions
*weight* |  | float64 | Range: (0 ... infinity)<br><br>The weight, with which this control point pulls on the curve.<br>When not defined, the default will be 1.0.
} |  |  |


## <a name="actions"></a> 6.8 actions

AGV如果支持驾驶以外的其他actions,则这些actions将通过附加到点或段的action字段执行,或通过单独的主题Instantaction发送(请参阅6.9).

在段上执行的actions,仅限AGV在片段上运行时执行(请参见6.10.2).

actions that are triggered on points can run as long as they need to run. 
`在点上触发的actions,可以在AGV需要的时候执行.`

点上的actions应该是自动终止(完成)的(例如,蜂鸣器信号持续五秒钟或取货action,在取货后自动完成)或者成对设计(例如,AcivalateWarningLights和UnctivateWarninglights),尽管有可能存在.

以下部分介绍了AGV必须使用的预定义actions,....

如果有明确定义的参数,参数必须被使用.
额外的参数也可以被定义,以便成功执行action.

如果无法将某些action映射到以下部分的actions之一,则AGV制造商可以定义RCS必须使用的其他actions.


### <a name="Padtpeas"></a> 6.8.1 预定义action 定义, 参数, 效果 和 范围

general |  | scope 
:---:|--- | :---:
action, counter action, Description, idempotent, Parameter | linked state |  instant, point, segment 
action,counter action,描述,幂等,参数| 链接状态| 即时(立即),点,片段

action | counter action | Description | idempotent | Parameter | linked state | instant | point | segment
---|---|---|---|---|---|---|---|---
startPause | stopPause | 激活暂停模式. <br>连接状态是必须的,因为很多AGVs可以被硬件开关暂停. <br>AGV不继续运动 - 到下一个点不是必须的.<br>actions可以继续. <br>task是可以恢复的. | yes | - | paused | yes | no | no 
stopPause | startPause | 停用暂停模式. <br>移动和所有其他actions将恢复 (如果有的话).<br>连接状态是必须的,因为很多AGVs可以被硬件开关暂停. <br>stopPause可以恢复硬件触发的停止车辆(例如软停)(如果配置). | yes | - | paused | yes | no | no 
startCharging | stopCharging | 激活充电流程. <br>可以在充电位置进行充电 (停车状态)或者在一个charging lane (运行时). <br>防止过度充电是车辆的责任. | yes | - | .batteryState.charging | yes | yes | no
stopCharging | startCharging | 解除充电流程去接任务. <br>充电过程也可以被车辆或者充电站中断, 例如,如果电池已满. <br>电池状态仅被允许为 "false”, 当AGV准备接收任务时. | yes | - |.batteryState.charging | yes | yes | no
initPosition | - | 重新设置 (overrides) 具有给定参数的AGV的位置姿态. | yes | x  (float64)<br>y  (float64)<br>theta  (float64)<br>mapId  (string)<br>lastNodeId  (string) | .agvPosition.x<br>.agvPosition.y<br>.agvPosition.theta<br>.agvPosition.mapId<br>.lastNodeId | yes | yes<br>(Elevator) | no 
stateRequest | - | 请求AGV发送新的状态报告. | yes | - | - | yes | no | no 
logReport | - | 请求AGV生成和存储日志报告. | yes | reason<br>(string) | - | yes | no | no 
pick | drop<br><br>(如果自动化) | 请求AGV取货. <br>带有多个负载处理设备的AGV可以并行处理多个取货操作. <br>在这种情况下,需要存在参数LHD (例如. LHD1). <br>参数stationType 说明如何详细处理取货操作 (例如, 楼层位置, 货架位置, 被动输送机, 主动输送机, 等等.). <br>load type 展示load unit 并且可以用来切换field 例如 (例如, EPAL, INDU, 等等). <br>用于准备负载处理设备 (例如, 基于高度参数的提升前动作), 动作(action)可以在horizon高级项里定义. <br>注意, 提升前动作(pre-Lift)等, 不会再AGV运行中上报,因为关联点尚未释放.<br>如果车辆在一个片段上,可以使用它自己的传感器设备检测取货点的位置. | no |lhd (string, 可选)<br>stationType (string)<br>stationName(string, 可选)<br>loadType (string) <br>loadId(string, 可选)<br>height (float64) (可选)<br>定义货物底部高度related to the floor<br>depth (float64) (可选) for forklifts<br>side(string) (可选) 例如 conveyor | .load | no | yes | yes 
drop | pick<br><br>(如果自动化) | 请求AGV放货. <br>更多细节查看取货action. | no | lhd (string, 可选)<br>stationType (string, 可选)<br>stationName (string, 可选)<br>loadType (string, 可选)<br>loadId(string, 可选)<br>height (float64, 可选)<br>depth (float64, 可选) <br>… | .load | no | yes | yes
detectObject | - | AGV检测对象(例如 货物, 充电点, 自由停车位置). | yes | objectType(string, 可选) | - | no | yes | yes 
finePositioning<br>精准寻迹(上线) | - | 对于站点, AGV将精确寻迹到目标点.<br>AGV允许偏离点位置.<br>对于片段, AGV will 例如 align on stationary equipment while traversing an segment.<br>Instantaction: AGV starts positioning exactly on a target. | yes | stationType(string, 可选)<br>stationName(string, 可选) | - | no | yes | yes
waitForTrigger | - | AGV需要等待触发信号(例如按压按钮,手动装货). <br>如果需要,RCS负责处理超时和取消任务. | yes | triggerType(string) | - | no | yes | no 
cancelOrder | - | AGV应尽可能停止. <br>需要立即执行或者到下一个点. <br>然后任务删除,所有actions取消. | yes | - | - | yes | no | no 
factsheetRequest | - | 请求AGV资料单factsheet | yes | - | - | yes | no | no 



### <a name="Padtpeas1"></a> 6.8.2 预定义action 的定义和状态描述 

action | action states 
---|---
   初始, 运行, 暂停, 完成, 失败 |

action | 初始|运行|暂停|完成|失败 
---|---|---|---|---|---
startPause | - | 该模式的切换(激活)正在准备中. <br>如果AGV支持立即切换(暂停状态),这个(运行)状态可以被忽略. | - | 车辆静止不动. <br>所有actions将暂停. <br>暂停模式被激活. <br>AGV上报 .paused: true. | 某些情况下不能被激活(例如,被硬件开关控制).
stopPause | - | 该模式的切换(解除)正在准备中. <br>如果AGV支持立即切换(暂停解除状态),这个(运行)状态可以被忽略. | - | 暂停被解除. <br>所有暂停的actions将恢复继续. <br>AGV上报 .paused: false. | 某些情况下不能被激活 (例如,被硬件开关控制). 
startCharging | - | 激活充电流程正在进行中 (正在与充电桩交互中). <br>如果AGV支持立即切换(充电状态),这个(运行)状态可以被忽略. | - | 开始充电. <br>AGV上报 .batteryState.charging: true. | 某些情况下不能被激活充电 (例如, 没有对齐充电桩).充电问题应对应错误码. 
stopCharging | - | 解除充电流程正在进行中 (正在与充电桩交互中). <br>如果AGV支持立即切换(非充电状态),这个(运行)状态可以被忽略. | - | 充电流程终止. <br>AGV上报 .batteryState.charging: false | 某些情况下充电流程不能停止 (例如, 没有对齐充电桩).<br> 充电问题应对应错误码. 
initPosition | - | 新姿势的初始化 (confidence 检查 等等.). <br>如果AGV支持立即切换,这个(运行)状态可以被忽略. | - | pose重置了. <br>AGV 上报 <br>.agvPosition.x = x, <br>.agvPosition.y = y, <br>.agvPosition.theta = theta <br>.agvPosition.mapId = mapId <br>.agvPosition.lastNodeId = lastNodeId | pose无效或者不能被重置. <br>定位问题应有错误码.
stateRequest | - | - | - | state已经发送 | - 
logReport | - | 正在创建日志. <br>如果AGV支持立即切换,这个(运行)状态可以被忽略. | - | 日志已经完成记录. <br>日志名称将在上报状态内. | 日志无法保存 (例如,没有空间).
pick | 初始化取货流程, 例如, outstanding 举升操作. | 取货流程执行中 (AGV进入站点, 货物处理设备忙碌, 与站台的通信正在进行中, 等等.). | 取货流程暂停中, 例如,安全防护检测异常. <br>安全防护异常解除后, 操作继续. | 取货完成. <br>货物到位并且AGV上报新的负载状态. | 取货失败, 例如, 站台无货. <br> 取货失败应有错误码.
drop | 初始化放货流程, 例如, outstanding 举升操作. | 放货流程执行中  (AGV进入站点, 货物处理设备忙碌, 与站台的通信正在进行中, 等等.). | 放货流程执行中, 例如, ,安全防护检测异常. <br>安全防护异常解除后, 操作继续. | 放货完成. <br>货物离开并且AGV上报新的负载状态. | 放货失败, 例如, 站台被占用.  <br>放货失败应有错误码. 
detectObject | - | 目标检测运行中. | - | 目标检测到. | AGV无法检测到目标. 
finePositioning | - | AGV精确定位自己到一个目标上. | 精准寻迹暂停中, 例如,安全防护检测异常. <br>安全防护异常解除后, 寻迹继续. | 到达提供的目标位置. | 提供的目标位置无法达到. 
waitForTrigger | - | AGV正在等待触发信号 | - | 触发信号获取到. | 如果任务被取消waitForTrigger失败. 
cancelOrder | - | AGV正在停止或者运行,直到到达下个点. | - | AGV静止不动并且取消任务. | - 
factsheetRequest | - | - | - | 资料单factsheet已经传递 | - 



## <a name="Tifmc"></a> 6.9 MQTT Topic: "instantActions" (从RCS to control to AGV)

在某些情况下,需要将actions发送到AGV,并且立即执行.通过将instantAction消息发布instantActions主题来实现.instantActions不得与AGV当前任务的内容相抵触(例如:instantAction降低货叉,而任务说要抬高货叉). 

 一些立即执行动作的例子: 
  - AGV暂停,不更改当前任务中的任何内容; 
  - 暂停后恢复任务; 
  - 激活信号(灯光,蜂鸣器等). 

额外信息,请参考第8章最佳实践.

Object structure | | Data type | Description 
---|---|---|---
headerId | | uint32 | 信息头ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如, "2017-04-15T11:40:03.12Z”)
version | | string | 协议版本 [Major].[Minor].[Patch] (例如, 1.3.2).
manufacturer | | string | AGV厂商. 
serialNumber | | string | AGV序列号.
Actions [action] | | array | 需要立即执行的动作组并且不是常规任务重的一部分. 

当AGV收到一个立即动作,AGV状态(state)需要加入适合的actionStatus到actionStates;actionStatus依据动作的进度进行更新,查看图12,区分actionStatus的不同transitions;


## <a name="TSfAtmc"></a> 6.10 MQTT Topic: "state" (从 AGV 到 RCS)

AGV状态将仅在一个主题topic上传输.相比不同的消息 (例如, 针对 任务, 电池状态和错误码) 使用同一个topic将减少broker/RCS的工作量,同时还保持AGV状态的信息同步.AGV-State信号 和相关事件触发一起或者至少每30s 通过MQTT-Broker发布给RCS.

触发传输AGV状态消息的事件:
- 接收任务 
- 接收任务更新
- 负载状态的变化
- 错误或警告
- 运行通过一个点
- 切换操作模式
- "driving" 变化 
- nodeStates, edgeStates 或者 actionStates 变化

~~应该努力尽量减少通信次数~~.如果2个事件有关联(例如,接收新任务通常强制一个点和边的状态更新;像是运行通过一个点),应该触发一个状态更新而不是多个.


### <a name="CaLe"></a> 6.10.1 概念和逻辑

任务进度由 `nodeStates` 和 `edgeStates`跟踪. 另外,如果AGV可以获得当前位置,可以通过"position”字段发布它的位置.

如果AGV自己规划路线,必须将它的计算轨迹(包含base和horizon) 以NURBS(曲线)形式通过状态消息内的`trajectory`对象传递, 除非RCS不能使用这个字段并且在集成时候确定了,那么这个自动不能被发送.一旦点被RCS确定(released), AGV不允许改变轨迹路线.

`nodeStates` 和 `edgeStates`包含所有的点/片段,AGV仍旧必须要通过的.

![Figure 12 任务信息由state topic提供. 仅传输最后一点和剩余点和段的ID](./assets/Figure12.png) 
>Figure 12 任务信息由state topic提供. 仅传输最后一点和剩余点和段的ID

### <a name="Tonaeletoa"></a> 6.10.2 遍历点和进入/离开片段,动作触发 

AGV自己判断一个点什么时候被计算为已经通过.通常,AGV控制点需要在 the node’s `deviationRangeXY` 并且方向角度在`deviationRangeTheta`.

AGV上报遍历点通过从`nodeStates`数组移除`nodeState`并且设置`lastNodeId`, `lastNodeSequenceNumber`为遍历点的值;

当AGV上报遍历点的时候,必须触发这个点设置的Actions,如果存在的情况;

点的遍历同样标志着离开指向点的片段.这个片段必须从`edgeStates`删除并且这个片段上激活的Actions必须完成;

该点的遍历也标志着一个时刻, AGV进入记下来的片段,如果有一个片段的话,这个片段的Actions必须立即触发.这条规则里外的情况是,如果AGV在片段上暂停 (因为软停或者hard blocking segment,或者其他) – 然后AGV进入片段当它开始重新移动.

![Figure 13 nodeStates, edgeStates, actionStates 在任务处理过程中](./assets/Figure13.png)
>Figure 13 nodeStates, edgeStates, actionStates 在任务处理过程中



### <a name="Br"></a> 6.10.3 基础请求Base request 

If the AGV detects, that its base is running low, it can set the `newBaseRequest` flag to `true` to prevent unnecessary braking.
如果AGV检测到,它的base运行过短,可以设置`newBaseRequest`标志为`true`避免不必要的刹车.


### <a name="Information"></a> 6.10.4 Information 


AGV可以通过`information`数组提交任意的其他信息给RCS.它通过information消息传递,取决于AGV多久上报information;

RCS逻辑上不能使用这心信息消息,只能用来可视化或者debug目的.



### <a name="Errors"></a> 6.10.5 Errors 

AGV通过`errors`数组上报错误码. 错误有两种级别`WARNING` 和 `FATAL`.`WARNING`是一个可以自动解除的错误,例如,防护入侵. `FATAL`错误需要人干预.错误可以传递说明,有助于通过errorReferences组查找错误的原因.

### <a name="Implementation"></a> 6.10.6 Implementation(任务?)

Object structure | Unit | Data type | Description 
---|---|---|---
headerId | | uint32 | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”).
version | | string | 通讯协议 [Major].[Minor].[Patch] (例如 1.3.2).
manufacturer | | string | AGV厂商.
serialNumber | | string | AGV序列号.
orderId|  | string | 当前任务或先前完成任务的唯一任务标识. <br>orderId保持不变直到新任务. <br>空字符串 (""),如果没有有效的前orderId. 
orderUpdateId |  | uint32 | `用来识别任务更新标识,已经被AGV获取的任务更新`. <br>"0”如果没有有效的前orderUpdateId. 
*zoneSetId* |  |string |AGV当前用于路径规划的区域(不同楼或者位置区域)的唯一ID. <br>必须与任务中使用的区域id相同,否则AGV必须拒绝任务.<br><br>可选:如果AGV不使用zones,字段可以忽略.
lastNodeId |  | string | 上次到达的点id或者AGV当前点id (例如, "node7”). 如果没有有效的lastNodeId,留空字符串 ("").
lastNodeSequenceId |  | uint32 | 最后到达点的序列ID 或者AGV当前点的序列ID. <br>"0"如果没有有效的前lastNodeSequenced. 
**nodeStates [nodeState]** |  |array | nodeState-Objects数组, 需要穿过以完成任务<br>(空闲时空list)
**edgeStates [edgeState]** |  |array | edgeState-Objects数组, 需要穿过以完成任务<br>(空闲时空list)
***agvPosition*** |  | JSON-object | AGV当前在地图上的位置.<br><br>可选:<br><br>只能被无定位能力的AGV忽略, 例如, 线引导AGV(磁导航?).
***velocity*** |  | JSON-object | 车辆坐标的AGV速度. 
***loads [load]*** |  | array | 负载(多个), 当前被AGV控制的.<br><br>可选:如果AGV不能确定负载状态,不要放入state. <br>如果AGV可以确定负载状态,但是数组为空,则将AGV视为空载(无货).
driving |  | boolean | "true”: 表示,AGV在行走并且/或者在旋转.AGV其他移动(例如, 货叉移动)不包含在这里.<br><br>"false”:表示AGV既不是在行走也不在旋转.
*paused* |  | boolean | "true”: AGV是暂停,要么是物理按钮暂停要么是立即动作暂停.<br>AGV可以恢复任务.<br><br>"false”: 当前AGV不在暂停模式.
*newBaseRequest* |  | boolean | "true”: AGV几乎到达base的终点,如果没有新的base传达,将会减速. <br>触发RCS下发新base.<br><br>"false”: 不需要base更新.
*distanceSinceLastNode* | meter | float64 | line guided 车辆使用,显示行走过"lastNodeId"的距离. <br>距离是米为单位.
**actionStates [actionstate]** |  | array | 包含一组尚未完成当前动作和动作组. <br>可能包含之前的点的动作,仍旧在执行中.<br><br>当一个动作完成,一个更新的state消息发布并且设置actionStatus为完成并且如果可行包含相应结果的描述. <br><br>action state持续保持到收到新任务.
**batteryState** |  | JSON-object | 包含所有与电池相关的信息.
operatingMode |  | string | Enum {AUTOMATIC, SEMIAUTOMATIC, MANUAL,  SERVICE,  TEACHIN}<br>有关其他信息, 查看表格OperatingModes 6.10.6. 
**errors [error]** |  | array |错误对象的数组. <br>AGV的所有当前激活的错误都应在列表中.<br>一个空列表表示AGV没有当前激活的错误.
***information [info]*** |  | array | 信息对象数组. <br>一个空数组表明AGV没有信息. <br>这仅应用于可视化或调试 – 它不能在RCS中用于逻辑判断.
**safetyState** |  | JSON-object | 包含所有与安全有关的信息. 

Object structure | Unit | Data type | Description 
---|---|---|---
**nodeState** { | JSON-object |  |
nodeId |  | string | 点的唯一标识.
sequenceId |  | uint32 | sequenceId(顺序id?)用来分辨同一个nodeId的多个点.
*nodeDescription* |  | string | 有关该点的其他信息.
***nodePosition*** |  | JSON-object | 点位置. <br>对象在6.6章节中定义  <br>可选: <br>RCS有此信息. <br>可以另外发送, 例如. 用来debug.
released<br><br>}|  | boolean | "true”表示点是base的一部分.<br>"false”表示点是horizon的一部分.

Object structure | Unit | Data type | Description 
---|---|---|---
**edgeState** { |  | JSON-object |  |
edgeId |  | string | 段的唯一标识.
sequenceId |  | uint32 | sequenceId(顺序id?)用来分辨同一个edgeId的多个片段.
*edgeDescription* |  | string | 有关片段的其他信息.
released |  | boolean | "true” 表示该段是base的一部分.<br>"false” 表示该段是horizon的一部分.
***trajectory*** <br><br>} |  | JSON-object | 轨迹以NUBS曲线传递,在6.4定义<br><br>轨迹段是从AGV开始进入段的点,直到报告的下一个遍历点.

Object structure | Unit | Data type | Description
---|---|---|---
**agvPosition** { |  | JSON-object | 定义世界坐标系中地图上的位置.每个楼层都有自己的地图.
positionInitialized |  | boolean | "true”: 位置正在初始化.<br>"false”: 位置没有初始化.
*localizationScore* |  | float64 | Range: [0.0 ... 1.0]<br><br>描述定位的质量,以便可以使用, 例如 被SLAM-AGV描述, 当前位置信息的准确程度.<br><br>0.0: 位置未知<br>1.0: 已知位置<br><br>可选 对于车辆无法估计其定位质量的.<br><br>仅用于记录和可视化目的. 
*deviationRange* | m | float64 | 用米定义的位置偏差范围.<br><br>可选 对于无法估计其偏差的车辆 例如 grid-based 定位.<br><br>仅用于记录和可视化目的.
x | m | float64 | 地图上的X位置坐标 参考地图坐标系. <br>精度取决于特定的实现(任务?).
y | m | float64 | 地图上的Y位置坐标 参考地图坐标系. <br>精度取决于特定的实现(任务?).
theta |  | float64 | Range: [-Pi ... Pi]<br><br>AGV的方向(弧度?). 
mapId |  | string | 地图位移标识id,用来定位的.<br><br>每个地图具有相同的原始坐标. <br>当AGV使用电梯时, 例如, 从一个楼层到另一个楼层,它将小时在离开楼层的地图出现在目标楼层电梯点.
*mapDescription*<br>} |  | string | 地图上的其他信息. 

Object structure | Unit | Data type | Description 
---|---|---|---
**velocity** { |  | JSON-object |  
*vx* | m/s | float64 | AVGS在其X方向上的速度.
*vy* | m/s | float64 | AVGS沿其Y方向上的速度.
*omega*<br>}| Rad/s | float64 | AVG在其Z轴上转动速度.

Object structure | Unit | Data type | Description 
---|---|---|---
**load** { |  | JSON-object |  
*loadId* |  | string | 负载的唯一标识号 (例如, 条形码 或者 RFID).<br><br>空字段,如果AGV可以识别负载,但尚未识别负载.<br><br>可选, 如果AGV无法识别负载.
*loadType* |  | string | 负载类型.
*loadPosition* |  | string | 指示使用AGV的哪个负载处理/载荷单元, 例如, 如果AGV有多个spots/位置可以载货.<br><br>例如: "前", "后", "positionC1”, 等等.<br><br>可选 对于只有一个负载位置的车辆
***boundingBoxReference*** |  | JSON-object | 货物边界 位置的参考点. <br>参考点通常是货物边界 底部表面的中心 (在height = 0) 并且 在AGV坐标系的坐标中描述.
***loadDimensions*** |  | JSON-object | 负载货物边界的尺寸(米). 
*weight*<br><br>} | kg | float64 | 范围: [0.0 ... 无穷)<br><br>测量负载的绝对重量(kg). 

Object structure | Unit | Data type | Description 
---|---|---|---
**boundingBoxReference** { |  | JSON-object | 货物边界位置的参考点. <br>参考点通常是货物边界 底部表面的中心 (在height = 0) 并且 在AGV坐标系的坐标中描述.
x |  | float64 | 参考点的x坐标. 
y |  | float64 | 参考点的y坐标.
z |  | float 64 | 参考点的z坐标. 
*theta*<br> } |  | float64 | 货物边界的方向. <br>对于输送机tugger, trains等很重要. 

Object structure | Unit | Data type | Description 
---|---|---|---
**loadDimensions** { |  | JSON-object | 货物边界框的尺寸(米). 
length | m | float64 | 货物边界框的绝对长度. 
width | m | float64 | 货物边界框的绝对宽度. 
*height* <br><br><br><br>}| m | float64 | 货物边界框的绝对高度.<br><br>可选:<br><br>仅在已知的情况下设置值.

Object structure | Unit | Data type | Description 
---|---|---|---
**actionState** { |  | JSON-object |  
actionId |  |string  | action_ID
*actionType* |  | string | 动作的动作类型.<br><br>可选: 仅出于信息或可视化目的.任务知道类型.
*actionDescription* |  | string | 有关当前动作的其他信息. 
actionStatus |  | string | Enum {WAITING; INITIALIZING; RUNNING; PAUSED; FINISHED; FAILED}<br><br>WAITING: 等待触发<br>(passing the mode, 进入片段)<br> PAUSED: 通过立即动作或外部触发暂停<br>FAILED: 无法执行动作. 
*resultDescription*<br><br><br><br>} |  | string | 结果的描述, 例如, RFID-读取数据.<br><br>错误将在错误消息中传达.<br><br>results例子在6.5

Object structure | Unit | Data type | Description 
---|---|---|---
**batteryState** { |  | JSON-object |  
batteryCharge | % | float64 | 充电状态: <br> 如果AGV仅提供好或差电池水平的值,则将这些值表示为20％（差）和80％（好）. 
*batteryVoltage* | V | float64 | 电池电压.
*batteryHealth* | % | int8 | 范围: [0 .. 100]<br><br>健康状况. 
charging |  | boolean | "true”: 正在充电中.<br>"false”: AGV目前不充电.
*reach* <br><br>}| m | uint32 | 范围: [0 ... 无穷)<br><br>估算当前充电状态. 

Object structure | Unit | Data type | Description 
---|---|---|---
**error** { |  | JSON-object |  
errorType |  | string | Type/name of error 
***errorReferences [errorReference]*** |  | array | 参考列表 用来识别错误的来源 (例如, headerId, orderId, actionId, 等等.).<br>有关其他信息,请参见"Best practices" 第8章.
*errorDescription* |  | string | 错误说明. 
errorLevel <br><br> }|  | string | Enum {WARNING, FATAL}<br><br>WARNING: AGV准备开始 (例如 维护保养周期到期警告).<br>FATAL: AGV不处于运行状态,需要用户干预 (例如 激光扫描仪脏了).

<a name="errorReferenceImpl"></a>
Object structure | Unit | Data type | Description 
---|---|---|---
**errorReference** { |  | JSON-object |  
referenceKey |  | string | 参考类型References the type of reference (例如, headerId, orderId, actionId, 等等.).
referenceValue <br>} |  | string | 参考值.

Object structure | Unit | Data type | Description 
---|---|---|--- 
**info** { |  | JSON-object |  
infoType |  | string | Type/name of information. 
*infoReferences [infoReference]* |  | array | 参考数组. 
*infoDescription* |  | string | Info说明. 
infoLevel <br><br><br>}|  | string | Enum {DEBUG,INFO}<br><br>DEBUG: 用来调试.<br> INFO: 用于可视化. 

Object structure | Unit | Data type | Description 
---|---|---|---
**infoReference** { |  | JSON-object |  
referenceKey |  | string | 参考类型References the type of reference(例如, headerId, orderId, actionId, 等等.).
referenceValue <br>} |  | string | 参考值.


Object structure | Unit | Data type | Description 
---|---|---|---
**safetyState** { |  | JSON-object |  
eStop |  | string | Enum {AUTOACK,MANUAL,REMOTE,NONE}<br><br>Acknowledge-Type of eStop:<br>AUTOACK: auto-acknowledgeable e-stop激活, 例如, 通过防撞条或安全防护区.<br>MANUAL: e-stop 将在车辆上手动承认.<br>REMOTE: facility e-stop has to be acknowledged remotely.<br>NONE: 无e-stop激活.
fieldViolation<br><br>} |  | boolean | 防护区域侵犯.<br>"true":区域被侵犯<br>"false":区域没有被侵犯.

#### Operating Mode Description
以下说明列出了"states"中的操作模式.

Identifier | Description 
---|---
AUTOMATIC | AGV完全控制RCS. <br>AGV根据RCS的任务运行和执行操作.
SEMIAUTOMATIC | AGV被RCS控制.<br> AGV根据RCS的任务运行和执行操作. <br>行驶速度由HMI控制 (速度不能超过自动模式的速度).<br>转向在自动控制下 (非安全的HMI).
MANUAL | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>HMI可用于控制AGV的转向和速度和处理设备. <br>AGV的位置发送给RCS. <br>当AGV进入或离开此模式时,立即清除所有任务 (安全HMI需要).
SERVICE | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>授权个人可以重新配置AGV. 
TEACHIN | RCS不控制AGV. <br>管理者不会将驾驶任务或动作发送到AGV. <br>正在训练AGV, 例如,录入地图?mapping is done by a RCS.



## <a name="actionStates"></a> 6.11 actionStates

当AGV接收到`action` (不论是关联在`point`或者`segment`或者通过`instantaction`), 必须将`action`反馈在`actionstate` 在`actionStates`组里.

`actionStates` 在字段`actionStatus`中描述了动作生命周期的哪个阶段.

表 1 描述, 哪个actionStatus枚举可以保持. 

actionStatus | Description 
---|---
WAITING | AGV接收到动作,单数AGV没有到达触发的点或者没有进入激活的片段.
INITIALIZING | 动作触发, 启动准备措施.
RUNNING | 动作正在运行.
PAUSED | 由于立即动作或外部触发(AGV上的暂停按钮),该动作被暂停 
FINISHED | 动作完成了. <br>结果通过resultDescription报道
FAILED | 无论出于何种原因,都无法完成动作.

>表 1 actionStatus 字段的可接受值

图14中提供了状态过渡图.

![图 14 actionStates所有可能的过渡状态](./assets/Figure14.png)
>图 14 actionStates所有可能的过渡状态



## <a name="ABTas"></a> 6.12 动作阻塞类型和顺序Action Blocking Types and Sequence

包含多个动作的任务 在需要执行的动作列表中定义顺序. 
动作的并行执行被它们的`blockingType`管理.

动作可以具有三种不同的阻塞类型, 表2中描述. 

actionStatus | Description 
---|---
NONE | 行动可以与其他动作并行执行,可以在车辆行驶时执行.
SOFT | 动作可以与其他动作并行执行,不能在车辆行驶中执行.
HARD | 不得与其他动作并行执行操作. 不能在车辆行驶中执行.

>表2动作阻塞类型

如果在同一点上有多个动作,不同的阻塞类型,图15描述了AGV应如何处理这些动作.

![图15处理多个动作](./assets/Figure15.png)
>图15处理多个动作



## <a name="TV"></a> 6.13 MQTT Topic "visualization" 

对于近乎实时的位置,AGV可以广播其位置和速度通过消息 `visualization`.

位置消息的结构 与状态中的位置和速度消息 结构相同,有关其他信息,请参见第6.7章实施,该消息主题的更新速率由集成商控制.



## <a name="Tc"></a> 6.14 MQTT Topic "connection"

在将AGV客户端连接到broker时,可以设置最后一个will主题和消息,该主题和消息是由broker与AGV client断开连接后发布的. 
因此,RCS可以通过订阅所有AGV的连接主题来检测断开事件. 
通过broker和客户端之间通过心跳检测到断开连接. 
该间隔在大多数brokers中都是可配置的,应设置约15秒钟. 
"connection"主题的服务质量至少应为1-至少一次.

建议的最后一个主题结构是:

**uagv/v2/manufacturer/SN/connection**

最后一条will消息定义为带有以下字段的JSON封装消息:

Identifier | Data type | Description 
---|---|---
headerId |uint32 | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). 
timestamp | string | 日期时间 (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ (例如"2017-04-15T11:40:03.12Z”).
version | string | 通讯协议 [Major].[Minor].[Patch] (例如 1.3.2).
manufacturer | string | AGV厂商.
serialNumber | string | AGV序列号.
connectionState | string | Enum {`ONLINE`, `OFFLINE`, `CONNECTIONBROKEN`}<br><br>`ONLINE`:AGV和broker连接着<br><br>`OFFLINE`: AGV与broker的连接以一种协调的方式离线. <br><br> `CONNECTIONBROKEN`: AGV与broker的连接以外结束. 

当连接以优雅的方式结束(使用MQTT断开连接命令以优雅的方式结束时),将不会发送最后一条will消息. 
如果连接意外中断,则仅由broker发送最后一条will消息.

**注意**: 由于MQTT中最后一个will功能的性质, 最后一条will消息 是在AGV和MQTT broker之间连接阶段 定义的.
结果，时间戳和headerld字段将始终过期.

AGV希望优雅地断开连接: 

1. AGV发送 "uagv/v2/manufacturer/SN/connection" 包含`connectionState`设置`OFFLINE`.
2. 用断开命令断开MQTT连接.

AGV上线: 

1. 将最后一个will设置为 "uagv/v2/manufacturer/SN/connection" 包含字段`connectionState`且设置`CONNECTIONBROKEN`, 当创建MQTT连接时.
2. 发送主题 "uagv/v2/manufacturer/SN/connection" 包含 `connectionState` 设置`ONLINE`.

该主题上的所有消息均应带有保留标志。

当AGV和broker之间的连接意外停止时, broker将发送最后一个will主题: "uagv/v2/manufacturer/SN/connection" 包含字段`connectionState`,且设置 `CONNECTIONBROKEN`.

## <a name="Tf"></a> 6.16 MATT Topic "factsheet"

FactSheet提供了有关特定AGV类型系列的基本信息.
该信息允许比较不同的AGV类型,可以应用于AGV系统的规划,尺寸和仿真.FactSheet还包括有关将AGV类型系列集成到一个AGV VDA-5050-compliant RCS通信接口的信息.

AGV FactSheet中某些字段的值只能在系统集成期间指定,例如,特定于项目的负载和站台类型的分配,以及该AGV支持的站点和负载类型列表.

factsheet既旨在作为人类可读文档,又用于机器处理, 例如, RCS应用程序导入, 因此被指定为JSON文档.

RCS可以通过发送立即动作`factsheetRequest` 从AGV请求事实表.

该主题上的所有消息均应带有保留标志.

### 6.16.1 Factsheet JSON strcture
The factsheet consists of the JSON-objects listed in the following table.

| **Field**                  | **data type** | **description**                                              |
| -------------------------- | ------------- | ------------------------------------------------------------ |
| headerId                   | uint32        | 消息Header ID.<br> headerId每个topic定义并且每次发送信息自增1(但不一定收到). |
| timestamp                  | string        | 日期时间 (ISO8601, UTC); YYYY-MM-DDTHH:mm:ss.ssZ(例如"2017-04-15T11:40:03.12Z”). |
| version                    | string        | 协议版本 [Major].[Minor].[Patch] (例如 1.3.2). |
| manufacturer               | string        | AGV厂商.                                     |
| serialNumber               | string        | AGV序列号.                                     |
| **typeSpecification**      | JSON-object   | 这些参数通常指定AGV的类和功能. |
| **physicalParameters**     | JSON-object   | 这些参数指定AGV的基本物理属性. |
| **protocolLimits**         | JSON-object   | 标识符,数组,字符串和类似的 在MQTT通信中的限制. |
| **protocolFeatures**       | JSON-object   | 支持的功能 of VDA5050 protocol.                      |
| **agvGeometry**            | JSON-object   | AGV几何学的详细定义.                         |
| **loadSpecification**      | JSON-object   | 负载功能的抽象规范.                 |
| **localizationParameters** | JSON-object   | 定位的详细规范.                      |

#### typeSpecification

This JSON object 描述AGV类型的一般特性.

| **Field**           | **data type**   | **description** |
|---------------------|-----------------|-----------------|
| seriesName          | string          | 制造商指定的通用系列名称.任意文本 |
| *seriesDescription* | string          | AGV类型系列的可读描述.任意文本    |
| agvKinematic        | string          | 简化的AGV运动学类型的描述.<br/> [DIFF, OMNI, THREEWHEEL]<br/>DIFF: 差速器驱动器<br/>OMNI: 全向车辆<br/>THREEWHEEL: 三轮驱动的车辆 或者 具有类似运动学的车辆 |
| agvClass            | string          | 简化的AGV类描述.<br/>[FORKLIFT, CONVEYOR, TUGGER, CARRIER]<br/>FORKLIFT: 叉车.<br/>CONVEYOR: 带有输送机的AGV.</br>TUGGER: 拖车.<br/>CARRIER: 货物 carrier 有或者没有顶升机构. |
| maxLoadMass         | float64         | [kg], 最大负荷质量. |
| localizationTypes   | Array of String  | 定位类型的简化描述.<br/>示例值:<br/>NATURAL: 自然地标;<br/>REFLECTOR: 激光反射器;<br/>RFID: RFID-tags;<br/>DMC: data matrix code;<br/>SPOT: magnetic spots;<br/>GRID: magnetic grid.<br/>
| navigationTypes     | Array of String | AGV支持的路径规划类型列表,按优先级排序.<br/>示例值:<br/>PHYSICAL_LINE_GUIDED: 没有路径规划, AGV跟踪物理安装的路径.<br/>VIRTUAL_LINE_GUIDED: AGV行走固定（虚拟）路径.<br/>AUTONOMOUS: AGV自动规划其路径.|

#### physicalParameters

This JSON-object describes physical properties of the AGV.

| **Field**       | **data type** | **description**                                       |
|-----------------|---------------|-------------------------------------------------------|
| speedMin        | float64       | [m/s] AGV的最小控制连续速度.  |
| speedMax        | float64       | [m/s] AGV的最大速度.                        |
| accelerationMax | float64       | [m/s²] 最大负载下的最大加速度.         |
| decelerationMax | float64       | [m/s²] 最大负载下的最大减速.         |
| heightMin       | float64       | [m] AGV的最小高度.                             |
| heightMax       | float64       | [m] AGV的最大高度.                             |
| width           | float64       | [m] AGV的宽度.                                      |
| length          | float64       | [m] AGV的长度.                                     |

#### protocolLimits

This JSON-object describes the protocol limitations of the AGV.
If a parameter is not defined or set to zero then there is no explicit limit for this parameter.

| **Field**                     | **data type** | **description**                             |
|-------------------------------|---------------|---------------------------------------------|
| **maxStringLens** {           | JSON-object   | 字符串的最大长度.                 |
| &emsp;*msgLen*                      | uint32        | 最大MQTT消息长度                 |
| &emsp;*topicSerialLen*              | uint32        | MQTT-Topics中serial-number的最大长度.<br/><br/>受影响的参数:<br/>task.serialNumber<br/>instantactions.serialNumber<br/>state.SerialNumber<br/>visualization.serialNumber<br/>connection.serialNumber   |
| &emsp;*topicElemLen*                | uint32        | MQTT-Topics中所有其他部分的最大长度.<br/><br/>受影响的参数:<br/>task.timestamp<br/>task.version<br/>task.manufacturer<br/>instantactions.timestamp<br/>instantactions.version<br/>instantactions.manufacturer<br/>state.timestamp<br/>state.version<br/>state.manufacturer<br/>visualization.timestamp<br/>visualization.version<br/>visualization.manufacturer<br/>connection.timestamp<br/>connection.version<br/>connection.manufacturer |
| &emsp;*idLen*                       | uint32        | ID-Strings的最大长度.<br/><br/>受影响的参数:<br/>task.orderId<br/>task.zoneSetId<br/>point.nodeId<br/>nodePosition.mapId<br/>action.actionId<br/>segment.edgeId<br/>segment.startNodeId<br/>segment.endNodeId |
| &emsp;*idNumericalOnly*             | boolean          | If "true" ID-strings 需要仅包含数字. |
| &emsp;*enumLen*                     | uint32        | 最大长度ENUM- and Key-Strings.<br/><br/>受影响的参数:<br/>action.actionType action.blockingType<br/>segment.direction<br/>actionParameter.key<br/>state.operatingMode<br/>load.loadPosition<br/>load.loadType<br/>actionstate.actionStatus<br/>error.errorType<br/>error.errorLevel<br/>errorReference.referenceKey<br/>info.infoType<br/>info.infoLevel<br/>safetyState.eStop<br/>connection.connectionState                                               |
| &emsp;*loadIdLen*                   | uint32        | 最大长度loadId Strings |
| }                             |               |                                  |
| **maxArrayLens** {            | JSON-object   | 最大数组长度.                                 |
| &emsp;*task.points*                 | uint32        | AGV每个任务可处理的的最大点数.  |
| &emsp;*task.segments*                 | uint32        | AGV每个任务可处理的的最大片段数.  |
| &emsp;*point.actions*                | uint32        | AGV每个点可处理的的最大actions数. |
| &emsp;*segment.actions*                | uint32        | AGV每个片段可处理的的最大actions数. |
| &emsp;*actions.actionsParameters*   | uint32        | AGV每个操作可处理的最大参数个数. |
| &emsp;*instantactions*              | uint32        | 每条消息AGV可以处理的最大立即动作数量. |
| &emsp;*trajectory.knotVector*       | uint32        | AGV可处理的每个轨迹的最大结数knots. |
| &emsp;*trajectory.controlPoints*    | uint32        | AGV可处理的每个轨迹的最大控制点数. |
| &emsp;*state.nodeStates*            | uint32        | AGV发送的最大nodeStates数，AGV base中的最大点数. |
| &emsp;*state.edgeStates*            | uint32        | AGV发送的最大edgeStates数，AGV base中的最大片段数. |
| &emsp;*state.loads*                 | uint32        | AGV发送的最大负载对象load-objects数.                |
| &emsp;*state.actionStates*          | uint32        | AGV发送的最大actionStates数.                |
| &emsp;*state.errors*                | uint32        | AGV在一个state消息中发送的最大错误数量. |
| &emsp;*state.informations*          | uint32        | AGV在一个state消息中发送的最大informations数量.    |
| &emsp;*error.errorReferences*       | uint32        | AGV发送的每个错误的最大错误参考errorReferences数量.      |
| &emsp;*informations.infoReferences* | uint32        | AGV发送的每个信息的最大信息参考infoReferences数量. |
| }                             |               |                                                                        |
| **timing** {                  | JSON-object   | Timing information.                                            |
| &emsp;minOrderInterval              | float32       | [s], 将任务消息发送到AGV最小间隔时间.        |
| &emsp;minStateInterval              | float32       | [s], 发送state-messages的最小间隔.               |
| &emsp;*defaultStateInterval*        | float32       | [s], 发送state-messages的默认间隔, *如果未定义，则使用主文档的默认值*. |
|  &emsp;*visualizationInterval*      | float32       | [s], 用于发送可视化主题消息的默认间隔.       |
| }                             |               |                                                               |

#### agvProtocolFeatures

该JSON对象定义了由AGV支持的 操作action和参数parameter。

| **Field**    | **data type** | **description**  |
|--------------|---------------|------------------|
| **optionalParameters** [**optionalParameter**] | JSON-object数组 | 支持和/或必须的 可选参数列表.<br/>此处未列出的可选参数,假定AGV不支持这些参数. |
| {            |               |                  |
| &emsp;parameter    | string        | 可选参数的全名, 例如 "*task.points.nodePosition.allowedDeviationTheta”*.|
| &emsp;support      | enum      | 可选参数支持的类型, 以下值是可能的:<br/>SUPPORTED: 可选参数像指定的一样支持.<br/>REQUIRED: 可选 适当的AGV操作需要参数. |
| &emsp;*description*| string        | 任意形式文字: 可选参数描述, 例如:<ul><li>原因, 为什么此AGV类型需要可选参数‘direction’及其可以包含的值.</li><li>参数 ‘nodeMarker’ 必须只包含unsigned int值.</li><li>NURBS-Support 仅限于直线和圆形片段.</li>|
| }            |               |                  |
| **agvactions** [**agvaction**] | JSON-object数组 | 这个AGV支持所有带参数的动作列表. 这包括VDA5050中指定的标准操作和制造商的特定动作. |
| {            |               |                  |
| &emsp;actionType   | string        | 唯一actionType 与action.actionType一致. |
| &emsp;*actionDescription* | string  | 任意形式文字: action描述. |
| &emsp;actionscopes | array of enum  | 使用此允许的范围列表action-type.<br/><br/>INSTANT: 可用作为立即.<br/>point: 可在点上使用.<br/>segment: 可在片段上使用.<br/><br/>例如: ```["INSTANT", "NODE"]```|
| &emsp;***actionParameters** [**actionParameter**]* | JSON-object数组 | 参数列表<br/>如果未定义，则该动作没有参数 |
|&emsp;*{*     |               |                  |
|&emsp;&emsp;key     | string        | Key-String for 参数. |
|&emsp;&emsp;valueDataType | enum    | 数据类型, 可能的数据类型是: BOOL, NUMBER, INTEGER, FLOAT, STRING, OBJECT, ARRAY. |
|&emsp;&emsp;*description* | string  | 任意形式文字: 参数描述. |
|&emsp;&emsp;*isOptional*  | boolean    | "true": 可选 参数. |
|&emsp;*}*           |         |                          |
|*resultDescription* | string  | 任意形式文字: resultDescription描述. |
|*}*                 |         |                          |

### agvGeometry

此JSON对象定义了AGV的结构特性, 例如, 轮廓和轮子位置.

| **Field**                            | **data type**        | **description**                                        |
|--------------------------------------|----------------------|--------------------------------------------------------|
| ***wheelDefinitions** [**wheelDefinition**]* | JSON-object数组 | List of wheels, containing wheel-arrangement and geometry. |
| {                                    |                      |                                                        |
| &emsp;type                                 | enum                 | 车轮类型<br/>```DRIVE, CASTER, FIXED, MECANUM```.     |
| &emsp;isActiveDriven                       | boolean                 | "true": 主动驱动车轮.       |
| &emsp;isActiveSteered                      | boolean                 | "true": 车轮主动转向.    |
| &emsp;**position** {                           | JSON-object          |                                                        |
|&emsp;&emsp; x                              | float64              | [m], x位置在AGV坐标中. system          |
|&emsp;&emsp; y                              | float64              | [m], y位置在AGV坐标中. system          |
|&emsp;&emsp; *theta*                        | float64              | [rad], 固定车轮所需的AGV坐标系统中的车轮方向. |
| &emsp;}                                    |                      |                                                        |
| &emsp;diameter                             | float64              | [m], 车轮直径.                          |
| &emsp;width                                | float64              | [m], 车轮宽度.                             |
| &emsp;*centerDisplacement*                 | float64              | [m], 车轮中心到旋转点的位移 (caster wheels必要).<br/> 如果未定义参数，则假定为0.            |
| &emsp;*constraints*                        | string               | 任意形式文字: 制造商可以用来定义约束. |
| }                                    |                      |                                                        |
| ***envelopes2d** [**envelope2d**]*   | JSON-object数组  | 2D下的AGV-包络曲线列表(german: "Hüllkurven"), 例如, 机械包络在载货和不载货状态, 不同速度的安全防护. |
| {                                    |                      |                                                        |
| &emsp;set                             | string               | 包络曲线集合数量.                         |
| &emsp;**polygonPoints**  **[polygonPoint]**         | JSON-object数组  | 包络曲线 X/Y-Polygon多边形预期是封闭的，必须是非自交的. |
| &emsp;{                                    |                      |                                                        |
|&emsp;&emsp; x                              | float64              | [m], 多边形点的x位置.                        |
|&emsp;&emsp; y                              | float64              | [m], 多边形点的y位置.                        |
| &emsp;}                                    |                      |                                                        |
| &emsp;*description*                        | string               | 任意形式文字: 包络曲线集合的描述.   |
| *}*                                  |                      |                                                        |
| ***envelopes3d [envelope3d]***       | JSON-object数组  | 3D下的AGV-包络曲线列表. |
| *{*                                  |                      |                                                        |
| &emsp;set                                  | string               | 包络曲线集合数量.                         |
| &emsp;format                               | string               | 数据格式, 例如, DXF.                                |
| &emsp;***data***                           | JSON-object          | 3D-包络曲线数据, 格式以'format'指定.   |
| &emsp;*url*                                | string               | 下载3D-包络曲线数据的协议和URL定义, 例如 <ftp://xxx.yyy.com/ac4dgvhoif5tghji>. |
| &emsp;*description*                        | string               | 任意形式文字: 包络曲线集合的描述          |
| *}*                                  |                      |                                                        |

#### loadSpecification

此JSON对象定义 AGV的负载处理和负载类型.

| **Field**                        | **data type**        | **description**                                                      |
|----------------------------------|----------------------|----------------------------------------------------------------------|
| *loadPositions*         | Array of String      | 负载位置清单 / 负载处理设备.<br/>这列表包含"state.loads[].loadPosition”参数的有效值 和 对于action取货和放货的'lhd'参数.<br/>*如果此列表不存在或为空,则AGV没有负载处理设备.* |
| ***loadSets [loadSet]*** | JSON-object数组  | 可以由AGV处理的load-sets列表                     |
| {                                    |                      |                                                        |
|&emsp; setName                 | string               | load-set的唯一名称, 例如, DEFAULT, SET1, 等等.                 |
|&emsp; loadType                | string               | 负载类型, 例如, EPAL, XLT1200, 等等.                                  |
|&emsp; *loadPositions*         | Array of String      | 负载位置列表btw.负载处理设备,此load-set有效.<br/>*如果此参数不存在或为空,则此load-set对此AGV上的所有负载处理设备有效.* |
|&emsp; ***boundingBoxReference***  | JSON-object          | Bounding box reference在state-message中的loads[]中定义. |
|&emsp; ***loadDimensions***        | JSON-object          | Load dimensions在state-message中的loads[]中定义.     |
|&emsp; *maxWeight*             | float64              | [kg], 负载类型的最大重量.                                      |
|&emsp; *minLoadhandlingHeight* | float64              | [m], 最小允许的处理高度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp;  *maxLoadhandlingHeight* | float64              | [m], 最大允许的处理高度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp; *minLoadhandlingDepth*  | float64              | [m], 最小允许的深度 针对load-type and –weight<br/>参考boundingBoxReference. |
|&emsp; *maxLoadhandlingDepth*  | float64              | [m], 最大允许的深度 针对load-type 和 –weight<br/>参考boundingBoxReference. |
|&emsp; *minLoadhandlingTilt*   | float64              | [rad], 最低允许的倾斜度 针对load-type 和 –weight.            |
|&emsp; *maxLoadhandlingTilt*   | float64              | [rad], 最大允许的倾斜 针对load-type 和 –weight.            |
|&emsp; *agvSpeedLimit*         | float64              | [m/s], 最大允许速度 针对load-type 和 –weight.           |
|&emsp; *agvAccelerationLimit*  | float64              | [m/s²], 最大允许加速 针对load-type 和 –weight.   |
|&emsp; *agvDecelerationLimit*  | float64              | [m/s²], 最大允许减速 针对load-type 和 –weight.   |
|&emsp; *pickTime*              | float64              | [s], 大约. 取货时间                  |
|&emsp; *dropTime*              | float64              | [s], 大约. 放货时间                    |
|&emsp; *description*           | string               | 任意形式文字: 负载处理集合 的描述.            |
| }                       |                      |                                                           |


# <a name="Bp"></a> 7 Best practice

本节包括其他信息,有助于促进与协议逻辑同时发生的共同理解.


## <a name="Er"></a> 7.1 Error reference 

如果由于erroneous任务而发生错误, AGV需要返回一个有含义的错误在errorReference字段 (参考 [6.10.6 Implementation](#errorReferenceImpl)).
这可以包括以下信息:

- headerId
- Topic (任务 或者 立即动作)
- orderId 和 orderUpdateId如果错误是因为任务更新导致
- actionId如果错误是因为一个动作导致
- 参数列表如果是因为erroneous动作参数导致 

如果由于外部因素无法完成操作 (例如 预期位置没有负载),actionId需要referenced.



## <a name="Fop"></a> 7.2 Format of parameters 

错误，信息，操作(errors, information, action)的参数被设计为带有键值对 的JSON-Objects数组。
动作参数的"someaction”带有键值对 stationType和loadType的例子:

``
"actionParameters":[
{"key":"stationType", "value": "floor"},
{"key": "loadType", "value": "pallet_eu"}
]
``

使用这个方案的原因 "key”: "actualKey”, "value”: "actualValue” 是保持实施通用.这是在多次会议中进行了彻底而有争议的讨论.



# <a name="Glossary"></a> 8 名词Glossary 



## <a name="Definition"></a> 8.1 定义

Concept | Description 
---|---
自由导航AGV | 使用地图规划自己道路的车辆. <br> RCS仅发送启动和目标坐标.<br>车辆将其路径发送到RCS.<br>当与RCS的通信断开时, 车辆能够继续路径.<br>自由导航车辆可以被允许绕过障碍.<br>也有可能对车辆本身对接收/分配位置进行精细调整.
引导车辆 (物理或虚拟) | 车辆从RCS得到路径. <br>在RCS中进行路径的计算.<br>当与RCS的通信断开后,该车辆终止其released的点和片段 (the "base") 并且停止.<br>可以允许引导车辆绕过障碍.<br>也有可能对车辆本身对接收/分配位置进行精细调整..
中央地图 | 将在RCS中心保存的地图.<br>创建然后使用.<br> 未来的接口版本 可以将此地图转移到车辆 (例如, 针对自由导航).
