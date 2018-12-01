# Configuring Salesforce Inbound Endpoint

The Salesforce streaming inbound endpoint allows you to perform various Salesforce streaming data via WSO2 ESB.
1.The Salesforce streaming API receives notifications for changes to Salesforce data that match a Salesforce Object Query Language (SOQL) query you define, in a secure and scalable way. For more information see [Salesforce streaming documentation](https://developer.salesforce.com/docs/atlas.en-us.202.0.api_streaming.meta/api_streaming/quick_start_workbench.htm .

2.[Platform events](https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/platform_events_intro.htm)



```
Reliable message delivery is only available in Salesforce API version 37.0 and later.
```

## Creating a PushTopic 
you need to first create a PushTopic that contains an SOQL query.
Go to the developer console of your Salesforce account and click on Debug->Open Execute Anonymous Window. Add the following entry in the Enter Apex Code window. 

```
PushTopic pushTopic = new PushTopic();
pushTopic.Name = 'InvoiceStatementUpdates';
pushTopic.Query = 'SELECT Id, Name, wso2__Status__c, wso2__Description__c FROM InvoiceStatement__c';
pushTopic.ApiVersion = 35.0;
pushTopic.NotifyForOperationCreate = true;
pushTopic.NotifyForOperationUpdate = true;
pushTopic.NotifyForOperationUndelete = true;
pushTopic.NotifyForOperationDelete = true;
pushTopic.NotifyForFields = 'Referenced';
insert pushTopic;
```
Click Execute.

## Retrieving the Account information 
WSO2 ESB Salesforce inbound endpoint acts as a message consumer. It creates a connection to the Salesforce account, consumes the Salesforce data and injects the data to the ESB sequence.
Now that you have configured the Salesforce streaming inbound endpoint, use the following ESB inbound endpoint configuration to retrieve account details from your Salesforce account.

```
<?xml version="1.0" encoding="UTF-8"?>
<inboundEndpoint xmlns="http://ws.apache.org/ns/synapse"
                 name="SaleforceInboundEP"
                 sequence="test"
                 onError="fault"
                 class="org.wso2.carbon.inbound.salesforce.poll.SalesforceStreamData"
                 suspend="false">
   <parameters>
      <parameter name="inbound.behavior">polling</parameter>
      <parameter name="interval">100</parameter>
      <parameter name="sequential">true</parameter>
      <parameter name="coordination">true</parameter>
      <parameter name="connection.salesforce.replay">false</parameter>
      <parameter name="connection.salesforce.EventIDStoredFilePath">/Users/nalaka/Desktop/a.txt</parameter>
      <parameter name="connection.salesforce.packageVersion">37.0</parameter>
      <parameter name="connection.salesforce.salesforceObject">/topic/InvoiceStatementUpdates</parameter>
      <parameter name="connection.salesforce.loginEndpoint">https://login.salesforce.com</parameter>
      <parameter name="connection.salesforce.userName">MyUsername</parameter>
      <parameter name="connection.salesforce.password">MyPassword</parameter>
      <parameter name="connection.salesforce.waitTime">5000</parameter>
      <parameter name="connection.salesforce.connectionTimeout">20000</parameter>
      <parameter name="connection.salesforce.soapApiVersion">22.0</parameter>
   </parameters>
</inboundEndpoint>
```
* connection.salesforce.replay: replay enable or disable. If this enabled read from event id stored in registry DB or from the text file stored in the local machine.
* connection.salesforce.EventIDStoredFilePath: leave this property to empty if need to replay from last event id stored in registry DB (property-“eventID” resource path:connector/salesforce/event)
* connection.salesforce.EventIDStoredFilePath: when replay enabled and specify a text file to replay onwards the stored event id from the file specified here.
* connection.salesforce.packageVersion: The version of the connection.salesforce.packageName.
* connection.salesforce.salesforceObject : The name of the push topic or platform event that is added to the Salesforce account.
* connection.salesforce.loginEndpoint: The endpoint for the Salesforce account.
* connection.salesforce.userName:  The user name for accessing the Salesforce account.
* connection.salesforce.password: The password provided here is a concatenation of the user password and the security token provided by Salesforce. For information on creating a security token in Salesforce, see https://help.salesforce.com/articleView?id=user_security_token.htm&type=5.
* connection.salesforce.connectionTimeout: The time to connect to the Salesforce account (default : 20 * 1000 mili seconds).
* connection.salesforce.waitTime: The time to wait when connecting to the client (default : 5 * 1000 ms).
* connection.salesforce.soapApiVersion: The version of the salesforce soap API.



