We are currently working thtough some issues with errors preventing Camel from running as a SpringBoot application. The community is working on it and we apologize for the inconvenience. Please feel free to look at https://github.com/RedHat-Healthcare/iDaaS-Connect. This solution upstream continues to run.

# iDaaS Connect HL7
This is the upstream for RedHat Healthcare's <a href="https://github.com/RedHat-Healthcare/iDaaS-Connect/tree/master/iDaaS-Connect-HL7" target="_blank">iDaaS Connect HL7</a>. iDaaS Connect HL7 is designed to simplify HL7 integration platform development and delivery.
iDAAS has several key components that provide many capabilities. iDAAS Connect is intended ONLY
to enable iDAAS connectivity. iDAAS-Connect-HL7 specifically ONLY deals with enabling
iDAAS to process the healthcare industry standard HL7 based transactions ONLY.
It will process the following HL7 messages (ADT, ORM, ORU, MFN, MDM, PHA, SCH and VXU)
from any vendor and any version of HL7 v2.

## Add-Ons
This solution contains three supporting directories. The intent of these artifacts to enable
resources to work locally: <br/>
1. platform-scripts: support running amq, amq-streams (kafka) and doing very specific things with 
Kafka such as: creating/listing and deleting topics needed for this solution
and also building and packaging the solution as well. All the scripts are named to describe their capabilities <br/>
2. platform-testdata: sample transactions to leverage for using the platform. 

## Pre-Requisites
For all iDaaS design patterns it should be assumed that you will either install as part of this effort, or have the following:

1. An existing Kafka (or some flavor of it) up and running. Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/Kafka.md)<br/>
[KafkaWindows](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/KafkaWindows.md)<br/>
No matter the platform chosen it is important to know that the Kafka out of the box implementation might require some changes depending upon your implementation needs. Here are a few we have made to ensure: <br/>
In /config/consumer.properties file we will be enhancing the property of auto.offset.reset to earliest. This is intended to enable any new system entering the group to read ALL the messages from the start. <br/>
auto.offset.reset=earliest <br/>
2. Some understanding of building, deploying Java artifacts and the commands associated. If using Maven commands then Maven would need to be intalled and runing for the environment you are using. More details about Maven can be found [here](https://maven.apache.org/install.html)<br/>
3. An internet connection with active internet connectivity, this is to ensure that if any Maven commands are
run and any libraries need to be pulled down they can.<br/>

We also leverage [Kafka Tools](https://kafkatool.com/) to help us show Kafka details and transactions; however, you can leverage
code or various other Kafka technologies ot view the topics.

# Scenario(s)
This section is intended to cover any scenarios covered within this demo.

## Basic HL7 Message Integration 
This repository follows a very common general clinical care implementation pattern. The implementation pattern involves one system sending data to 
another system via the HL7/MLLP message standard. 

|Identifier | Description |
| ------------ | ----------- |
| Healthcare Facility| MCTN |
| Sending EMR/EHR | MMS |
| HL7 Message Events | ADT (Admissions, Discharge and Transfers),ORM (Orders),ORU (Results) |
<br/>
It is important to know that for every HL7 Message Type/Event there is a specifically defined, and dedicated, HL7 socket server endpoint.

### Integration Data Flow Steps
Here is a general visual intended to show the general data flow and how the accelerator design pattern is intended to work. <br/>
 <img src="https://github.com/RedHat-Healthcare/iDAAS/blob/master/content/images/iDAAS-Platform/DataFlow-HL7.png" width="800" height="600">

1. Any external connecting system will use an HL7 client (external to this application) will connect to the specifically defined HL7
Server socket (one socket per datatype) and typically stay connected.
2. The HL7 client will send a single HL7 based transaction to the HL7 server.
3. iDAAS Connect HL7 will do the following actions:<br/>
    a. Receive the HL7 message. Internally, it will audit the data it received to a specifically defined topic.<br/>
    b. The HL7 message will then be processed to a specifically defined topic for this implementation. There is a 
    specific topic pattern -  for the facility and application each data type has a specific topic define for it.
    For example: Admissions: MCTN_MMS_ADT, Orders: MCTN_MMS_ORM, Results: MCTN_MMS_ORU, etc.. <br/>
    c. An acknowledgement will then be sent back to the hl7 client (this tells the client he can send the next message,
    if the client does not get this in a timely manner it will resend the same message again until he receives an ACK).<br/>
    d. The acknowledgement is also sent to the auditing topic location.<br/>

# Start The Engine!!!
This section covers the running of the solution. There are several options to start the Engine Up!!!

## Step 1: Kafka Server To Connect To
In order for ANY processing to occur you must have a Kafka server running that this accelerator is configured to connect to.
Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/Kafka.md)<br/>
[KafkaWindows](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/KafkaWindows.md)<br/>

## Step 2: Running the App: Maven or Code Editor
This section covers how to get the application started.

### Design Pattern/Accelerator Configuration
All iDaaS Design Pattern/Accelelrators have application.properties files to enable some level of reusability of code and simplfying configurational enhancements.<br/>
In order to run multiple iDaaS integration applications we had to ensure the internal http ports that
the application uses. In order to do this we MUST set the server.port property otherwise it defaults to port 8080 and ANY additional
components will fail to start. iDaaS Connect HL7 uses 9980. You can change this, but you will have to ensure other applications are not
using the port you specify.

```properties
server.port=9980
```
Once built you can run the solution by executing `./platform-scripts/start-solution.sh`.
The script will startup Kafka and iDAAS server.

Alternatively, if you have a running instance of Kafka, you can start a solution with:
`./platform-scripts/start-solution-with-kafka-brokers.sh --idaas.kafkaBrokers=host1:port1,host2:port2`.
The script will startup iDAAS server.

It is possible to overwrite configuration by:
1. Providing parameters via command line e.g.
`./start-solution.sh --idaas.adtPort=10009`
2. Creating an application.properties next to the idaas-connect-hl7.jar in the target directory
3. Creating a properties file in a custom location `./start-solution.sh --spring.config.location=file:./config/application.properties`

Supported properties include:
```properties
idaas.kafkaBrokers=localhost:9092 #a comma separated list of kafka brokers e.g. host1:port1,host2:port2
#ports to listen for HL7 messages on
idaas.adtPort=10001 
idaas.ormPort=10002
idaas.oruPort=10003
idaas.rdePort=10004
idaas.mfnPort=10005
idaas.mdmPort=10006
idaas.schPort=10007
idaas.vxuPort=10008
```

### Running the App: Maven
You can use Maven from the command line, you would need to go the specific directory where this code exists and has a pom.xml and then run the
command: mvn clean install

### Running the App: From Your Code Editor/IDE
You can right click on Application.java file and select Run in the src directory.

### Running the App from Command Line
You will need to have built the application into a jar.

1. You download the code
2. You build the code into a jar
3. Move the application.properties file from the \src\main\resources to where you would like it or just remember the location as it
it will be needed later
4. Make sure you have a Kafka instance up and running to process data with
5. Go to a command line and execute java -jar <jarlocation>/<jarname.jar> --spring.config.location=file: ./<directory>/application.properties`

Happy using and coding....

# Getting Involved
Here are a few ways you can get or stay involved.

## Ongoing Enhancements
We maintain all enhancements within the Git Hub portal under the projects tab, please feel free to make recommendations our goal is to continually improve.

## Defects/Bugs
All defects or bugs should be submitted through the Git Hub Portal under the issues tab



Happy using and coding....
