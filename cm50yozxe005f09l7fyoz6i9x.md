---
title: "Security Policy - Basic Authentication"
seoTitle: "Security Policy"
seoDescription: "Security Policy - Basic Authentication"
datePublished: Mon Dec 23 2024 11:36:35 GMT+0000 (Coordinated Universal Time)
cuid: cm50yozxe005f09l7fyoz6i9x
slug: security-policy-basic-authentication
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734953852410/e915770b-6e72-4078-adb8-40e2ef77879a.png
tags: apis, apigee, security-policy, security-policy-basic-authentication

---

## Encode:

The client sends and username and password in the header. But the target end server requires authentication Base64 format in the Authorization header  
For that, we are using a basic authentication policy in the API proxy

This policy takes username and password as an input. It converts input into Base64 format.  
The username and password values are concatenated with the colon before Base64 encoding,  
The resulting string is assigned to the Authorization header in the below format  
`request.header.Authorization = Basic` &lt;String converted in Base64 format&gt;

create a reverse proxy: with a target endpoint: [http://httpbin.org/get](http://httpbin.org/get)  
let’s assume this target needs authorization `username: ABCD` and `password: xyz@123` we will pass in the parameters  
let’s assume the backend(target) is expecting an authorization header.  
attach basic authentication policy. proxy endpoint --&gt; preflow -&gt;&gt; request

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<BasicAuthentication continueOnError="false" enabled="true" name="Basic-Authentication-1">
    <DisplayName>Basic Authentication-1</DisplayName>
    <Operation>Encode</Operation>
    <!-- operation we are dng is encode-->
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <User ref="request.queryparam.username"/>
    <Password ref="request.queryparam.password"/>
    <!-- we are taking username and password from quesry param-->
    
    <!-- we are assigning the base64 encoded username and password into autherization header-->
    <AssignTo createNew="false">request.header.Authorization</AssignTo>
    
    <Source>request.header.Authorization</Source>
</BasicAuthentication>
```

let’s understand how Base64 encodes the username and password. The first policy will take username and password in the colon-separated form  
ex: `ABCD:xyz@123`  
encoded code is -- `QUJDRDp4eXpAMTIz`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699890396103/ec3598c2-59ae-4831-b0fe-6bb80644db33.png align="center")

Save and deploy the proxy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699890777231/d0f55ec8-596f-41d7-91cb-d4df33953e19.png align="center")

as we can see above there is an authorization header with a basic encoded version of colon separate username and password.

## Decode:

In this example, application1 sends the request along with the authorization header(in base 64 format)  
in API proxy, the basic authentication policy reads the authorization header decodes it gets the username and password, and saves that in query parameters. the that query parameters are used to construct the response message using assign msg policy.  
after that this end to the target system.  
target system responds with the constructed response.

Policy code:  
this expects the input from the authorization header `<Source>request.header.Autherization</Source>`  
It decodes the value of this and assigns the value of the user to the username query parameter and the value of the password to the password query parameter.  
These decoded values can be used for other purposes.

Create a reverse proxy, target endpoint: [http://httpbin.org/get](http://httpbin.org/get)  
passthrough --&gt; eval --&gt; deploy

edit -&gt; develop  
add basic authentication policy proxy endpoint --&gt; preflow --&gt; request

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699955139998/56448c5d-c0df-4fab-9482-e87d98d9c41d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699955148612/8b8f0320-f5dc-4c1b-884c-3e172f191222.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699955314016/edbe7bc9-fcb8-446f-8a49-e3de58b1e255.png align="center")

Policy code looks like below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<BasicAuthentication continueOnError="false" enabled="true" name="Basic-Authentication-1">
    <DisplayName>Basic Authentication-1</DisplayName>
    <Operation>Decode</Operation>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <User ref="request.queryparam.username"/>
    <Password ref="request.queryparam.password"/>
    
    <!-- we no need this part
    <AssignTo createNew="false">request.header.Authorization</AssignTo>
    -->
    
    <!-- source of input for decode operation coming from header-->
    <Source>request.header.Authorization</Source>
</BasicAuthentication>
```

Save and deploy. see the output in the header we need to send the encoded form of username and password.  
Before converting colon-separated username and password to encoded format  
ex: `ABCD:xyz@123`  
encoded code is -- `QUJDRDp4eXpAMTIz`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699957828089/aae02c83-addc-4951-851b-e64c92bff1f4.png align="center")

We sent the Authorization code (encoded code of username and password) and we got the username and password in response