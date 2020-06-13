# Implement an RFC using Azure Logic App

## Prerequisites
- RFC Destination to Azure Logic App is setup (see[Receive RFC](/RFCReceive.md))
- Skeleton RFC is setup in SAP

## SetUp
In this example, my SAP system will call an external system to create a Alert. The external system will respond with the AlertId.
The signature of my RFC call is ass follows :

<img src='Images\receive\RFCSignature.jpg'>

Our Logic app will have the 'SAP - When a message is received' trigger as a first step. For setup, see[Receive RFC](/RFCReceive.md).

<img src='Images\receive\receiveAlert.jpg>

The xml received by SAP looks as follows :

```xml
<Z_IOTALERT_CREATE_PRIO xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
	<PRIORITY>H</PRIORITY>
	<SENSOR>TEST123</SENSOR>
	<VALUE>90.00</VALUE>
	<RETURN></RETURN>
</Z_IOTALERT_CREATE_PRIO>
```
Here you recognise the input parameters from the Z_IOTALERT_CREATE_PRIO function module.

The logic app can then use this input to call the external system. In our example the external system will respond with the id of the created alert. This id then needs to passed on as the respose to our RFC call.
For this we need to create an xml document and indicate this is the response.

The response xml looks as follows. Suppose the generated alertId is ```99```.

```xml
<Z_IOTALERT_CREATE_PRIOResponse xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
	<RET_ALERTID>99</RET_ALERTID>
	<RETURN></RETURN>
</Z_IOTALERT_CREATE_PRIOResponse>
```

At the moment I just hardcoded this xml. (I know shame on me ...)

<img src="Images\receive\composeXML.jpg>

To pass this as a response to the RFC, you need to use the 'Reponse' action. (This is the same action you'd use when implementing a http request- response).
I set the output of the compose step, which is the xmkl string, as the Response Body. 

<img src="Images\receive\responseXML.jpg>

You can now test the RFC call in ```se37```.

Upon successfull execution the alertId is received from the LogicApp.

<img src="Images\receive\RFCResponse.jpg>


## ToDo
- Implement RFC? Write Alert to SQL Server? 
- Experiment with the Status Code of the Response step. cfr http 200 success, 400 response
- Use Return table to return errors


