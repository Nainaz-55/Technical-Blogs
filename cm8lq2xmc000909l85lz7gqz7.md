---
title: "Apigee Project : Enterprise API Gateway Implementation using Apigee X"
seoTitle: "Enterprise API Gateway Architecture & Management using Apigee"
seoDescription: "Enterprise API Gateway Architecture & Management using Apigee"
datePublished: Sun Mar 23 2025 14:17:45 GMT+0000 (Coordinated Universal Time)
cuid: cm8lq2xmc000909l85lz7gqz7
slug: blog-series-enterprise-api-gateway-implementation-using-apigee-x
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744218970379/ea1d25e3-2c5d-4c6f-b2ed-8bc4f4b71232.jpeg
tags: jwt, apigee, json-web-token-jwt-policy, apigee-assignment, apigee-projects, enterprise-api-gateway, apigee-x

---

#### **Project Overview**

We will design, implement, and manage an enterprise-grade **API Gateway** using **Apigee X**. Provides a **multi-cloud SaaS solution** for financial services, requiring a **secure, scalable, and high-performance** API management platform.

The project will involve:

1. **Designing and implementing** an Apigee API Gateway for managing public and private APIs.
    
2. **Security enforcement** through OAuth 2.0, JWT validation, and API keys.
    
3. **Performance optimization** using caching, rate limiting, and load balancing.
    

---

# Part 1: Setting Up Apigee X and Deploying Your First API Proxy

### **Introduction**

In this multi-part blog series, we will implement an **Enterprise API Gateway** using **Apigee X** to meet advanced requirements such as security enforcement,and traffic management. We will use **free public APIs** as backend targets, test with **Postman and cURL**, and implement **OAuth 2.0, JWT validation, API keys, and monitoring**.

## **Step 1: Setting Up Apigee X in Google Cloud**

### **1.1 Prerequisites**

Before we start, ensure you have:

* A **Google Cloud Project** with billing enabled
    
* **Apigee X** provisioned (Follow Google’s guide)
    
* A domain mapped for Apigee (or use default endpoints)
    
* Installed **gcloud CLI**
    

### **1.2 Create an Apigee Organization**

```bash
gcloud config set project [PROJECT_ID]
gcloud apigee organizations create --project=[PROJECT_ID] --authorized-network=default
```

### **1.3 Enable Required APIs**

```bash
gcloud services enable apigee.googleapis.com runtimeconfig.googleapis.com
```

## **Step 2: Creating an API Proxy**

### **2.1 Choose a Free Target API**

For testing, we will use the **JSONPlaceholder API**, a free public REST API for mock testing.

* Base URL: `https://jsonplaceholder.typicode.com`
    
* Sample endpoint: `/posts/1`
    

### **2.2 Create an API Proxy in Apigee X**

1. Go to the Apigee Console
    
2. Navigate to **Develop &gt; API Proxies**
    
3. Click **\+ Create Proxy**
    
4. Select **Reverse Proxy**
    
5. Enter:
    
    * **Proxy Name**: `jsonplaceholder-proxy`
        
    * **Base Path**: `/json`
        
    * **Target URL**: `https://jsonplaceholder.typicode.com`
        
6. Click **Next**, then **Deploy**
    

### **2.3 Test the API Proxy**

#### **Using Postman**

* Open **Postman**
    
* Make a `GET` Request to:
    
    #### **Using cURL**
    
    curl -X GET "https://\[APIGEE\_HOST\]/json/posts/1" -H "Accept: application/json"
    

Expected Response:

```bash
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit..."
}
```

---

---

# Part 2: Implementing API Security in Apigee X

### **Introduction**

Now that we have deployed our first API proxy, it’s time to secure our APIs. In this part, we will implement **OAuth 2.0 authentication, API key validation, and JWT verification** to ensure secure access control.

## **Step 1: Setting Up an OAuth 2.0 Proxy**

### **1.1 Create a New OAuth 2.0 Proxy**

1. Navigate to **Develop &gt; API Proxies**
    
2. Click **\+ Create Proxy**
    
3. Select **No Proxy**
    
4. Enter:
    
    * **Proxy Name**: `oauth-proxy`
        
    * **Base Path**: `/oauth`
        
5. Click **Next**, then **Deploy**
    

### **1.2 Apply OAuth 2.0 Token Generation Policy**

