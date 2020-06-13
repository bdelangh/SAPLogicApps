# Receive RFC, BAPI and Idocs from SAP
In this example we're implementing the basic setup for a logic app to receive a BAPI, RFC or Idoc from an SAP System.
This means we need to setup a logic app which registers as an external program on the SAP Gateway. In SAP we need to setup a RFC Destination.

## Logic App SetUp

In the logic app we use the 'Receive from SAP' as action trigger.\
In the settings of this action, you enter :
- GatewayHost = hostname where the SAP Gateway is running
- GatewayService = port number of the SAP Gateway, typically this is ```33<SAP SystemId>```, eg ```3300```
- ProgramId = this is the program id that the Logic App Gateway will register at the SAP Gateway. You can choose this name. The program id will also be used in the RFC Connection which will be used by the SAP system to call the logic app.
Degree of Parallellism = nr of times the programId will be registered at the SAP Gateway. Basically this represents the number of parallel calls the logicApp will be able to handle.

<img src='Images\receive\logicApp.JPG'>

Upon save the Logic App Gateway will register the ProgramId at the SAP Gateway. You can check this in the SAP Gateway monitor: Transaction SMGW, select Goto-> logged on clients. The programId should appear here as TP Name.

<img src='Images\receive\smgw.jpg'>

## RFC Destination Setup
The next step is to create a RFC destination in transaction sm59.
The RFC destination needs to be of type T - TCP/IP Connection and activation type 'Registered Server Program'.
As program id of the registered server program you need to enter the program id entered in the logic app.

<img src='Images\receive\sm59.jpg'>

Use the ```connection test```button to see if the connection is working.

<img src='Images\receive\sm59ConnectionTest.jpg'>

Note :
* If you get a connection error, eg timeout, you can check if you're using the correct gateway name in the logic app. The gateway host is mentioned in the error.

<img src='Images\receive\sm59Error.jpg'>

* In case of a connection error and the gateway is correct, you need to check the Gateway security settings. In my case these settings only allowed local programs to register at the Gateway security.
Gateway Monitor (Transaction smgw) -> Goto -> Expert Functions -> External Security --> Maintain ACL files.
Here I needed to change the ```Reginfo.dat file```.\
See [Security Settings for Gateway - Making Security Settings for External Programs](https://help.sap.com/viewer/62b4de4187cb43668d15dac48fc00732/7.3.20/en-US/48b2096b7895307be10000000a42189b.html).

<img src='Images\receive\securitySettings.jpg'>

```reginfo.dat```
```
#VERSION=2

 P TP=* HOST=* ACCESS=*
``` 

These settings are not to be used in a production environment since it basically allows any program to register at the Gateway.

Now you can execute a first test. Use the Function Builder to execute a RFC to the logicApp.
Enter the RFC destination.

<img src='Images\receive\RFCcall.jpg'>

Upon successfull exection of the RFC, you can go to the Runs History of the logicApp.
Here you can see the xml contents of the RFC call.

<img src='Images\receive\RFCxml.jpg'>
 
Now the SAP call is successfully working you can implement your logicApp.
See [RFC Implementation](RFCImplementation).\