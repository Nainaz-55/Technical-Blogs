---
title: "Loopback API proxy"
seoTitle: "Loopback API proxy - APIgee"
seoDescription: "create Loopback API proxy in APIgee -x account"
datePublished: Tue Dec 05 2023 11:24:47 GMT+0000 (Coordinated Universal Time)
cuid: clps96ptk000408l69eg92ygi
slug: loopback-api-proxy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701765821671/bc13420a-5e7d-4771-b36e-0bc0e1494d75.png
tags: api-proxy, apigee

---

### What is a loopback service?

* It receives the request from the client, processes it, and responds without calling the backend service.
    
* Any modifications to the response can be done inside the proxy.
    
* The loopback service can also be used as a dummy back-end service.
    

**Example:** Client requests for JSON message to an API proxy. we need to construct and JSON response message inside the Apigee. And send the response back to the client.

To create a No-target API proxy.  
click on **API proxies.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769045053/1640fd67-6423-45c1-934e-b3f32221f490.png align="center")

click on **CREATE NEW**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769091609/601bdbb2-9022-4876-897c-d9fd4ff09f26.png align="center")

Click on **No target**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769122548/cbbe0cfc-38f3-4ec9-ad59-873dffa560bc.png align="center")

Fill **Name** and **Base path** in the **Proxy details** as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769176155/04a688a7-88af-447d-9a3f-78fa5e679f52.png align="center")

click on **Next.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769183296/bfe16940-cc95-4a71-86d3-427d3ba2553d.png align="center")

As of now select **security** options as **Pass through** and don't check **Quota**. Click on **Next.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769195480/9c707787-6be0-417a-a66e-e2b33daacd66.png align="center")

Select the Environment(**eval**), where this proxy is going to be deployed. Click on **Create and deploy**

Add a **GET conditional flow.** This proxy will accept only **GET** requests from the client. if the Client sends, other than a GET request proxy will through an error.

Click on, Proxy Endpoint -&gt; default -&gt; +(front of the default).  
Fill details as shown below. Click on **Add**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769203229/3a7610ee-eebc-4c96-bcfb-646538603e91.png align="center")

`getCustomer` conditional flow added.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769214787/5f894a1a-18b5-47cb-9caa-26dfbbc17592.png align="center")

To add Assign message Policy.  
Click on, Proxy endpoint -(inside)-&gt; getCustomer --&gt; Request Flow --&gt; **+Step**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701770692381/1a9e9df9-709c-4118-8c41-24e6130eebc1.png align="center")

Select **Assign Message** Policy from the list. Click on **Add.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769220917/887b3df3-3285-4626-a1fb-0094a7269c19.png align="center")

**Assign Message** policy code should look like below.

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
        <Payload contentType="application/json">
            {
                "ID": "58",
                "Name": "ABC",
                "Age": "45"
            }
        </Payload>
        <Path/>
    </Set>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

See the output in Postman.  
Refer to the below link To download and test the API in Postman.  
[https://www.outrightcrm.com/blog/postman-api-testing/](https://www.outrightcrm.com/blog/postman-api-testing/)

Generate the request message  
https://hostname//proxy\_base\_path/contional\_flow\_path(optional)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701769255780/fb02f75c-4d5d-4cab-8d76-150c37ee5d9b.png align="center")

Successfully got the JSON message.