1. Open **oauth-proxy** in Apigee
    
2. Edit the **PreFlow** in `ProxyEndpoint` and add:
    
    ```xml
    <OAuthV2 name="OAuth-v2-Policy">
        <Operation>GenerateAccessToken</Operation>
        <SupportedGrantTypes>
            <GrantType>client_credentials</GrantType>
        </SupportedGrantTypes>
        <ExpiresIn>3600</ExpiresIn>
        <GenerateResponse enabled="true"/>
    </OAuthV2>
    ```
    
    ### **1.3 Test OAuth 2.0 Token Generation**
    
    #### **Using Postman**
    
3. Make a `POST` request to:
    
    ```bash
    https://[APIGEE_HOST]/oauth/token
    ```
    
4. In **Body**, add:
    
    ```bash
    grant_type=client_credentials&client_id=[YOUR_CLIENT_ID]&client_secret=[YOUR_CLIENT_SECRET]
    ```
    
5. Expected Response:
    
    ```bash
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
    ```
    

#### **Using cURL**

```bash
curl -X POST "https://[APIGEE_HOST]/oauth/token" -d "grant_type=client_credentials&client_id=[YOUR_CLIENT_ID]&client_secret=[YOUR_CLIENT_SECRET]"
```

## **Step 2: Enforcing API Key Authentication**

### **2.1 Generate an API Key**

1. Navigate to **Apigee Console &gt; Publish &gt; API Products**
    
2. Click **\+ Create Product**
    
3. Set the **Product Name** as `TestProduct`
    
4. Add `/json/*` as the API resource path
    
5. Enable **Require API Key**
    
6. Save and deploy the product
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744287550723/a9370281-ba77-46e5-b7f3-6c025bafb24e.png align="center")
    

### **2.2 Create a Developer & App**

1. Go to **Publish &gt; Developers**, click **\+ Create Developer**
    
2. Create an app under **Publish &gt; Apps**, and associate it with `TestProduct`
    
3. Copy the generated **API Key**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744287648444/d32f0ac1-acd6-47f5-a51b-91e9b20186a3.png align="center")
    

### **2.3 Apply API Key Verification Policy**

1. Open **jsonplaceholder-proxy**
    
2. Edit the **PreFlow** in `ProxyEndpoint` and add:
    
    ```xml
    <VerifyAPIKey name="Verify-API-Key">
        <APIKey ref="request.header.x-api-key"/>
    </VerifyAPIKey>
    ```
    
3. Save and deploy the proxy
    

### **2.4 Test API Key Enforcement**

#### **Using Postman**

* Make a `GET` request:
    
    ```bash
    https://[APIGEE_HOST]/json/posts/1
    ```
    
* Add `x-api-key` in **Headers** with the copied API key
    
* Expected Response: **200 OK**
    
* Without API key: **403 Forbidden**
    

#### **Using cURL**

```bash
curl -X GET "https://[APIGEE_HOST]/json/posts/1" -H "x-api-key: [YOUR_API_KEY]" -H "Accept: application/json"
```

---

---

# Part 3: Implementing JWT Authentication and mTLS Security

### **Introduction**

In this part, we will enhance the security of our Apigee X API Gateway by implementing **JWT (JSON Web Token) authentication**. JWT ensures that only authenticated clients access our APIs.

## **Step 1: Implementing JWT Authentication**

### **1.1 Enable JWT Verification in Apigee**

1. Open **Apigee Console** and navigate to **Develop &gt; API Proxies**.
    
2. Select `jsonplaceholder-proxy`.
    
3. Edit the **PreFlow** in `ProxyEndpoint`.
    
4. Add the following policy to verify JWT tokens:
    
5. Before Verify JWT policy add an assign message policy to extract the secret.
    

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="Extract-Secret">
    <DisplayName>Extract-Secret</DisplayName>
    <AssignVariable>
        <Name>private.secretkey</Name>
        <Ref>request.queryparam.secret</Ref>
    </AssignVariable>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

The `VerifyJWT` policy

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<VerifyJWT continueOnError="false" enabled="true" name="Verify-JWT-Token">
    <DisplayName>Verify-JWT-Token</DisplayName>
    <Algorithm>HS256</Algorithm>
    <Source>request.header.Authorization</Source>
    <SecretKey>
        <Value ref="private.secretkey"/>
        <Id/>
    </SecretKey>
    <Issuer>https://dev-zao4x20.us.auth0.com</Issuer>
    <Audience>audience1</Audience>
    <AdditionalClaims>
        <Claim name="role">admin</Claim>
        <Claim name="scope">read:posts</Claim>
    </AdditionalClaims>
