---
title: "HMAC Policy Implementation."
seoTitle: "HMAC Policy"
seoDescription: "HMAC Policy Implementation."
datePublished: Wed Dec 25 2024 16:11:14 GMT+0000 (Coordinated Universal Time)
cuid: cm543dwmm000109l24ketg9hn
slug: hmac-policy-implementation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735142994698/055efe3c-b1ba-4509-bb50-66492f5a17f2.png
tags: google-cloud, apis, api-management, apigee, hmac-policy-implementation

---

This policy generates and verifies the Hash-Based Message Authentication Code(MAC).  
It is also called keyed Message Authentication code as it uses a secret key to generate the MAC.  
HMAC uses any of the following cryptographic Hash functions.  
\- SHA-1  
\- SHA-224  
\- SHA-256  
\- SHA-384  
\- SHA-512  
\- MD-5  
HMAC policy applied on the message to generate MAC on that message.  
During this process, it uses any of these hash functions and a secret key.  
The message can be of type any content type.

The sender and receiver share the same secret key. ex: code4545

Sender before sending the data to the receiver. The sender creates a MAC on the message by using the secret key and the Hash algorithm.  
genrates the MAC code: aoZDvpB2Misg=  
Then it sends the request along with the MAC code and the secret key used.

The receiver receives this data and again it applies the Hash algorithm by using the secret key on the received message to generate the MAC code  
Generates the MAC code: aoZDvpB2Misg=  
then checks Reciever receiver-generated MAC code with Sender sent MAC code.

If both are equal validation is successful. and conclude that the message is not tampered

let’s build practically,  
We will create two API proxies. One is to generate the MAC code. second validate the message

**first:** create no target proxy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700230182670/bd9a79ce-fbcf-4a11-8f27-48297a714b3f.png align="center")

next - next- create and deploy.  
edit proxy - develop.  
add HMAC policy - proxy endpoint - preflow - request.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700230310484/3d6c733d-8b04-426e-a4dc-0907ffbb651e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700230319308/573dc5c3-c4b2-43e4-a7da-b1796701a229.png align="center")

The HMAC policy code looks like beloved.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<HMAC name="HMAC-1">
    <DisplayName>HMAC-1</DisplayName>
    <Algorithm>SHA-256</Algorithm>
    <!-- it uses SHA-256 algorithm-->
    <Message ref="request.content"/>
    <!-- it means how it recieves the message.. It takes the message from request-->
    <SecretKey ref="private.secretkey"/>
    <!-- and how it recieves the secret key. we need to pass the secret key in this location. then it fetches the secret key-->
    <!-- we pass the value to secrete key using Assign message policy-->
    <Output>hmac_value</Output>
    <!-- generated MAC code is stored in it-->
</HMAC>
```

we apply the assign message policy, to fetch the value of the secret key from the request.  
proxy endpoint - preflow - request  
Note: Before HMAC policy we need to add the Asiign msg policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700231247162/2b45c24d-0dfc-4864-8c2f-e33591006ef7.png align="center")

assign message policy code looks like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <!-- Assign variable "private key"-->
    <AssignVariable>
        <Name>private.secretkey</Name>
        <!-- value we will pass it from the query param (reference)-->
        <Ref>request.queryparam.secretkey1</Ref>
    </AssignVariable>
    
    <!-- after assigning value to that variable, we no longer need that query param so need to remove it-->
    <!-- once value of secretkey1 assign to "private.secretkey". we no need "secretkey1". we will remove it. otherwise it will pass to next step-->
    <Remove>
        <QueryParams>
            <queryparam name="secretkey1"></queryparam>
        </QueryParams>
    </Remove>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

the adding this we got value for a secret key. it should be added before the HMAC policy. so move it first.  
Drag assign message policy to come first.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700233922053/97924ed0-1b8f-49ad-a9ca-e671f331ccc5.png align="center")

Create a sample message to respond back to the client with the HMAC code( generated in the above process)

proxy endpoint - postflow - response - add assign message policy (to add the JSON payload to send back to client with the HMAC code).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700234563545/a9238443-6526-482c-8fe5-bb1fcf61c6f1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700234573267/7a23f3dc-e458-4f7e-be2a-81085a14c8e4.png align="center")

assign message policy 2 code should look like below

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-2">
    <DisplayName>Assign Message-2</DisplayName>
    <Properties/>
    
    <Set>
        <Headers/>
        <QueryParams/>
        <FormParams/>
        <Verb>POST</Verb>
        <Payload contentType="application/json">
            { 
                "status": "generated MAC successfully",
                "MAC_VALUE": {hmac_value}
            }
            
        </Payload>
        <Path/>
    </Set>
    
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Save and deploy.  
send the request with the secret key (secretkey: code4545) in the query param and add the body-raw-json.  
in output, we will get MAC\_VALUE generated in policy using secretkey and content.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700240220968/ea4a5801-4ac8-441c-b26d-e014e95308ad.png align="center")

---

**Second:** proxy to verify the message content passed from the request  
create no target API proxy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700235613988/2afa5ee7-0467-4c07-beae-bf402188e48d.png align="center")

next(passthrough)- next(eval)- create and deploy- edit proxy - develop  
in preflow -request we will verify these message content.  
add HMAC policy. proxy endpoint - preflow - request

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700235734179/1bbfd9e0-3abc-4b02-a9fa-483cddc3e266.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700235769588/2f6f2219-7339-44de-a4d2-84c5913e9521.png align="center")

HMAC code should look like Below

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<HMAC name="HMAC-1">
    <DisplayName>HMAC2</DisplayName>
    <!-- same algorithm we should use as sender-->
    <Algorithm>SHA-256</Algorithm>
    <!-- we are getting content from request //in order understand how HMAC will work-->
    <Message ref="request.content"/>
    <SecretKey ref="private.secretkey"/>
    <!-- no need of output hmac_value. Bcz we are verifying here
    <Output>hmac_value</Output>
    -->
    <!-- we need to add the verification value tag to verify the MAC value
    we are getting hmac value from request header-->
    <VerificationValue encoding="base64" ref="request.header.hmacvalue"/>
</HMAC>
```

