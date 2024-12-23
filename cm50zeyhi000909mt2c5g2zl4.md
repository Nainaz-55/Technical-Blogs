---
title: "JSON Web Signature (JWS)"
seoTitle: "JSON Web Signature (JWS)"
seoDescription: "JSON Web Signature (JWS)"
datePublished: Mon Dec 23 2024 11:56:46 GMT+0000 (Coordinated Universal Time)
cuid: cm50zeyhi000909mt2c5g2zl4
slug: json-web-signature-jws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734954960895/811b8ade-a540-4834-8dbd-1a428ea39774.png
tags: google-cloud, api-management, api-security, apigee, json-web-signature-jws

---

This policy is a data structure representing a MACed (HMAC algorithm) or digitally signed message.  
It is signed using a shared key or public/private key.  
We can choose different algorithms for these.  
Example:  
\- HS256 algo uses shared key.  
\- RS256 algo uses public/private key.

How it is different from JWT?  
\- In JWS, Payload can be in any format but in JWT, it is always a JSON object.  
\- There is a Detach option in JWS, in this payload is ignored, but payload is always attached to JWT.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700564923049/47be4c92-ca93-42ee-959c-2406d90f26d9.jpeg align="center")

It has three structures.  
\- Header  
\- Payload (optional)  
\- Signature  
These 3 sections are separated by .(dot) operator  
encoded formate ASCII format of header and payload.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700565046484/5e62f4bc-3a9c-456e-aa55-dca31cfdb49f.jpeg align="center")

JWS Policy implementation.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700565160502/7d0fbfc0-ba4e-4921-8f12-ee62c3946ffc.jpeg align="center")

The client sends the request with a secret key and payload (optional). when it reaches the API proxy, the proxy will add Generate JWS Policy, here it accepts the key and generates the token using Algorithm. The generated token is sent back to the client using the Asign message policy by constructing the message.

**JWS Verification Flow:**  
The client sends a request with a secret key, Payload data (optional), and Authorization barrier &lt;JWS&gt; or Send JWS token from other paths. In the API proxy verify the Token sent using the Verify JWS Policy. and send back the successful message to the client if verification is successful by creating the Success response using the assign message policy.

First proxy:  
Create no target proxy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700565763364/40dc9e8f-7eef-4b44-bb0d-19dd28c44b95.png align="center")

next- passthrough- eval next- edit proxy - develop.  
We are sending the JWS token back to the client so  
Proxy endpoint - postflow - response - add Generate JWS Policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700566743340/38b781ed-58a4-4360-b368-14302988ded7.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700566761205/207cc909-e1a5-4fae-8f2c-8e8839cfac4b.png align="center")

Generate JWS policy code should look like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<GenerateJWS name="Generate-JWS-1">
    <DisplayName>Generate JWS-1</DisplayName>
    <Algorithm>HS256</Algorithm>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <CriticalHeaders/>
    <SecretKey>
        <Value ref="private.secretkey"/>
    </SecretKey>
    <!-- Payload we will pass it from the request-->
    <Payload ref="request.content"/>
    <!-- Generate the token by applying the secret key on payload content by uing algorithm--> 
    <!-- true, we can detach the content(means we are not adding any content while requesting
    false, meand we are attaching the content-->
    <DetachContent>false</DetachContent>
    <!-- can claim the additional headers name from client like user name-->
    <AdditionalHeaders>
        <Claim name="issuedby" ref="request.header.user1"/>
    </AdditionalHeaders>
    <!-- generated token is stored in variable "jwstokenvalue"-->
    <OutputVariable>jwstokenvalue</OutputVariable>
</GenerateJWS>
```

we need to get the value private.secretkey. assign a value to it using the assign message policy by taking the value from the query parameter in the request.  
Proxy endpoint - postflow - response - add assign message policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700573641977/9f2eec1e-d51b-4cac-b855-b9d7991015f0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700573709072/f23c94b6-bbda-4b71-a4e8-68a2cf6bff8e.png align="center")

Before flow goes to Generate JWS policy it should pass through the Assing message-1 policy. So drag it to the first

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700573719239/73a43de2-9232-4b8f-804f-86c69dcf84df.png align="center")

The assign message-1 policy code should look like the one below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <AssignVariable>
        <Name>private.secretkey</Name>
        <Ref>request.queryparam.key1</Ref>
    </AssignVariable>
    <!--afetr this we need to remove this query param from the request-->
    <Remove>
        <QueryParams>
            <QueryParam name="key1"/>
        </QueryParams>
    </Remove>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- type="response". bcz policy is in response flow-->
    <AssignTo createNew="false" transport="http" type="response"/>
</AssignMessage>
```

To send the response back to the client we construct a JSON message to display the generated token to the client.  
Proxy endpoint - postflow - response - add assignh message-2 policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700574204113/1e2c48a6-3512-466d-9dbe-ad7e5dcf43b3.png align="center")

The assign message-2 policy code should look like the one below.

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
                "JWS_TOKEN": "{jwstokenvalue}"
            }
        </Payload>
        <Path/>
    </Set>
    
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- type="response". bcz policy is in response flow-->
    <AssignTo createNew="false" transport="http" type="response"/>