</VerifyJWT>
```

### **1.2 Create a Separate Proxy for JWT Token Generation**

#### **Steps to Create a New Proxy for JWT Generation**

1. **Go to Apigee Console** → Navigate to **Develop &gt; API Proxies**.
    
2. **Create a New Proxy**:
    
    * Click **\+ Create Proxy**.
        
    * Select **Reverse Proxy**.
        
    * Set **Proxy Name** as `generate-jwt-proxy`.
        
    * Set **Base Path** as `/generate-jwt`.
        
    * Target Endpoint: Select **No Target (Service Callout Only)**.
        
    * Click **Next** and deploy.
        
3. **Add GenerateJWT Policy**:
    
    * Go to **Develop &gt; Policies**.
        
    * Add a new policy:
        
        * Type: `GenerateJWT`
            
        * Name: `Generate-JWT-Token`
            
    * Use the following configuration:\\
        
        ```xml
        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <GenerateJWT continueOnError="false" enabled="true" name="Generate-JWT-Token">
            <DisplayName>Generate-JWT-Token</DisplayName>
            <Algorithm>HS256</Algorithm>
            <SecretKey>
                <Value ref="private.secretkey"/>
                <Id/>
            </SecretKey>
            <Issuer>https://dev-zaao4x20.us.auth0.com</Issuer>
            <Subject>api-client</Subject>
            <Audience>audience1</Audience>
            <ExpiresIn>3600</ExpiresIn>
            <AdditionalClaims>
                <Claim name="role">admin</Claim>
                <Claim name="scope">read:posts</Claim>
            </AdditionalClaims>
            <OutputVariable>jwt_token</OutputVariable>
        </GenerateJWT>
        ```
        
4. **Accept Secret Key via Query Parameter**:
    
    * Add an AssignMessage policy before GenerateJWT to extract secret from query:
        
        ```xml
        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <AssignMessage continueOnError="false" enabled="true" name="Extract-Secret">
            <DisplayName>Extract-Secret</DisplayName>
            <AssignVariable>
                <Name>private.secretkey</Name>
                <Ref>request.queryparam.secret</Ref>
            </AssignVariable>
            <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
            <AssignTo createNew="false" transport="http" type="request"/>
        </AssignMessage>
        ```
        
5. **Return JWT in Response**:
    
    * Add an `AssignMessage` policy to return the JWT:
        
        ```xml
        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <AssignMessage continueOnError="false" enabled="true" name="Return-JWT">
            <Properties/>
            <Set>
                <Headers/>
                <QueryParams/>
                <FormParams/>
                <Verb>POST</Verb>
                <Payload contentType="application/json">
                    { 
                        "status": "generated JWT successfully",
                        "JWT_Token": {jwt_token}
                    }
        
                </Payload>
                <Path/>
            </Set>
            <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
            <AssignTo createNew="false" transport="http" type="request"/>
        </AssignMessage>
        ```
        
6. **Attach Policies to Flow**:
    
    * In `ProxyEndpoint` **PreFlow**, add the policies in sequence:
        
        ```xml
        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <ProxyEndpoint name="default">
            <PreFlow name="PreFlow">
                <Request>
                    <Step>
                        <Name>Extract-Secret</Name>
                    </Step>
                    <Step>
                        <Name>Generate-JWT-Token</Name>
                    </Step>
                    <Step>
                        <Name>Log-JWT-Token</Name>
                    </Step>
                    <Step>
                        <Name>Return-JWT</Name>
                    </Step>
                </Request>
                <Response/>
            </PreFlow>
            <Flows/>
            <PostFlow name="PostFlow">
                <Request/>
                <Response/>
            </PostFlow>
            <HTTPProxyConnection>
                <BasePath>/generate-jwt</BasePath>
            </HTTPProxyConnection>
            <RouteRule name="noroute"/>
        </ProxyEndpoint>
        ```
        
7. **Deploy the Proxy**.
    

### **1.3 Adding the Proxy to an Existing API Product**

1. **Go to Apigee Console** → **Publish &gt; API Products**.
    
2. Select the existing API Product where JWT should be included.
    
3. Click **Edit** → **Add Proxy**.
    
4. Select `generate-jwt-proxy`.
    
5. Click **Save** and **Deploy**.
    

### **1.4 Testing in Postman**

#### **Generate Token Request**

```bash
curl -X POST "https://[APIGEE_HOST]/generate-jwt?secret=client_secret" \
  -H "Content-Type: application/json"
