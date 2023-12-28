---
title: "Elements of API proxy."
seoTitle: "Elements of API proxy."
seoDescription: "In this blog, we will understand the elements of API proxy in detail."
datePublished: Thu Dec 28 2023 13:22:52 GMT+0000 (Coordinated Universal Time)
cuid: clqp8j6b0000208jv558ociyf
slug: elements-of-api-proxy
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1703759660458/6701c1db-8d55-4b52-bf72-994fa3a43199.png
tags: apis, google-cloud-platform, apigee

---

In this blog, we will understand the elements of API proxy in detail.

Before going to Elements of API proxy let's understand the role played by Environments and Environment groups.

## Environments

It is the runtime context to deploy API proxies. API proxies are in a running state once it is deployed to an environment. Proxies can deployed in multiple environments.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703685287477/728c0a86-d424-4d45-85da-5f8f8eda074f.png align="center")

## **Environment groups**

It is a collection of environments.  
\- hostnames are defined on the environment groups.  
Environment groups --&gt; Hostnames --&gt; Environments (*ex. prod, dev*)  
To access each (environment --&gt; proxy), it should be associated with hostnames, Hostnames should be added inside the environment groups.  
We can add multiple hostnames in each group.  
We can't add the same hostnames to multiple environment groups.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703671296328/2eb2b37e-714a-46ed-a6e5-4826159071bd.png align="center")

Create a new **+Environment** **Group** and add hostnames and environments to that group

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703685161596/5a7c66ed-81db-464a-b938-1d83fc1c42ab.png align="center")

Hostname: **34.144.253.144.nip.io**  
The domain name of that hostname: **secoundproject-407905-eval.apigee.net**

## **API Proxy**

* API proxies act as interfaces for the developers to access backend services.
    
* Backend services are secured by using API proxies.
    
* All the security and rate-limiting policies are applied to the API proxy before passing it to the backend.
    
* In this type of architecture, the backend service URL is not shared with the consumers, the consumer must hit the API proxy to get the service from the backend.
    
* the backend service URLs are not shared with the consumers as the services are exposed through proxy endpoints.
    
* Consumers are unaware of the backend service changes, for example in the database.
    
* API tools provide out-of-the-box policies to apply to the API proxies.
    

Let's create a new proxy and understand each element of the Proxy:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703693919254/686e2396-1a96-40a1-8d6b-de08956ec2de.png align="center")

1. **Name**: This field identifies the name of the proxy.
    
2. **Base path**: This field identifies the base path of the proxy. It can be the same as the name of the proxy. But it should not contain the capital alphabets.
    
3. **Target (Existing API):** Actulal API of the Backend Server.  
    [https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights](https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights)  
    The output of this is,
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703758762664/b96098b6-6659-463d-baeb-fe6b718deb3b.png align="center")
    

Click on **Next**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703694571761/e421e197-aaf9-4ab4-8ded-fdcbeae917a5.png align="center")

1. **Security: Authorization**: We can choose security policies while creating the proxy. If we don't want to choose any policy we can select **pass-through (no authorization).**
    
2. **Security: Browser:** Select this if you want to access the proxy through the web browser.
    
3. **Quota**: Select this if wanted to apply the Quota policy on Proxy while creating it.  
    But one condition, To make this option available, we need to select a security option, and Quata details for this proxy should be configured in the API product.
    

Click on **Next**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703695298706/59d1e873-aeec-4faa-a6de-02c370a83018.png align="center")

Select the **eval** environment to deploy while creating the API proxy or this option is optional we can deploy this after creating and editing the proxy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703697302996/d85829d7-3927-4be3-bc7b-696957ee80f0.png align="center")

But now I will select the **eval** environment and we move forward to understand the rest of the elements of Proxy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703697608563/bb60535f-ae99-4f38-a9b4-8270746d1589.png align="center")

After successfully creating and deploying the proxy. Click on **Edit Proxy** to update the proxy as per our requirements.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703742777383/46c9eb25-9bd0-41a9-93eb-651ab3ed9405.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703742778768/ed76dbed-4829-4e9f-8949-332e2bc5b31f.png align="center")

