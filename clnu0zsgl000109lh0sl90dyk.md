---
title: "Introduction to Apigee"
seoTitle: "Introduction to Apigee"
seoDescription: "In this blog you will understand, the API development lifecycle, Why we need Apigee, What is an Apigee, and the Components of Apigee."
datePublished: Tue Oct 17 2023 07:51:34 GMT+0000 (Coordinated Universal Time)
cuid: clnu0zsgl000109lh0sl90dyk
slug: introduction-to-apigee
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697262095964/fe5befcb-74da-436c-a3aa-dadda1e21449.png
tags: introduction-to-apigee

---

## API Development Lifecycle:

To create an API we need to follow the API development life cycle. Which has three phases:  
1\. Design phase  
2\. Implement phase  
3\. Management phase

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697262844086/1f83f13e-174b-46ab-83ba-44c8b8e957f5.png align="center")

1. **Design Phase**: In this phase, API has to be designed, and simulated, tested to get feedback about the API design. if the API design is incorrect we will get negative feedback, then we need to validate the feedback and redesign the API until we get positive feedback.  
    Designing an API Is nothing but the creation of an API specification file.
    
2. **Implement Phase:** The next phase is to build the API, once we get positive feedback about the design. API implementation has to be tested if the test fails, we have to modify the functionality until all the test cases are successfully passed.  
    Building an API is nothing but creating API implementation.
    
3. **Management phase:** This phase is to manage the APIs by versioning the APIs, Securing the APIs, Deploying, monitoring them, Analyzing the data coming from them, Troubleshooting if something goes wrong, and managing the traffic by scaling the application up and down.
    

The entire outer circle corresponds to API management, This API management is done by using Apigee.

## What is an Apigee?

Apigee is an API management tool, used to manage the APIs.  
Apigee can also perform the Design phase. In some scenarios, we can also perform the Implementation phase.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697357918290/fc3d34ee-63ba-4b82-b509-e58b0d501f18.png align="center")

* Apigee is a platform to Develop and Manage APIs.
    
* It creates a proxy layer between the Backend and the client.
    
* A client(Developer) has to call the Proxy service to get the services from the backend(*where actual APIs are stored*).
    
* The client is not exposed to the backend directly for security reasons.
    
* Once the proxy receives the request, it validates it before passing it to the Backend service.
    
* All the security features are applied at the proxy layer using policies, it will protect the Backend service from security attacks.
    
* Apigee makes it easy for developers to consume their APIs.
    
* Backend service implementation changes will not affect the calling application.
    
* It provides monitoring, analytics, and developer portal features.
    

### Components of Apigee

1. **Environment**: It is the runtime context to deploy API proxies. API proxies are in a running state once it is deployed to an environment. Proxies can deployed in multiple environments.
    
2. **Environment groups:** It is a collection of environments.  
    \- hostnames are defined on the environment groups.  
    Environment groups --&gt; Hostnames --&gt; Environments (*ex. prod, dev*)  
    To access each (environment --&gt; proxy), it should be associated with hostnames, Hostnames should be added inside the environment groups.  
    We can add multiple hostnames in each group.  
    We can't add the same hostnames to multiple environment groups.
    
3. **API Proxy:** Api proxies act as interfaces to the developers to access backend services.  
    Backend services are secured by using API proxies.  
    All the security and rate-limiting policies are applied to the API proxy before passing it to the backend.
    
    In this type of architecture, the backend service URL is not shared with the consumers, the consumer must hit the API proxy to he the service from the backend.  
    Any changes to the backend service URL are not shared with the consumers as the services are exposed through proxy endpoints.  
    Consumers are unaware of the backend service changes, for example in the database.  
    API tools provide out-of-the-box policies to apply to the API proxies.
    
4. **API Products:** It is a collection of API proxies with quota settings.  
    It helps to bundle related API proxies in one APU product.
    
    The developer can subscribe to the required API product and the plan. They should create an App to subscribe to the product. They can subscribe to multiple products using the same App.  
    Apigee has provided a Developer portal to register and create an App for the Developers.
    
5. **Developer:** The developer is an entity, which is uniquely identified by Email address.  
    Apigee provides 2 types of solutions to developers
    
    Self-service:
    
    \-- Here developer registers himself in the developer portal  
    \-- Create an APP  
    \-- Subscribe to the APIU products with a suitable rate plan.  
    Manual service:  
    \--Here the API team creates the Developer and Appa in the Apigee management UI.
    
6. **Developer portal:** In the Developer portal, API developers list their APIs, and the same can be utilized by the APP developers.  
    These are details available in the portal.  
    \-- API documentation.  
    \-- What kind of data does the API receive, and what kind of data does it respond back?  
    \-- What are the operations( GET, POST, PUT, DELETE) supported.  
    \-- Sample data to test and analyze the API.  
    \-- openAPU specification details.  
    \-- Rate plan to use the APIs.
    
    Apigee provides these 3 Developer portal solutions.  
    \-- Apigee integrated portal.  
    \-- Drupal 9 modules.  
    \-- Do-it-yourself
    
    We will use the Apigee integrated portal.
    

### Flows and Flow Sequence

* Using Flows, you can configure the behavior of the API by applying policies at the right place.
    
* During the processing of the request, different kinds of flows are executed with policies attached to them.
    
* Different types of Flows  
    \-- Pre Flow  
    \-- Conditional Flow  
    \-- Post Flow  
    \-- Post Client Flow
    
* When we are working with API proxies, Let's divide the API proxy into 2 parts.  
    \-- Proxy Endpoint: One part of the API proxy that is exposed to the Client  
    \-- Target Endpoint: Another part of the backend service and get the required data/information.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697531059134/2432069d-e9fd-4c25-879d-bdfa6949355a.png align="center")

Client request for an API, That request hits the proxy endpoint Preflow. Then it goes to conditional flows(*Conditional flow is optional, we will add based on our need.*) if any, and if the condition is satisfied, it goes to post flow request. once it is done, it goes to the target endpoint request flow, then it executes Preflow, condition flow, and postflow. After this, it will hit the backend service based on the target endpoint we set.  
Backend replies back to target endpoint response flow. Again here goes with Preflow, conditional flow if any, then postflow. Then it hits the proxy endpoint response flow with Preflow, conditional flow if any, and postflow. The response back to the client flow.  
Post-client flow: we use this to add some login mechanisms.