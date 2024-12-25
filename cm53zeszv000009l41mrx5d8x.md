---
title: "XML threat protection Policy"
seoTitle: "XML threat protection Policy"
seoDescription: "XML threat protection Policy"
datePublished: Wed Dec 25 2024 14:19:57 GMT+0000 (Coordinated Universal Time)
cuid: cm53zeszv000009l41mrx5d8x
slug: xml-threat-protection-policy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735136302606/0e12e0e8-cdca-49ea-b642-616b1877d42d.png
tags: google-cloud, apis, apigee, api-management-google-cloud

---

This policy detects XML payload attacks based on the configured limits on XML message parts.  
When the message is received from the client or other system, Content-Type should be application/XML, if it is not, a policy is not enforced on that message.  
Scenario:  
\- To destabilize the system, hackers may send Large and complex XML messages to the service.  
\- It uses more memory and CPU, and XML parsers cannot handle this kind of message.  
\- This results in the termination of the service.  
These attacks can be mitigated if the service uses XML Threat protection policy.

**XML Configuration file - XML Threat Protection Policy**

```xml
<XMLThreatProtection continueOnError="false" enabled="true" name="XML-Threat-Protection-1">
<DisplayName>XML Threat Protection-1</DisplayName>
<Properties/>
<NameLimits><!-- it will specify Element length, Attribute length, NamespacePrefix
length, ProcessingInstructionTarget should be 10 or within 10-->
    <Element>10</Element>
    <Attribute>10</Attribute>
    <NamespacePrefix>10</NamespacePrefix>
    <ProcessingInstructionTarget>18</ProcessingInstructionTarget>
</NameLimits>
<!-- Source: from where we are getting the XML message. ie request-->
<Source>request</Source>
<StructureLimits>
    <!-- length of the node structure should not excceed 5-->
    <NodeDepth>5</NodeDepth> 
    <!-- Attribute Count Per Element should be 2 or less than 2.  
    similarly,Namespace Count Per Element -->
    <AttributeCountPerElement>2</AttributeCountPerElement>
    <NamespaceCountPerElement>3</NamespaceCountPerElement>
    <ChildCount includeComment="true" includeElement="true" includeProcessingInstruction="true" includeText="true">3</ChildCount>
</StructureLimits>
<ValueLimits>
    <!-- it will check the size of values inside Text, Attribute, NamespaceURI
    , Comment, ProcessingInstructionData-->
    <Text>15</Text> <!-- element value shlould be 15 or within 15-->
    <Attribute>10</Attribute>
    <NamespaceURI>40</NamespaceURI>
    <Comment>10</Comment>
    <ProcessingInstructionData>40</ProcessingInstructionData>
</ValueLimits>
</XMLThreatProtection>
```

**Sample XML data:**

```xml

<BillInfo xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <BillNumber>8888</BillNumber>
    <BillDate>2022-07-28</BillDate>
    <BillTime>10:36:55.03</BillTime>
    <BillerDetails code="9836" name="string">Jane</BillerDetails>
    <Customer>
        <Name>abc</Name>
        <Phone>7483824106</Phone>
        <Mail>nkptech@gmail.com</Mail>
        <Address>
            <Street_Addr1>Bay area</Street_Addr1>
            <Street_Addr2>5th cross</Street_Addr2>
            <PostCode>A84HJK</PostCode>
            <Country>USA</Country>
        </Address>
    </Customer>
</BillInfo>

<!-- <BillInfo>, <BillNumber>, <BillDate>,.... are all Elements
      code="9836", name="string", .. Attributes 
      xmlns:xsi  .. NamespacePrefix
      href="xsl_for_each.xsl" .. ProcessingInstructionTarget-->

<!-- NodeDepth: length of the node structure -->
<!-- AttributeCountPerElement: <BillerDetails code="9836" name="string"> is 2  -->
<!-- NamespaceCountPerElement: <BillInfo xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<!-- ChildCount(number of child nodes inside noded): including Comments including Elements inside it -->
```

Implement policy in Apigee.

Create no target API proxy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700585920707/526d0125-1cbd-4a84-8a36-4072a9937649.png align="center")