1. **Deployments**:
    
    * **Environment** \- It identifies in which environment the proxy is deployed.  
        \- In the eval environment proxy is deployed.  
        \- *Revision* - Version of the proxy. when every time the proxy gets deployed, it will get stored in a new revision(version).
        
    
    Proxy has Two endpoints one is the proxy endpoint connected to the client and another one is the target endpoint connected to the backend service.
    
2. **Proxy Endpoints**:
    
    * By default the name of the proxy endpoint will be "**default**". You can change the name of the endpoint.
        
    * We can see the proxy basepath here "/basepathofproxy".
        
    * It has two flows Preflow and Postflow, we can add conditional flow if needed.
        
3. **Target Endpoint:**
    
    * By default the name of the target endpoint will be "**default**". You can change the name of the endpoint.
        
    * Here you can see the actual API of the backend service. Which we are not sharing with the client directly.
        
    * It has two flows Preflow and Postflow, we can add conditional flow if needed.
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703750470961/87046ceb-8f9e-47d7-91d0-490a6e48add4.png align="center")

1. **OVERVIEW**: In this tab, you can see the details of the proxy.
    
2. **DEVELOP**: In this tab, you can edit the proxy as per our requirement by adding the policies and developing the code.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703751418633/92a67d47-3a0c-4427-89e1-5eea6225864c.png align="center")
    
3. **DEBUG**: In this tab, you can debug or check the error of the proxy by looking at each flow.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703751448314/2a0019ae-9a0d-414d-8baf-c6f0fd8d236f.png align="center")
    
4. **PERFORMANCE**: In this tab, you can check the performance of the proxy.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703751468352/46e1a925-92d3-4dab-8b8a-edc50df1be57.png align="center")
    

Click on the **DEVELOP** tab to see the code of the endpoints.

The **proxy endpoint** code will look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!-- we can change the name of endoint below from "default" to "__"-->
<ProxyEndpoint name="default">
    <!-- It is preflow of proxy endpoint. If any policies added it will show inside 
    the preflow tab-->
    <PreFlow name="PreFlow">
        <Request/>
        <Response/>
    </PreFlow>
    <!-- if you add conditional flow. The detalis related to it will show here-->
    <Flows/>
    <!-- It is postflow of proxy endpoint. If any policies added it will show inside 
    the postflow tab-->
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <HTTPProxyConnection>
        <!-- basepath of proxy endpoint-->
        <BasePath>/basepathofproxy</BasePath>
    </HTTPProxyConnection>
    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

The **target endpoint** code will look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!-- we can change the name of endoint below from "default" to "__"-->
<TargetEndpoint name="default">
    <!-- It is preflow of target endpoint. If any policies added it will show inside 
    the preflow tab-->
    <PreFlow name="PreFlow">
        <Request/>
        <Response/>
    </PreFlow>
    <!-- if you add conditional flow. The detalis related to it will show here-->
    <Flows/>
    <!-- It is PostFlow of target endpoint. If any policies added it will show inside 
    the PostFlow tab-->
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <!-- Actual API of backend server-->
    <HTTPTargetConnection>
        <URL>https://virtserver.swaggerhub.com/jihobe4823/Indian-Air-Flight-API/v1/flights</URL>
    </HTTPTargetConnection>
</TargetEndpoint>
```

**Save** and **Deploy** the proxy.

The client/consumer is provided by the API of the Proxy instead of the API of the Backend server.

**API of the Proxy**: http://yourhost:port/base\_path\_of\_proxy  
As per the above example,  
\- yourhost:port = 34.144.253.144.nip.io  
\- base\_path\_of\_proxy = basepathofproxy  
API for the above example proxy is:  
https://34.144.253.144.nip.io/basepathofproxy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1703758670008/e6659129-6ec7-4f80-9625-43e2db22b606.png align="center")

> Without sharing an actual Backend API to the client/consumer. By using the proxy API, client/consumer got the data same as we will get the data by using the actual Backend server API.
> 
> In proxy we can secure the Backend API from the client actions.