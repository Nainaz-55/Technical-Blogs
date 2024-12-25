---
title: "Shared Flows"
seoTitle: "Shared Flows and Flow callout policy"
seoDescription: "Shared Flows and Flow callout policy"
datePublished: Wed Dec 25 2024 14:35:48 GMT+0000 (Coordinated Universal Time)
cuid: cm53zz6p5000409jic7z0grsg
slug: shared-flows
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735137330198/9121906b-a33f-4737-81bb-33fd02557ab6.png
tags: google-cloud, apis, api-management, shared-flows, flow-callout-policy

---

In shared flow, we can create a sequence of steps that we can reuse at multiple places.  
Any changes to the shared flow will reflect for all the Flows which are referring to that.  
The shared flow is called from the FlowCallout policy.  
example: Some common functions are security, spike arrest, and message transformation, this kind of commonly occurring events can be kept in shared flows.

Let’s create shared flows in Apigee.

Develop tad-shared flows

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700846377905/f3501cb1-52ba-4d17-b828-786b1a9f96b6.png align="center")

Click **+create**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700847516504/9d2b91e1-8c17-45d6-81eb-369f75e902d4.png align="center")

Click on **create**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700847557437/b5327525-0d8c-4d7e-b3e4-5d2d8a7b6c55.png align="center")

GO to **develop** tab to modify the shared flow. in flow, we will add policies in step.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700847647609/1e8e828a-e356-41ea-a721-dadf7b0d7750.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700847822430/fabb8fa2-3864-4e9f-a6d2-ac5e85ca82e5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700847865000/f08425bd-72a5-411b-a8ee-432ec183d5da.png align="center")

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
<!-- here output from policy goes to response and source we are getting json
 message is also from response --> 
    <OutputVariable>response</OutputVariable>
    <Source>response</Source>
</JSONToXML>
```

save and deploy the shared flow. Once it is deployed it is available for proxies.  
Any proxies that want to convert JSON to XML formate can use this.

From the API proxy, we can’t directly call/hit the shared flow. So we need to create one more policy called **Flow callout.**

# Flow callout

This is used to call shared flows from the API proxy.  
We can also call other shared flows from shared flow by using the Flow callout policy.  
Shared flow is used to define common functionalities like security, and message validation.  
Using shared flows we can avoid redundancy in code, by utilizing a flow callout policy we can call the shared flow wherever required.  
Consider a scenario where shared flow is not deployed before the API proxy which is referring to that. Then while deploying the API proxy, API proxy throws a validation error saying, Shared flow does not exist.

**Scenario 1**:  
We will get a reject request when we pass the  
get -&gt; APIproxy (call backend to get the response) --&gt; JSON response  
(When we are sending back the response to the client we will call the flow callout to convert JSON to XML format.  
Flow callout -&gt; shared flow(convert JSON to XML)

create a reverse proxy  
with target endpoint: [https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights](https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701403976283/ecab6b49-057d-4e79-809e-0bddc84644c0.png align="center")

next - passthrough- eval- create and deploy - edit proxy - develop.  
ADD get condition as well  
proxyendpoint - GET condition

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404135117/85731383-4a82-4202-9104-955e1519b8a3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404167416/f228b673-93ba-4381-b2cd-8e2f27b9788e.png align="center")

Condition flow got added

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404182326/a1191e1a-fa1c-46f3-9cea-8883abc85435.png align="center")

before passing to backend we need to remove path suffix. "/getflights"  
target endpoint -&gt; preflow -&gt;request-&gt; Assign message policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404309739/2165908c-0faa-4ea6-ab98-e8ed9e8a5d19.png align="center")

Assign message policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    
    <AssignVariable>
        <Name>target.copy.pathsuffix</Name>
        <Value>false</Value>
        <Ref/>
    </AssignVariable>
    
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Save deploy and see the output before adding flow callout policy to convert to XML format.  
we are getting JSON responses.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404564056/efc3df4a-2996-4d7e-91cf-24f06c5f56a1.png align="center")

Convert this to XML.  
In response, we need to modify  
proxy endpoint -&gt; get condition -&gt; response - flow callout policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404657799/a8282370-b773-4f1e-adf8-71aa8242389d.png align="center")

Select the shared flow from the list of shared flows.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404687316/293a71bf-ab54-43be-8d8e-79387c55b50a.png align="center")

add

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404717416/3732dda0-c94b-4f08-8a99-1a6c63e9964a.png align="center")

Save and deploy to see the output.  
see we are getting the XML response

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701404931354/09ef8946-163c-487d-a605-58448295ab71.png align="center")

---

**scenario 2:**  
we will create one loop back service  
that receives JSON message --&gt; in response back it will send XML message(flow callout).

create a shared flow to convert JSON to XML. where is source is from the request.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701405859857/7454e7a8-10a9-44a7-ab54-928ee68b2c8e.png align="center")

In flow add JSON to XML policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701405888148/266be318-a10c-458a-9700-66b2d01a8ce5.png align="center")

Code JSON to XML policy should look like below.

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
    <!--reads the request and converts into xml then its will store back to request again-->
    <OutputVariable>request</OutputVariable>
    <!-- as per scenario we are gettign the json message from request -->
    <Source>request</Source>
</JSONToXML>
```

Save and deploy the shared flow

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701406919399/9ec22d76-d266-4ae0-b2db-7d69ee7b0995.png align="center")

Create no target proxy (loop back proxy)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701405161798/3b586b30-d4f3-49ac-9bec-c97e9003ec44.png align="center")

next - passthrough - eval - create and deploy - edit proxy - develop.  
add POST conditional flow

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701406740120/fb59745f-db93-4e22-900c-62214872d307.png align="center")

when the request comes we need to modify the message JSON to XML.  
proxy endpoint -&gt; post conditional flow -&gt; request -&gt; flow callout policy

Select the shared flow from the list. **Add**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701407017046/5e54988f-0b83-45dd-8dd1-40f0d753eaaa.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701407028039/30fb9fe8-c009-4ba9-b5f0-2f84f750c183.png align="center")

save and deploy  
send a post request with body - raw - JSON

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701407147132/e42cc292-778b-4b1c-b549-cf17157e559a.png align="center")