create and deploy - edit proxy - develop  
we need to validate when the request will come.  
proxy endpoint - preflow - request - add XML threat protection policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700586031894/e80e9d3d-8898-4f7a-84c2-98cf65ed32e7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700586045053/fbcb9202-c409-43e1-87be-2d95ca585365.png align="center")

XML threat protection policy code should look like below. save and deploy

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<XMLThreatProtection continueOnError="false" enabled="true" name="XML-Threat-Protection-1">
    <DisplayName>XML Threat Protection-1</DisplayName>
    <Properties/>
    <NameLimits>
        <Element>10</Element>
        <Attribute>10</Attribute>
        <NamespacePrefix>10</NamespacePrefix>
        <ProcessingInstructionTarget>18</ProcessingInstructionTarget>
    </NameLimits>
    <Source>request</Source>
    <StructureLimits>
        <NodeDepth>5</NodeDepth>
        <AttributeCountPerElement>2</AttributeCountPerElement>
        <NamespaceCountPerElement>3</NamespaceCountPerElement>
        <ChildCount includeComment="true" includeElement="true" includeProcessingInstruction="true" includeText="true">3</ChildCount>
    </StructureLimits>
    <ValueLimits>
        <Text>15</Text>
        <Attribute>10</Attribute>
        <NamespaceURI>40</NamespaceURI>
        <Comment>10</Comment>
        <ProcessingInstructionData>40</ProcessingInstructionData>
    </ValueLimits>
</XMLThreatProtection>
```

See the output  
Send POST request with sample XML data.

```xml
<!--  sample XML data -->

<?xml-stylesheet type="text/xsl "href="xsl_for_each.xsl" ?>
<BillInfo xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<BillNumber>8888</BillNumber>
<BillDate>2022-07-28</BillDate>
<BillTime>10:36:55.03</BillTime>
<BillerDetails code="9836" name="string">Jane</BillerDetails>
<Customer>
<Name>abc</Name>
<Phone>7483824106</Phone>
<Mail>nkptech@gmail.com</Mail>
<Address>
<Street_Addr1>Bay area</Street_Addr1>
<Street_Addr2>5th cross</Street_Addr2>
<PostCode>A84HJK</PostCode>
<Country>USA</Country>
</Address>
</Customer>
</BillInfo>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700586484605/6ba7a66f-2785-49ab-90a0-af7a214c9165.png align="center")

we are getting errors.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700662200118/7633fee8-d112-4e0f-84b4-b26f6e0c40ad.png align="center")

Because: Namespace uri length exceeded 40 at line 1(possibly around char 64)  
So, the length of the namespace URI should be less than 40. modify it  
`<BillInfo xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">`  
TO  
`<BillInfo xmlns:xsi="http://www.w3.org/2001/XML">`  
send the request again.  
Got the error:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700662454045/3dba792f-4082-4f23-bfb0-88b4d7064deb.png align="center")

Execution failed. reason: Children's count exceeded 3  
delete some child elements.  
`<BillNumber>8888</BillNumber> <BillDate>2022-07-28</BillDate>`  
Code looks like

```xml
<BillInfo xmlns:xsi="http://www.w3.org/2001/XML">
    
    <BillTime>10:36:55.03</BillTime>
    <BillerDetails code="9836" name="string">Jane</BillerDetails>
    <Customer>
        <Name>abc</Name>
        <Phone>7483824106</Phone>
        <Mail>nkptech@gmail.com</Mail>
        <Address>
            <Street_Addr1>Bay area</Street_Addr1>
            <Street_Addr2>5th cross</Street_Addr2>
            <PostCode>A84HJK</PostCode>
            <Country>USA</Country>
        </Address>
    </Customer>
</BillInfo>
```

After modifying the Sample XML code based on the XML threat protection needs.  
Code looks like.

```xml
<BillInfo xmlns:xsi="http://www.w3.org/2001/XML">


    <BillTime>10:36:55.03</BillTime>
    <Biller code="9836" name="string">Jane</Biller>
    <Customer>
        <Name>abc</Name>
        <Phone>7483824106</Phone>
        
        <Address>
            <Strdr1>Bay area</Strdr1>
            
            <PostCode>A84HJK</PostCode>
            <Country>USA</Country>
        </Address>
    </Customer>
</BillInfo>
```

Output is.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700662987033/97ae2cfe-a4cf-4b72-828c-601269262923.png align="center")