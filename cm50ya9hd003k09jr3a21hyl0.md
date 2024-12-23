---
title: "Apigee Assignment - Based on the Service Callout policy and JSON protection."
seoTitle: "Apigee Assignment"
seoDescription: "Apigee Assignment - Based on the Service Callout policy and JSON threat protection."
datePublished: Mon Dec 23 2024 11:25:07 GMT+0000 (Coordinated Universal Time)
cuid: cm50ya9hd003k09jr3a21hyl0
slug: apigee-assignment-based-on-the-service-callout-policy-and-json-protection
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734952968473/227710eb-150c-4504-8b68-759f946759d0.png
tags: apigee, service-callout-policy, json-threat-protection-policy

---

CREATE 2 Proxy **JSON\_threat** and **JSON to XML**.  
PROXY 1 should be attached to the JSON threat protection policy and other policies if needed.  
PROXY 2 should be attached with service callout -Proxy chaining policy, JSON to XML policy, and other policies if needed,  
NOTE: JSON threat protection policy  
\- The number of array elements should not exceed 5.  
\- Container depth should not exceed 6.  
\- Name length should not exceed 15.  
A client should send a request message with JSON data and get XML data in response.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703310518450/b9bcfcef-d34d-4a2e-80f5-d47464a50053.png align="center")

## Implementation

Create No target API PROXY 1 - **JSON\_threat**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703310828256/5cb8c699-c123-49b3-b89d-668b52cdc01f.png align="center")

Proxyendpoint - Preflow - Request - ADD JSON threat protection policy.

JSON threat protection policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<JSONThreatProtection continueOnError="false" enabled="true" name="JSON-Threat-Protection-1">
    <DisplayName>JSON Threat Protection-1</DisplayName>
    <Properties/>
    <ArrayElementCount>5</ArrayElementCount>
    <ContainerDepth>6</ContainerDepth>
    <ObjectEntryCount>5</ObjectEntryCount>
    <ObjectEntryNameLength>15</ObjectEntryNameLength>
    <Source>request</Source>
    <StringValueLength>15</StringValueLength>
</JSONThreatProtection>
```

Save and Deploy.  
PROXY 1 detects malicious JSON data based on the configured limits on JSON structure.

---

Create No Target PROXY 2 - **JSONtoXML.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703311852008/ac1a7a9c-3875-4b79-bbdd-c2c5e375650d.png align="center")

Proxyendpoint - Preflow - Request - **ADD** Service callout proxy chaining policy (from here call the **JSON\_threat** proxy).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703312001166/1a7548bf-fb01-4c02-b455-93c5c5b905a8.png align="center")

Proxyendpint - Preflow - Request - ADD Assign message policy (To add the response of JSON\_threat proxy to Request message.  
The assign message policy code should look like the one below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <Copy source="calloutResponse">
        <Payload>true</Payload>
    </Copy>
    <Set>
        <!-- posting the XML msg to request-->
        <Verb>POST</Verb>
    </Set>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Proxyendpoint - Postflow = Request - Add JSON to XML policy.  
The JSON to XML policy code should look like the one below.

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

Save and Deploy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703314008674/61db8b81-a9cf-419e-9121-494e4affab86.png align="center")

A client sends a POST request message to PROXY 2 (**JSONtoXML)** with JSON data. In response, client should get XML format data.

JSON data is based on the configured limits in the JSON threat protection policy.

```json
{
    "BillNumber": "8888",
    "BillDate": "2022-02-23",
    "Customer": {
        "Name": "Prashant",
        "Mail": "nk@gmail.com",
        "Address": {
            "Strdr1": "Bay area",
            "Country": "USA"
        }
    },
    "pDetails": {
        "paymentType": "CARD"
    },
	"pDetails2": {
        "paymentType": "CARD"
    }
   
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703313876528/2b834241-4a1f-466c-9a10-21a827128561.png align="center")