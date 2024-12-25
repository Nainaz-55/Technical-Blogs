---
title: "Service Callout Policy"
seoTitle: "Service Callout Policy"
seoDescription: "Service Callout Policy"
datePublished: Wed Dec 25 2024 14:55:43 GMT+0000 (Coordinated Universal Time)
cuid: cm540oso6000g0ala38gp3qkl
slug: service-callout-policy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735138480914/995d8958-8e51-4b98-b857-52a6efe4bdad.png
tags: google-cloud, apis, api-management, service-callout-policy

---

Service callout policy can call other services from API proxy.  
These services can be external or internal.  
Internal means, the API proxy and services present in the same organization and environment.  
This supports requests over http and https.

This callout policy makes use of the other 2 policies whenever required.  
\- Assign Message Policy: It creates the request message to be sent to Target, and  
also populates HTTPs headers, query parameters, and HTTP verbs.  
\- Extract message policy: Parses the response and extracts the specific data.

There are 2 ways in which the service callout gets the request message to pass to the target endpoint  
\- Using the Assign message policy create a request message and the call service callout policy.  
\- Can create the request message in the service callout policy itself.

There are 3 ways we can use the service callout policy:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701594026567/3882c4ec-8066-4318-8f1d-80b5d6cab88f.jpeg align="left")

**HTTP target:** we need to mention the target endpoint here. This option is used when calling the external service  
**Proxy chaining:** This option is used when calling the internal API proxy. here we will give the proxy name  
**path chaining:** here we won’t give the proxy name instead we give the proxy path. Bcz if the proxy name is not constant, it may change in the future but the path is always constant.

---

## Service callout - HTTP method

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701594494799/1261fb9e-a963-496f-a2a0-92088c840c81.jpeg align="center")

In this example, receiving the request from the client and it hits the API proxy, then this API proxy will use a service callout policy to call the external endpoint.  
HTTP external endpoint converts the JSON to XML format. The callout policy sends the JSON message to the external endpoint, and that JSON message is constructed in the service callout policy. then that JSON message reaches that endpoint and gets the XML message back to the service callout policy stored in the variable. that XML message is attached to the request using the assign message policy(copy the response of the callout policy to request payload). send to target, then that XML message is sent back to the client  
Note: see here in API proxy, service callout policy, and assign message policy in apigee. others Target server and HTTP external endpoint are outside the apigee. So we will use the HTTP option in the service callout.

Implent it in the Apigge portal: Here we will use two endpoints one for HTTP external endpoint, and another for the target endpoint.

Scenario 2: to understand how service callout works with HTTP method 1  
here we will use two endpoints:  
External endpoint: [https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights](https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights) (it will give flight data in JSON format).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701601108147/f084adcd-8f3d-443c-a96c-c6e947d88f92.png align="center")

Target endpoint: [http://httpbin.org/get](http://httpbin.org/get) (it will give the data that it will receive with some more information).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701601188324/bd440aad-f903-413c-b920-05f0f7b61329.png align="center")

in response, the client will get the length of the JSON data sent by service callout.

Create a reverse proxy with the target endpoint.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701601256661/38d8aa23-acb5-4768-8961-f7308a7dc6f1.png align="center")

Next - passthrough - eval - create and deploy - edit proxy - develop.  
The policy will get called when we get a request from the client.  
proxy endpoint - preflow - request - service callout policy.  
Service callout policy uses HTTP and HTTP target ([https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights](https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights))

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701601613459/14814ea0-37ba-4f77-aef1-5c34befb202e.png align="center")

The service callout policy code should be like below. and save

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ServiceCallout continueOnError="false" enabled="true" name="Service-Callout-1">
    <DisplayName>Service Callout-1</DisplayName>
    <Properties/>
    <Request clearPayload="true" variable="myRequest">
        <set>
        <!-- add header to identify we are getting json message as response from this policy-->
        <Headers>
            <Header name="Accept">application/json</Header>
        </Headers>
        <!-- we are getting the data so GET method -->
        <Verb>GET</Verb>
        </set>
        <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    </Request>
    <!-- whatever the response we will receiv fro this policy. it will get stored in calloutResponse-->
    <Response>calloutResponse</Response>
    <HTTPTargetConnection>
        <Properties/>
        <URL>https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights</URL>
    </HTTPTargetConnection>