```

#### **Expected Response**

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743871073029/e094dfd5-359d-4e1b-91cb-d9162e191eef.png align="center")

## **Step 2: Verifying JWT Token via Protected API**

#### **2.1 Call Protected Proxy with JWT Token**

```bash
curl -X GET "https://[APIGEE_HOST]/jsonplaceholder/posts/1?secret=client_secret" \
  -H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

```json
{
  "userId": 1,
  "id": 1,
  "title": "Sample JSONPlaceholder Post",
  "body": "This is a sample post body from JSONPlaceholder API."
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743871034545/53acc9f5-45e9-450f-b872-0848a52810ac.png align="center")

---

---

# Part 4: Implementing Rate Limiting, Response Caching, and Logging in Apigee X

These features help ensure **performance**, **security**, and **operational visibility** for your APIs.

### **Step 1: Add Spike Arrest (Rate Limiting)**

Spike Arrest prevents backend overloading by throttling requests.

#### 1.1 Add Spike Arrest Policy

In your `jsonplaceholder-proxy`, open the **PreFlow** under `ProxyEndpoint`. Add:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<SpikeArrest continueOnError="false" enabled="true" name="Apply-SpikeArres">
    <DisplayName>Apply-SpikeArres</DisplayName>
    <Rate>5ps</Rate>
</SpikeArrest>
```

#### 1.3 Test Rate Limiting in Postman

Use a script or Postman runner to send multiple rapid requests. After 5 requests per second, you'll receive:

```json
{
  "fault": {
    "faultstring": "Spike arrest violation. Allowed rate : 5ps",
    "detail": {
      "errorcode": "policies.ratelimit.SpikeArrestViolation"
    }
  }
}
```

### **Step 2: Enable Response Caching**

Apigee can cache responses to reduce latency and backend load.

#### 2.1 Add ResponseCache Policy

Add this in the `Response` flow of your proxy:

```xml
<ResponseCache name="Cache-JSONPlaceholder">
    <CacheKey>
        <KeyFragment>jsonplaceholder</KeyFragment>
        <KeyFragment ref="request.uri"/>
    </CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
        <TimeoutInSec>60</TimeoutInSec>
    </ExpirySettings>
</ResponseCache>
```

#### 2.2 Test the Cache

* Make a GET request.
    
* Then repeat the same request. The second response will be served from Apigee cache (check headers like `X-Apigee-Cache: HIT`).
    

### **Step 3: Add Message Logging for Observability**

#### 3.1 Add MessageLogging Policy

Go to `Develop > Policies`, create:

```xml
<MessageLogging name="Log-Request-Response">
    <Syslog>
        <Message>
            Request: {request.uri}, Token: {request.header.Authorization}, ResponseCode: {response.status.code}
        </Message>
        <Host>syslog.example.com</Host>
        <Port>514</Port>
    </Syslog>
</MessageLogging>
```

3.2 Attach to policy Flow should look like

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>Apply-SpikeArres</Name>
            </Step>
            <Step>
                <Name>Extract-Secret</Name>
            </Step>
            <Step>
                <Name>verify-api-key</Name>
            </Step>
            <Step>
                <Name>Verify-JWT-Token</Name>
            </Step>
        </Request>
        <Response>
            <Step>
                <Name>Log-Request-Response</Name>
            </Step>
            <Step>
                <Name>Cache-JSONPlaceholder</Name>
            </Step>
        </Response>
    </PreFlow>
    <Flows/>
    <PostFlow name="PostFlow">
        <Request/>
        <Response/>
    </PostFlow>
    <HTTPProxyConnection>
        <BasePath>/json</BasePath>
    </HTTPProxyConnection>
    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

### Summary

| Feature | Benefit |
| --- | --- |
| Spike Arrest | Protects the backend from floods |
| Response Caching | Reduces latency & backend load |
| Logging | Helps with debugging & auditing |

In this blog, we learnt to develop API gateway for Enterprise API!!