Save and add the assign msg policy as we added in the rest proxy to get the value of secretkey form queryparam. and drag the assign msg policy to first where HMAC policy is there.  
Assign message policy code should look like below

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
  
    <AssignVariable>
        <Name>private.secretkey</Name>
        <Value/>
        <Ref>request.queryparam.secretkey1</Ref>
    </AssignVariable>
    
    <Remove>
        <QueryParams>
            <queryparam name="secretkey1"></queryparam>
        </QueryParams>
    </Remove>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700237602541/7f4ed569-5c26-4924-9a27-9266b88658fb.png align="center")

after validation. it will give a success response else the validation error.  
In proxy endpoint - postflow -response. we will add the assign message policy to add the success or failure of response.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700237902556/2873a5ad-bef7-42fd-b59a-86ddd2945197.png align="center")

Assign message policy 2 code look like below

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-2">
    <DisplayName>Assign Message-2</DisplayName>
    <Properties/>
  
    <Set>
        <Headers/>
        <QueryParams/>
        <FormParams/>
        <Verb>POST</Verb>
        <Payload contentType="application/json">
            {
                "status": "success",
                "hmac_value": "{request.header.hmacvalue}",
                "secret_key": "{private.secretkey}"
            }
        </Payload>
        <Path/>
    </Set>
   
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

In response, we will add HAMC value and secret key to the client if it is successful.  
To check whether it is a success or failure we need to add one condition in the proxy endpoint code -inside the response tag

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700238399760/742c5a06-f28a-400a-9663-57a7c1f93d47.png align="center")

The proxy endpoint (default) code should look like the below

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>Assign-Message-1</Name>
            </Step>
            <Step>
                <Name>HMAC-1</Name>
            </Step>
        </Request>
        <Response/>
    </PreFlow>
    <Flows/>
    <PostFlow name="PostFlow">
        <Request/>
<!-------------------------------------------- -->
        <Response>
            <Step>
                <!--Add these condition to chek whthere verifcation sucessfull or failed-->
                <Condition>HMAC.fail = ''</Condition>
                <Name>Assign-Message-2</Name>
            </Step>
        </Response>
<!------------------------------------------  -->
    </PostFlow>
    <HTTPProxyConnection>
        <BasePath>/hmacverify</BasePath>
    </HTTPProxyConnection>
    <RouteRule name="noroute"/>
</ProxyEndpoint>
```

Save and deploy.  
output1:: By sending the same payload and secretkey, and sending HMAC value in the request header.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700240449237/7099e580-5c87-4cd1-af51-cddee5aa8f7b.png align="center")

output2: let's see the output by sending the incorrect(modifying the content) content in the request

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700240508683/1a06a26d-afb3-4ff2-aba1-ab678eba8f13.png align="center")

Getting error message.

The main purpose of using HMAC policy is to verify the content between the client and the backend, whether the client is getting correct data or not, OR verify whether correct data is getting stored in the backend as like client sent, whether it got modified in between flow.