</AssignMessage>
```

Save and deploy  
see the output: by sending a POST request with key1=code4545 in param, with user1 = and in the header and with body content.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700574682624/fe6e6eb2-7f3e-4fc9-b748-8586af95a217.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700574740869/5e4bc096-3ec3-4bf7-a50f-b9fcf4227c9c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700574875039/03ea6e40-273d-48e7-a74b-af51c45d7b71.png align="center")

Let’s decode the encoded token.  
https://jwt.io

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700575024658/6081d7bc-7dd9-4020-95c0-3ab1df31cda2.png align="center")

send the request without payload. before that modify the Generate JWS policy code.  
make detach content as true.  
the Generate JWS policy code looks like below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<GenerateJWS name="Generate-JWS-1">
    <DisplayName>Generate JWS-1</DisplayName>
    <Algorithm>HS256</Algorithm>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <CriticalHeaders/>
    <SecretKey>
        <Value ref="private.secretkey"/>
    </SecretKey>
    <!-- Payload we will pass it from the request-->
    <Payload ref="request.content"/>
    <!-- true, we can detach the content(means we are not adding any content while requesting
    false, meand we are attaching the content-->
    <DetachContent>true</DetachContent>
    <!-- can claim the additional headers name from client like user name-->
    <AdditionalHeaders>
        <Claim name="issuedby" ref="request.header.user1"/>
    </AdditionalHeaders>
    <!-- generated token is stored in variable "jwstokenvalue"-->
    <OutputVariable>jwstokenvalue</OutputVariable>
</GenerateJWS>
```

Save and deploy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700575351869/155b51de-5a5e-4564-acaf-a3e281ab554b.png align="center")

Let’s decode the generated token.  
https://jwt.io

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700575400563/f9d29d34-d1ed-40c9-b17c-8d655898c6a0.png align="center")

Again make detach content as false. save and deploy and see the output.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700579278058/de5d4e1c-d7bc-4b09-9b47-d65967e5cf47.png align="center")

---

One more API proxy to verify the JWS.  
create no target proxy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700575475443/0ec34f81-e550-49d3-8d45-81e6a1415f24.png align="center")

Add a policy to verify the token.  
proxy endpoint - preflow - request - add Verify JWS policy

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700575578323/9f5f59dd-76b9-4253-b97d-f282964c9b2f.png align="center")

Verify JWS policy code should look like below. and save it.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<VerifyJWS name="Verify-JWS-1">
    <DisplayName>Verify JWS-1</DisplayName>
    <!--we should use the same algorithm and same secret key as we used to generate th JWS token-->
    <Algorithm>HS256</Algorithm>
    <SecretKey>
        <Value ref="private.secretkey"/>
    </SecretKey>
    <!-- source: we are gettign JWS token from request- form parameter-->
    <Source>request.formparam.JWS</Source>
    <!-- we will use detached content in Generate JWS part so we no need to ude that
    <DetachedContent>false</DetachedContent>
    -->
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <KnownHeaders/>
    <IgnoreCriticalHeaders/>
</VerifyJWS>
```

we need to get the private.secretkey from the client request through the query parameter  
proxy endpoint - preflow - request - add the assign message-1 policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700576138314/a220ca3a-bd5a-4026-bed4-42474404efdc.png align="center")

This policy should be executed before the Verify JWS policy, so drage it to first.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700576177712/16ef9679-d0e5-409a-8f60-ad3cc889a566.png align="center")

The assign message-1 policy code should look like the one below. save

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Assign-Message-1">
    <DisplayName>Assign Message-1</DisplayName>
    <Properties/>
    <AssignVariable>
        <Name>private.secretkey</Name>
        <Ref>request.queryparam.key1</Ref>
    </AssignVariable>
    <!--afetr this we need to remove this query param from the request-->
    <Remove>
        <QueryParams>
            <QueryParam name="key1"/>
        </QueryParams>
    </Remove>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

Send a success response to the client back, by instructing JSON code, using assign message policy.  
proxy endpoint - postflow - response - add assign message-2 policy.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700576592760/a95894b6-488e-4409-af40-621c21b9fef3.png align="center")

we need to add one condition before executing assign message policy-2 in postflow response.  
so update the proxy endpoint(default) code.  
proxy endpoint(default) code looks like below. SAVE

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>Assign-Message-1</Name>
            </Step>
            <Step>
                <Name>Verify-JWS-1</Name>
            </Step>
        </Request>
        <Response/>
    </PreFlow>
    <Flows/>
    <PostFlow name="PostFlow">
        <Request/>
        <Response>
            <!--JWS.failed='' (failed = 'blank' means verification successfull. if it is true means verification unsuccessfully)-->
            <Step>
                <Condition>JWS.failed=''</Condition>
                <Name>Assign-Message-2</Name>
            </Step>
        </Response>
    </PostFlow>
    <HTTPProxyConnection>
        <BasePath>/jwsverify</BasePath>
    </HTTPProxyConnection>
    <RouteRule name="noroute"/>
</ProxyEndpoint>
```

Assign message policy-2 code looks like below.

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
                "Status": "verified successfully"
            }
        </Payload>
        <Path/>
    </Set>
    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
    <!-- type="response". bcz policy is in response flow-->
    <AssignTo createNew="false" transport="http" type="response"/>
</AssignMessage>
```

save and deploy  
output  
send a request with a secret key(key1=code4545), the token generated in the first proxy will pass it from the form parameter of the body.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700577271315/278e2abd-eec6-477f-bd21-9ff5c3c9cea6.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1700579332625/0d6af428-6144-49b9-877a-8e721e25994c.png align="center")