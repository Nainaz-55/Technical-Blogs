---
title: "PROJECT - Based on Basic authentication, Verify API key, and Javascript policy"
seoTitle: "PROJECT - Based on Basic authentication, Verify API key, and"
seoDescription: "Apigee Assignment - Based on Basic authentication, Verify API key, and Javascript policy"
datePublished: Wed Dec 25 2024 15:54:58 GMT+0000 (Coordinated Universal Time)
cuid: cm542szgz000g09jwcwivewxe
slug: project-based-on-basic-authentication-verify-api-key-and-javascript-policy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735141974251/9597db8e-db2e-4447-abd0-af74575920dd.png
tags: google-cloud, apis, api-management, apigee-assignment, javascript-policy, verify-api-key, based-on-basic-authentication, apigee-assignment-based-on-basic-authentication-verify-api-key-and-javascript-policy

---

* CREATE an API proxy with an appropriate base path.
    
* Attach a verify API key policy {the client\_id and client\_secret must be sent in the Basic Authorization encoded form from Postman}.
    
* The base64 encoded authorization should be decoded using Basic Authorization and the client\_id should be verified in Verify API key policy.
    
* In the header, the below values have to be sent:
    
    \- FirstName: {your\_firstname}  
    \- LastName: {yout\_lastname}  
    \- Operation: FirstName/LastName
    
* Attach the Javascript policy and the following has to be done:
    
    * If operation = FirstName --&gt; "Hi, {your\_firstname}" should be the response.
        
    * If the Operation = Lastname --&gt; "Hi, {your\_lastname}" should be the response.
        

## Implementation

**CREATE NEW** no target API proxy without selecting an API key security option while creating.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702892673749/18b38517-e758-4c8f-a805-37a2d2a031f6.png align="center")

The request message consists of an encoded form of client\_id and client\_secret (we will know how to get this client\_id and client\_secret later).  
To Decode it use the **Basic Authentication policy**.  
Proxyendpoint -&gt; Preflow -&gt; Request -&gt; **ADD** Basic Authentication policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702893239067/528bee89-05c1-4c4a-a126-fd061a137b21.png align="center")

The basic Authentication policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<BasicAuthentication continueOnError="false" enabled="true" name="Basic-Authentication-1">
    <DisplayName>Basic Authentication-1</DisplayName>
    <Operation>Decode</Operation>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <User ref="request.queryparam.client_id"/>
    <Password ref="request.queryparam.client_secret"/>
    <Source>request.header.Authorization</Source>
</BasicAuthentication>
```

After decoding the client\_id should be verified in the **Verify API key policy**.  
Proxyendpoint -&gt; Preflow -&gt; Request -&gt; **ADD** Verify API key policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702893522882/df8e33f5-9175-496b-b37a-7e6ae8ebdecf.png align="center")

Verify API key policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<VerifyAPIKey continueOnError="false" enabled="true" name="Verify-API-Key-1">
    <DisplayName>Verify API Key-1</DisplayName>
    <Properties/>
    <APIKey ref="request.queryparam.client_id"/>
</VerifyAPIKey>
```

ADD **Javascript policy** to operate.  
Proxyendpoint -&gt; Preflow -&gt; Request -&gt; **ADD** Javascript policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702893869268/b97d8fbd-655c-4f32-b1a3-09a511c4fe67.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702894024061/eb390c5a-fea5-4d9b-b23d-0184732f8b74.png align="center")

The Javascript policy code should look like the below.

```javascript
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Javascript continueOnError="false" enabled="true" timeLimit="200" name="JavaScript-1">
    <DisplayName>JavaScript-1</DisplayName>
    <Properties/>
    <ResourceURL>jsc://JavaScript-1.js</ResourceURL>
</Javascript>
```

JavaScript-1.js (Below the resources) code should look like below.

```javascript
 var firstname = context.getVariable("request.header.Firstname");
 var lastname = context.getVariable("request.header.Lastname");
 var operation = context.getVariable("request.header.Operation");
 
 const result = operation === 'Firstname';
 var ot;
 
 if(result){
     ot = 'Hi,' + firstname;
 }
 else{
     ot = 'Hi,' + lastname;
 }
 context.setVariable("output",ot);
```

ADD Assign Variable policy to construct the response message.  
Proxyendpoint -&gt; Postflow -&gt; response -&gt; ADD Assign Variable policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702895464740/42efa104-a57e-4c19-9568-20373ab91826.png align="center")

Assign Variable policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <Set>
        <Headers/>
        <QueryParams/>
        <FormParams/>
        <Verb>POST</Verb>
        <Payload contentType="application/text">
            "{output}"
        </Payload>
        <Path/>
    </Set>
    <AssignVariable>
        <Name>name</Name>
        <Value/>
        <Ref/>
    </AssignVariable>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

**Save** and **Deploy** the Proxy.

Create an **APP** to get the client\_id and client\_secret.

**CREATE** an API product.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702895935431/56e98916-ac95-4c4f-801b-d5f7ea89ea8c.png align="center")

Click on **ADD AN OPERATION** to add a proxy.  
Fill in the corresponding details and click on **SAVE**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896229358/1a63b887-7077-4c4a-a8b9-be06230e694c.png align="center")

Click on **SAVE** to create a proxy.  
Add **Developer**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896347913/abbc054a-29e3-460c-ae5c-3005b954c047.png align="center")

Create an **App** developer by selecting the required API product.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896408202/139d7ede-c6f7-47ca-9496-dbc36f556086.png align="center")

Fill in the corresponding details and click on **Create.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896456173/02424c6d-7800-46cf-830c-e384f2193bb5.png align="center")

After the App gets approved we will get the key(client\_id) and secret(client\_secret).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896579468/856b4fe2-9bb4-404c-ad9c-fcb25fce5f74.png align="center")

Encode the client\_id and client\_secret by using the below website. By pasting client\_id and client\_secret in colen separated formate.  
[https://www.base64encode.org/](https://www.base64encode.org/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702896776088/41486608-beaa-45ae-996c-5ba9b1ca7ec7.png align="center")

Send the request message: https://yourhost/proxy\_basepath, add the below values in the header:  
Authorization = Basic encoded\_id\_and\_secret  
FirstName: {your\_firstname}  
LastName: {yout\_lastname}  
Operation: FirstName/LastName.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702897342782/930f8f7a-622a-44d4-b197-2f638662d8c5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702897307960/967d29d2-57bd-4165-b592-159edc9f9676.png align="center")