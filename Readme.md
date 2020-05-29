# Receive RFC, BAPI and Idocs from SAP
In this example we're implementing a basic logic app to receive an RFC call.

In the logic app we use the 'Receive from SAP' as action trigger.\
In the settings of this action, you need to enter :
* GatewayHost = hostname where the SAP Gateway is running
* GatewayService = port number of the SAP Gateway, typically this is 33<SAP SystemId>, eg 3300
* ProgramId = this is the program id that the Logic App Gateway will register at the SAP Gateway. You can choose this name. The program id will also be used in the RFC Connection which will be used by the SAP system to call the logic app.
Degree of Parallellism = nr of times the programId will be registered at the SAP Gateway. Basically this represents the number of parallel calls the logicApp will be able to handle.

<img src='Images\receive\logicApp.jpg'>

Upon save the Logic App Gateway will register the ProgramId at the SAP Gateway. You can check this in the SAP Gateway monitor: Transaction SMGW, select Goto-> logged on clients. The programId should appear here as TP Name.

<img src='Images\receive\smgw.jpg'>

The next step is to create a RFC destination in transaction sm59.
The RFC destination needs to be of type T - TCP/IP Connection and activation type 'Registered Server Program'.
As program id of the registered server program you need to enter the program id entered in the logic app.

<img src='Images\receive\sm59.jpg'>

You use the connection test to see if the connection is working.

<img src='Images\receive\sm59ConnectionTest.jpg'>

Note :
* If you get a connection error, eg timeout, you can check if you're using the correct gateway name in the logic app. The gateway host is mentioned in this error.

<img src='Images\receive\sm59Error.jpg'>

* In case of a connection error and the gateway is correct, you need to check the Gateway security settings. In my case these settings only allowed local programs to register at the Gateway security.
Gateway Monitor (Transaction smgw) -> Goto -> Expert Functions -> External Security --> Maintain ACL files
Here I needed to change the Reginfo.dat file. (see also (Security Settings for Gateway - Making Security Settings for External Programs)[https://help.sap.com/viewer/62b4de4187cb43668d15dac48fc00732/7.3.20/en-US/48b2096b7895307be10000000a42189b.html].

<img src='Images\receive\security.jpg'>

reginfo.dat

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


## Usefull Links :
https://www.linkedin.com/pulse/troubleshooting-sap-configuration-azure-logic-app-trigger-david-burg/

Other related information on the SAP trigger:
https://www.linkedin.com/pulse/how-does-azure-logic-app-sap-connector-trigger-works-david-burg/
https://www.linkedin.com/pulse/maximizing-throughput-calling-azure-logic-apps-from-sap-david-burg/
https://www.linkedin.com/pulse/parallel-execution-sap-calling-azure-logic-apps-david-burg/
https://www.linkedin.com/pulse/sap-sending-idoc-azure-logic-apps-reply-channel-failure-david-burg/
https://www.linkedin.com/pulse/sap-configuration-test-sending-idocs-azure-logic-apps-david-burg/
