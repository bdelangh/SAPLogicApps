# Execute RFC Call on SAP
In this document we'll create a logicApp which executes an RFC Call on SAP.

## Overview
In the example we'll want an external system, like an IoT Device, to create an 'Alert' in a SAP System by using a logic App.
The external system will execute an HTTP Post with the alert details on the logica App and then use the SAP Adapter to crete the alert in SAP.

## Setup
<!-- LogicApp : CreateIoTAlert-->
The logicApp will use the "On-Premises data gateway". Since my SAP system is running on Azure, I installed, despite the name, the Gateway on a vm in Azure. I also installed the SAP .Net Connector on this vm.
For installation see :
- [Install on-premises data gateway for Azure LogicApps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install)
- [SAP client library prerequisites](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-using-sap-connector#sap-client-library-prerequisites)

### SAP RFC
To create the IoT Alert I created a 'Z-function' in the SAP System. The signature of the system is mentioned beneath.

<img src='Images\send\RFCSignature.jpg'>

If you're interested in the code of this function module, please have a look at my [ABAP Repository](https://github.com/bdelangh/sap_iot_abap).
You can clone the code into your system using [ABAPGit](https://docs.abapgit.org/).

### LogicApp
#### HTTP Request
Since the external system will use HTTP post, the first step in our logic App is a HTTP request trigger.

<img src='Images\send\HTTPPost.jpg'>

The URL which you'll need to use to execute the HTTP Post is generated upon save of the logicApp.

#### Send to SAP
We'll assume the external system posts the xml document in the needed format.
For more info on how to generate this xml, see [Generate Schemas for Artifacts in SAP](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-using-sap-connector#generate-schemas-for-artifacts-in-sap).

<i>Example XML</i>
```xml
<?xml version="1.0" encoding="utf-8"?>
<Z_IOTALERT_CREATE_PRIO xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
	<SENSOR>LogicApp1</SENSOR>
	<PRIORITY>HI</PRIORITY>
	<VALUE>99.9</VALUE>
</Z_IOTALERT_CREATE_PRIO>
```

The next step in the LogicApp adapter is then just to send this xml document to the SAP System.
Behind the screens the xml document is first send to the Logic App gateway which will translate the RFC XML to the native RFC protocol which the SAP System can understand.

In the SAP Send action we need to enter the necessary information to connect to the SAP System. This information will be saved in an API Connection.

<i>API Connection</i>
<img src='Images\send\APIConnection.jpg'>

I filled in the following information :
- Client = SAP Client
- Authentication Type =  Basic, for userid and password
- SAP Username, typically this is a RFC (non dialog) user
- SAP Password
- Logon Type = Application Server
- AS Host = Host Name or IP address of the Application Server
- AS Service = Port Nr of of the Gateway, typically 33'<SAP System Nr>'. Form more info on SAP port numbers see [TCP/IP Ports of All SAP Products](https://help.sap.com/viewer/ports)
- AS System Number = SAP System Number

When the connection is setup successfully, you can select the RFC you want to call from the drop down list.

<img src='Images\send\RFCPicker.jpg'>

The ```Input Message```of the action is the body of the HTTP request.

The '```Send Message to SAP```' Action now looks as follows :

<img src='Images\send\RFCSend.jpg'>

#### HTTP Response
For now we'll just assume each call will be successfull and return a successfull HTTP Response with the Response XML in the body.
In real life you'd need to catch errors from the ```send message action``` and also check the 'RETURN' table from the RFC itself to see if there are any Error messages in this table.

<img src='Images\send\HTTPResponse.jpg'>

### Testing
For testing, I'll be using postman.
The URL to post to can be found in the HTTP Request trigger action.

<img src='Images\send\PostRequest.jpg'>

Upon successfull execution, you'll get the id of the created alert in SAP.

<i>Post Response</i>
```xml
<Z_IOTALERT_CREATE_PRIOResponse xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
    <RET_ALERTID>280</RET_ALERTID>
    <RETURN>
        <BAPIRET2 xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc/">
            <TYPE>S</TYPE>
            <ID>ZIOT</ID>
            <NUMBER>000</NUMBER>
            <MESSAGE>Alert 280 successfully created</MESSAGE>
            <LOG_NO></LOG_NO>
            <LOG_MSG_NO>000000</LOG_MSG_NO>
            <MESSAGE_V1>280</MESSAGE_V1>
            <MESSAGE_V2></MESSAGE_V2>
            <MESSAGE_V3></MESSAGE_V3>
            <MESSAGE_V4></MESSAGE_V4>
            <PARAMETER></PARAMETER>
            <ROW>0</ROW>
            <FIELD></FIELD>
            <SYSTEM></SYSTEM>
        </BAPIRET2>
    </RETURN>
</Z_IOTALERT_CREATE_PRIOResponse>
```

We've completed our first LogicApp.


### Documentation Links
- [Logic Apps using SAP Connector](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-using-sap-connector)
- [Install on-premises data gateway for Azure LogicApps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install)
- [SAP client library prerequisites](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-using-sap-connector#sap-client-library-prerequisites)
- [Generate Schemas for Artifacts in SAP](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-using-sap-connector#generate-schemas-for-artifacts-in-sap)
- [TCP/IP Ports of All SAP Products](https://help.sap.com/viewer/ports)

### ToDo
- Error Handling
- Return table Handling