</ServiceCallout>
```

We need to fetch the data from the response of the policy and add it to the request before sending it to target.  
Target endpoint - preflow - request - Assign message policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701602188499/bda87ced-bd22-4438-901f-3a6b8a3efdea.png align="center")

Assign message policy code should be like the below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <!-- source: data we are getting form calloutResponse -->
    <Copy source="calloutResponse">
        <!-- we have to copy to playload-->
        <Payload>true</Payload>
    </Copy>
    <Set>
        <!-- we are adding data to request payload and getting data back -->
        <Verb>GET</Verb>
    </Set>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- responce from policy added to request payload-->
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Save and deploy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701603436612/fc5a0edc-6975-40e8-9908-e773810c328c.png align="center")

---

## Service Callout - Proxy Chaining

In proxy chaining, we can call the API proxy which is in the same organization and environment as the calling API proxy.  
Here is the syntax to call the internal API proxy

```xml
<LocalTargetConnenction>
    <APIProxy>LocalEndpoint1</APIProxy><!-- API proxy name in the 
     same environment-->
    <ProxyEndpoint>localProxy1</ProxyEndpoint><!--it is the proxy
     endpoint of the API proxy specified in the<APIProxy>-->
</LocalTargetConnenction>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701611099674/10f6117c-6b3d-4fa5-9791-9bf227b161e0.jpeg align="center")

The client sent a request.  
Request first hit to API proxy endpoint. then we will use the assign msg policy to create a JSON message and msg to request. this request will be input into the service callout policy. callout policy calls the Internal endpoint in the APIgee network by sending JSON message to it. This internal endpoint proxy converts JSON to XML messages and sends XML msg back to service callout policy. once it sends an XML message stored into one variable, that XML msg will be added to the request payload using Assign message policy before sending it to the target server(outside of the APIgee network). this endpoint sends the XML message in response back to the client.

Implemention

Create a Local endpoint - no target API proxy(That will convert JSON message to XML msg).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701612321315/0e65efb3-7aeb-4558-b291-b3f7581fb4cd.png align="center")

next - pass through - eval - create and deploy - edit proxy - develop.  
Proxy endpoint - preflow - request - JSON to XML policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701612534655/339e20d5-3d2e-47bc-8809-1dc2453ac10f.png align="center")

when a request comes it replies back to the service callout policy.

JSON to XML policy code looks like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<JSONToXML continueOnError="false" enabled="true" name="JSON-to-XML-1">
    <DisplayName>JSON to XML-1</DisplayName>
    <Properties/>
    <Options>
        <NullValue>NULL</NullValue>
        <NamespaceBlockName>#namespaces</NamespaceBlockName>
        <DefaultNamespaceNodeName>$default</DefaultNamespaceNodeName>
        <NamespaceSeparator>:</NamespaceSeparator>
        <TextNodeName>#text</TextNodeName>
        <AttributeBlockName>#attrs</AttributeBlockName>
        <AttributePrefix>@</AttributePrefix>
        <InvalidCharsReplacement>_</InvalidCharsReplacement>
        <ObjectRootElementName>Root</ObjectRootElementName>
        <ArrayRootElementName>Array</ArrayRootElementName>
        <ArrayItemElementName>Item</ArrayItemElementName>
    </Options>
    <OutputVariable>request</OutputVariable>
    <Source>request</Source>
