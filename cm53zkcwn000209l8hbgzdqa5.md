---
title: "JSON Threat Protection Policy"
seoTitle: "JSON Threat Protection Policy"
seoDescription: "JSON Threat Protection Policy"
datePublished: Wed Dec 25 2024 14:24:16 GMT+0000 (Coordinated Universal Time)
cuid: cm53zkcwn000209l8hbgzdqa5
slug: json-threat-protection-policy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735136608629/27c7861d-cb26-45ce-adff-f441297be7bb.png
tags: apis, apigee, json-threat-protection-policy, api-management-google-cloud

---

This policy protects from JSON malicious API requests.  
This policy detects malicious JSON based on the configured limits on JSON structure.  
When the message is received from the client or other system, Content-Type should be application/JSON, if it is not, the policy won’t allow that API request to pass through.  
Scenario:  
\- To destabilize the system, hackers may send Large and complex JSON messages to the service.  
\- As it uses more memory and CPU, JSON parsers unable to handle this kind of message.  
\- This results in a service crash.  
These attacks can be mitigated if the service uses a JSON Threat protection policy.

**XML Configuration file - JSON Threat Protection Policy**

```xml
<JSONThreatProtection continueOnError="false" enabled="true" name="JSON-Threat-Protection-1">
<DisplayName>JSON Threat Protection-1</DisplayName>
<Properties/>
<!-- inside array there should be <=2 element-->
<ArrayElementCount>2</ArrayElementCount>
<!-- ContainerDepth: after creating a tree structure , Node depth(lengthg of tree)
is known as ContainerDepth-->
<ContainerDepth>3</ContainerDepth>
<!-- ObjectEntryCount: number of ojects in JSON message-->
<ObjectEntryCount>5</ObjectEntryCount>
<ObjectEntryNameLength>11</ObjectEntryNameLength>
<!-- Source: from where we are getting the JSON msg. ie,. from request-->
<Source>request</Source>
<!-- length/size of value of the objects/elements-->
<StringValueLength>15</StringValueLength>
</JSONThreatProtection>
```

Sample JSON message.

```json
{
    "BillNumber": "8888",
    "BillDate": "2022-02-23",
    "Customer": {
        "Name": "Prashant",
        "Mail": "nkptech@gmail.com",
        "Address": {
            "Street_Addr1": "Bay area",
            "Country": "USA"
        }
    },
    "paymentDetails": {
        "paymentType": "CARD"
    },
	"paymentDetails2": {
        "paymentType": "CARD"
    },
    "CartItems": [{
            "ProductName": "OVEN"
        }, {
            "ProductName": "Book"
        }
    ]
}

/*ArrayElementCount:  
[{
            "ProductName": "OVEN"
        }, {
            "ProductName": "Book"
        }
    ]*/

/* ContainerDepth: for above example:*/

/*ObjectEntryCount: { (1), Customer(2), Address(3), paymentDetails(4), 
                   paymentDetails2(5), CartItems(6) */

/*ObjectEntryNameLength: size of above object/elements*/
```

Let’s implement this in APigee.

Create no target API proxy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700718558351/63961ca1-f0c6-43fe-9e14-9b6dab8d9949.png align="center")

passthrough - eval - create and deploy - edit poxy - develop.  
when the request we need to check whether the JSON message is tampered or not. So  
proxy endpoint- preflow- request - add JSON threat protection policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700718689514/80c29b07-657b-4a96-b019-7f93297e0341.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700718698863/a0043a21-f1f6-44fb-8b30-0c1f35fe5d39.png align="center")

JSON threat protection policy code should be like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<JSONThreatProtection continueOnError="false" enabled="true" name="JSON-Threat-Protection-1">
    <DisplayName>JSON Threat Protection-1</DisplayName>
    <Properties/>
    <ArrayElementCount>2</ArrayElementCount>
    <ContainerDepth>4</ContainerDepth>
    <ObjectEntryCount>5</ObjectEntryCount>
    <ObjectEntryNameLength>11</ObjectEntryNameLength>
    <Source>request</Source>
    <StringValueLength>15</StringValueLength>
</JSONThreatProtection>
```

Save and deploy.  
see the output: send the POST request with JSON content. Make sure that content\_type: application/JSON is in the header of the request.  
JSON message.

```json
{
    "BillNumber": "8888",
    "BillDate": "2022-02-23",
    "Customer": {
        "Name": "Prashant",
        //"Mail": "nkptech@gmail.com", bcz of this getting error. So modify to
        "Mail": "nk@gmail.com"
        "Address": {
        //Exceeds object entry name length
        // "Street_Addr1" modify to
            "Strdr1": "Bay area",
            "Country": "USA"
        }
    },
    //Exceeded object entry name length
    //"paymentDetails" modify to
    "pDetails": {
        "paymentType": "CARD"
    },
    //Exceeded object entry name length
    //"paymentDetails2" modify to
	"pDetails2": {
        "paymentType": "CARD"
    }
    //Exceeded object entry count . so delete this
    /*"CartItems": [{
            "ProductName": "OVEN"
        }, {
            "ProductName": "Book"
        }
    ]*/
}
```

We got an error.  
Execution failed. reason: "Exceeded string value length at line 6"

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700719224201/853318fb-70d3-4cfd-ba7d-e8f90e07e453.png align="center")

Execution failed. reason: Exceeds object entry name length at line 8

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700719448115/a7eba010-e48f-4c18-ad5d-31542b878114.png align="center")

After modifying the code related to the threat protection code. sample JSON content looks like below.

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

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700720047746/a0ae2872-7dcd-4be1-bdb5-d10edd1e7a8f.png align="center")