</JSONToXML>
```

Save and deploy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701612979988/78a33b1f-8502-4df0-af59-640468f0703a.png align="center")

The local endpoint working well. It converts JSON msg to XML msg successfully.

Create reverse API proxy  
target endpoint: [https://httpbin.org/post](https://httpbin.org/post)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701613164498/9326012b-ed2f-4fb4-a983-d9a1ff410fb8.png align="center")

next - pass through - eval - create and deploy - edit proxy - develop.  
Adding service callout policy. that will call the local endpoint to convert JSON msg to XML msg.  
proxy endpoint - preflow - request - service callout policy(select proxy chaining option - proxy name\[loacalendpointjsontoxml\] - proxy endpoint\[default\])

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701613502978/fae47ab7-c1c2-4b1b-9103-9282c04c6247.png align="center")

proxy endpoint name is the default.  
why because - open the "loacalendpointjsontoxml" proxy in that proxy endpoint name is default by default (we can change that name as well, if changed update it in the service callout policy as well)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701613753328/88fdae69-e4c9-43f8-b382-d236ebd62fc9.png align="center")

before this, we need to add an assign message policy to create JSON messages.  
Add and move it to first.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701614154900/6883f615-0625-4f11-aae6-969fdcafd8af.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701614165710/7ed74cbd-17a1-4d7f-bf9b-00b8186e90ba.png align="center")

Till now we called the service callout. That will call the local endpoint(convert JSON to XML msg).  
Assign message policy 1 code should be like the below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
   
    <Set>
        
        <!-- <Verb>GET</Verb> -->
        <Payload contentType="application/json">
            {
        "ID": 13,
        "code": "INA629",
        "price": 5600,
        "departureDate": "26 Feb 2021",
        "origin": "DEL",
        "destination": "BLR",
        "emptySeats": 120,
        "plane": {
            "ptype": "Boeing 737",
            "totalSeats": 180
        }
    }
        </Payload>
        <Path/>
    </Set>
    
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- it should be assign to new request, so createNew = "true"-->
    <!--"myRequest" myreuest is from service callout policy code variable = "myrequest"-->
    <!-- whaterver payload created here is assign to myRequest variable/new_request-->
    <AssignTo createNew="true" transport="http" type="request">myRequest</AssignTo>
</AssignMessage>
```

service-policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ServiceCallout continueOnError="false" enabled="true" name="Service-Callout-1">
    <DisplayName>Service Callout-1</DisplayName>
    <Properties/>
    <!-- myrequest(new request has a payload), this request call "loacalendpointjsontoxml"
    endpoint, it will convert json to xml message, 
    that XML msg comes and stor in "calloutResponse" variable-->
    <Request clearPayload="true" variable="myRequest">
        <Set>
            <!-- post json msg to loacl endpoint -->
            <Verb>POST</Verb>
        </Set>
        <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    </Request>
    <Response>calloutResponse</Response>
    <!-- the data present in "calloutResponse" variable is send it to target endpont-->
    <LocalTargetConnection>
        <APIProxy>loacalendpointjsontoxml</APIProxy>
        <ProxyEndpoint>default</ProxyEndpoint>
    </LocalTargetConnection>
</ServiceCallout>
```

Then we need to add the returned data from "loacalendpointjsontoxml" endpoint to the request, before sending it to the target.  
target endpoint - preflow - request - assign message policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701615189143/a260e580-d367-4c74-91fb-a07f870f8489.png align="center")

Assign message policy 2 code should be like the below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-2">
    <DisplayName>Assign Message-2</DisplayName>
    <Properties/>
    <!-- here source , is response form the local endpoint proxy. ie, 
    responbse is stored in "calloutResponse" variable.
    Reade that response copied to request payload-->
    <Copy source="calloutResponse">
        <Payload>true</Payload>
    </Copy>
    <Set>
        <!-- posting the XML msg to request-->
        <Verb>POST</Verb>
    </Set>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- extracting the response and copied to payload that will go as a request to the target endpoint-->
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Save and deploy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701616861102/012d49a0-fecf-4615-8eed-fba87d549a2d.png align="center")

By this, we can conclude that JSON data is got converted to XML and response back to the client.

---

## Service callout - Path Chaining

In path chaining, we can call the API proxies which are in the same organization and environment as the calling API Proxy  
Here is the syntax to call the Internal API proxy.

```xml
<LocalTargetConnenction>
    <Path>/localendpoint1</Path>
    <!-- A path to the endpoint that is being targeted-->
</LocalTargetConnenction>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701682999369/2763fcf5-3fe3-4b40-bfc4-bd94b49de555.jpeg align="center")

It is also the same scenario. but we won't specify the API proxy name here. Instead, we specify the proxy endpoint name(the name is default by default).