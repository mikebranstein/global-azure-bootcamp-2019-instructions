## Securing REST APIs

In this chapter, you'll learn how to:
- Use Azure API Management to provide a secure wrapper around your REST APIs
- Restrict access to Web Apps, only allowing API Management to communicate  

### Overview

At the end of the previous chapter, you had successfully deployed a REST API to an Azure Web App. The REST API had also integrated Swagger to let other know how to consume the API. 

Deploying the REST API was a great accomplishment. 

Deploying the REST API without a security layer could be problematic. 

In this chapter, you'll work to address the API security layer issue with an Azure service called API Management.

> **What is API Management?**
>
> API Management (APIM) helps organizations publish APIs to external, partner, and internal developers to unlock the potential of their data and services. Businesses everywhere are looking to extend their operations as a digital platform, creating new channels, finding new customers and driving deeper engagement with existing ones. API Management provides the core competencies to ensure a successful API program through developer engagement, business insights, analytics, security, and protection. You can use Azure API Management to take any backend and launch a full-fledged API program based on it.

Sounds pretty cool, right? It is, and it's easy. 

### Creating an API Management Service

Creating an API Management service in Azure takes a little bit of time, so let's get started.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating an API Management Service
</h4>

In the Azure portal, create a new *API Management Resource*. Choose a unique name for your resource. For other details, see below:

<img src="images/chapter5/apim-create.png" />

Click *Create*, then wait. It will take a while to deploy API Management - maybe up to 10 minutes, so this is a great opportunity to learn about securing your API project.

This concludes the exercise. 

<div class="exercise-end"></div> 

### Restricting Access to Web Apps

As I've discussed earlier, exposing a REST API service to the entire internet without any controls/restrictions in place isn't a great idea. 

Let's learn how we can restrict access through the Azure portal.


<h4 class="exercise-start">
    <b>Exercise</b>: Exploring Access Restrictions on Web Apps
</h4>

Navigate to your deployed API project and find the *Networking* section of the Web App service. At the bottom of the page, click on the *Configure Access Restrictions* link:

<img src="images/chapter5/access-restrictions.png" />

On the Access Restrictions page, you have the ability to add firewall rules that restrict which IP addresses can access this service. This provides you with an easy way to ensure only certain individuals (or other Azure services) have access to your APIs.

<img src="images/chapter5/def-access-restrictions.png" />

As you can see, all internet traffic is allowed to access my API web app. 

When your API Management Service is provisioned, it will have a Virtual IP address assigned to it - with that IP address, you'll be able to create a firewall rule that only allows traffic from API Management. 

We'll revisit this page in the future. For now, take a break until API Management has finished deploying. Read through the next section as you're waiting.

This concludes the exercise. 

<div class="exercise-end"></div>


### Learning about API Management

Before we can start using the APIM service, there's a few concepts you should understand.

To use API Management, administrators create APIs. Each API consists of one or more operations, and each API can be added to one or more products. To use an API, developers subscribe to a product that contains that API, and then they can call the API's operation, subject to any usage policies that may be in effect. Common scenarios include:

- **Securing mobile infrastructure** by gating access with API keys, preventing DOS attacks by using throttling, or using advanced security policies like JWT token validation.
- **Enabling ISV partner ecosystems** by offering fast partner onboarding through the developer portal and building an API facade to decouple from internal implementations that are not ripe for partner consumption.
- **Running an internal API program** by offering a centralized location for the organization to communicate about the availability and latest changes to APIs, gating access based on organizational accounts, all based on a secured channel between the API gateway and the backend.

The system is made up of the following components:

- **The API gateway** is the endpoint that:
   - Accepts API calls and routes them to your backends.
   - Verifies API keys, JWT tokens, certificates, and other credentials.
   - Enforces usage quotas and rate limits.
   - Transforms your API on the fly without code modifications.
   - Caches backend responses where set up.
   - Logs call metadata for analytics purposes.

- **The Azure portal** is the administrative interface where you set up your API program. Use it to:
   - Define or import API schema.
   - Package APIs into products.
   - Set up policies like quotas or transformations on the APIs.
   - Get insights from analytics.
   - Manage users.

- **The Developer portal** serves as the main web presence for developers, where they can:
   - Read API documentation.
   - Try out an API via the interactive console.
   - Create an account and subscribe to get API keys.
   - Access analytics on their own usage.

#### APIs and operations

APIs are the foundation of an API Management service instance. Each API represents a set of operations available to developers. Each API contains a reference to the back-end service that implements the API, and its operations map to the operations implemented by the back-end service. Operations in API Management are highly configurable, with control over URL mapping, query and path parameters, request and response content, and operation response caching. Rate limit, quotas, and IP restriction policies can also be implemented at the API or individual operation level.

#### Products
Products are how APIs are surfaced to developers. Products in API Management have one or more APIs, and are configured with a title, description, and terms of use. Products can be Open or Protected. Protected products must be subscribed to before they can be used, while open products can be used without a subscription. When a product is ready for use by developers, it can be published. Once it is published, it can be viewed (and in the case of protected products subscribed to) by developers. Subscription approval is configured at the product level and can either require administrator approval, or be auto-approved.

Groups are used to manage the visibility of products to developers. Products grant visibility to groups, and developers can view and subscribe to the products that are visible to the groups in which they belong.

#### Groups

Groups are used to manage the visibility of products to developers. API Management has the following immutable system groups:

- **Administrators** - Azure subscription administrators are members of this group. Administrators manage API Management service instances, creating the APIs, operations, and products that are used by developers.

- **Developers** - Authenticated developer portal users fall into this group. Developers are the customers that build applications using your APIs. Developers are granted access to the developer portal and build applications that call the operations of an API.

- **Guests** - Unauthenticated developer portal users, such as prospective customers visiting the developer portal of an API Management instance fall into this group. They can be granted certain read-only access, such as the ability to view APIs but not call them.

In addition to these system groups, administrators can create custom groups or leverage external groups in associated Azure Active Directory tenants. Custom and external groups can be used alongside system groups in giving developers visibility and access to API products. For example, you could create one custom group for developers affiliated with a specific partner organization and allow them access to the APIs from a product containing relevant APIs only. A user can be a member of more than one group.

#### Developers

Developers represent the user accounts in an API Management service instance. Developers can be created or invited to join by administrators, or they can sign up from the Developer portal. Each developer is a member of one or more groups, and can subscribe to the products that grant visibility to those groups.

When developers subscribe to a product, they are granted the primary and secondary key for the product. This key is used when making calls into the product's APIs.

#### Policies

Policies are a powerful capability of API Management that allow the Azure portal to change the behavior of the API through configuration. Policies are a collection of statements that are executed sequentially on the request or response of an API. Popular statements include format conversion from XML to JSON and call rate limiting to restrict the number of incoming calls from a developer, and many other policies are available.

Policy expressions can be used as attribute values or text values in any of the API Management policies, unless the policy specifies otherwise. Some policies such as the Control flow and Set variable policies are based on policy expressions. 

#### Developer portal

The developer portal is where developers can learn about your APIs, view and call operations, and subscribe to products. Prospective customers can visit the developer portal, view APIs and operations, and sign up. The URL for your developer portal is located on the dashboard in the Azure portal for your API Management service instance.

You can customize the look and feel of your developer portal by adding custom content, customizing styles, and adding your branding.

### Creating your first API

By now, your APIM service should have deployed. If not, take a break and continue once it's ready.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating your first API
</h4>

Earlier in this chapter you learned about APIM and how you can create an API that talks to a backend API. In this exercise, we'll be creating an APIM API that references your Contoso University REST API hosted in Azure. 

Navigate to your APIM service and click the *APIs* option. Click the box *OpenAPI* under the *Add a new API* heading:

<img src="images/chapter5/add-api.png" class="img-override" />

> **Swagger Specifications and OpenAPI**
>
> You may recall that Swagger Specifications were renamed to OpenAPI Specifications. You'll also recall that we added Swagger definitions to our API project. The OpenAPI option you just selected will let us reference the swagger.json files produced by Swagger to create an API in APIM automatically.

Click the *Full* link at top and complete the necessary fields, including:
- OpenAPI specification: URL to your swagger.json file hsoted on your API web app
- Display Name
- Name
- URL Scheme: restrict to HTTPS
- API URL suffix: set this to *contoso*, which is the URL route developers will use to access your API
- Version this API?: check the box

<img src="images/chapter5/openapi.png" class="img-override" />

It's important that you version your APIs. APIM provides many ways of versioning your API (via URL route, HTTP Headers, and query strings). I prefer the URL route, but what you choose is ultimately up to you.

At the very bottom, you can see a preview URL of what developers will use as the base URL when communicating with your API. For example, mine says https://contoso-university-meb.azure-api.net/contoso/v1. This base URL will be combined with my Azure-hosted API web app URL routing scheme (i.e., /api/Courses and /api/Students), so the final URL other developers will use will look like this: https://contoso-university-meb.azure-api.net/contoso/v1/api/Courses. 

When an HTTP request is made to that URL, the APIM gateway willintercept the call, very the caller has permissions to access the API, then forward the request to the backend API (in this case https://contoso-web-api-meb.azurewebsites.net/api/Courses). When the backend API responds, it is routed back through the APIM gateway and back to the original caller.

Click *Create* to finish.

When APIM finishes creating the API gateway route, you're presented with detailed HTTP operations the APIM servie detected (via your Swagger file).

<img src="images/chapter5/api-v1.png" class="img-override" />

You can customize the data/routing/behavior of your API here, if you choose. This is out of scope for our workshop, but I encourage you to explore on your own.

#### Finishing the configuration

Click the *Settings* tab of the newly-created API, and type the backend URL of your API into the *Web service URL* field. For me this was *https://contoso-web-api-meb.azurewebsites.net*:

<img src="images/chapter5/web-url.png" class="img-override" />

Also select *Unlimited* from the *Products* area.

Click *Save*.

> **Unlimited Product**
>
> The unlimited product is a pre-created APIM product that gives developers unlimited access (without network throttling or other policies applied). It is also a product that requires Administrative appoval. So, when someone subscribes to the product, a request is sent to the API administrator, and it must be approved before the developer obtains an API subscription key.

#### Testing your APIM gateway

Next, click the *Test* tab, then select the GET /api/Course command. 

<img src="images/chapter5/test.png" class="img-override" />

Here, you'll see a test client to testing your API. The *Request URL* is the URL that an HTTP GET will be sent to when you click the *Send* button at the bottom. 

Click the *Send* button and you should see an HTTP 200 Success message:

<img src="images/chapter5/success.png" class="img-override" />

#### Testing with a 3rd party tool

It's normally a good idea to test access to your API from another tool. I happen to like Postman. Visit [https://www.getpostman.com/](https://www.getpostman.com/) to download Postman. 

After downloading and installing, create an HTTP GET request, and fill in the URL of your APIM API. For example, mine was https://contoso-university-meb.azure-api.net/contoso/v1/api/Courses.

Click *Send*.

<img src="images/chapter5/postman-no-auth.png" class="img-override" />

As you can see, the response at the bottom says:

```json
{
    "statusCode": 401,
    "message": "Access denied due to missing subscription key. Make sure to include subscription key when making requests to an API."
}
```

This is exactly what I'd expect, because I didn't supply a subscription key. When we placed the API behind the APIM gateway, it requires we submit a subscription key in the HTTP request header.

Let's try this again. We'll need to add an HTTP header named *Ocp-Apim-Subscription-Key* with a subscription key as the value. 

To get a subscription key, navigate back to the APIM service, and click *Developer portal*. When the portal loads, you will be logged in as the portal Administrator. 

Click your name in the upper-right and navigate to your profile. On the profile page, locate the Unlimited subscription (this is auto-assigned to you as an administrator), and click *Show*. Copy the subscription key and save it.

<img src="images/chapter5/portal.png" class="img-override" />

Return to Postman and create the key/value pair of:
- Ocp-Apim-Subscription-Key: {your subscription key}

Click *Send*.

<img src="images/chapter5/200ok.png" class="img-override" />

This time, you'll notice the request was processed by the gateway, forwarded to the backend API, and returned a list of courses.

#### Challenge

If you're up for a challenge, find a friend in the workshop that has also deployed APIM. Pretend you're a developer wanting access to their API. 

Get the URL of their developer portal, navigate to it, create a login, request an Unlimited product subscription, and make a successful request to their API.

#### Challenge #2

If you're up for a second challenge, customize the look and feel of your portal. 

This concludes the exercise. 

<div class="exercise-end"></div>

### Restrict Access to the Backend API

You'll recall that we had reviewed the Access Restrictions section of your backend API web app earlier in this chapter. Let's follow-through with our plans to prevent anyone from casually querying the backend directly by resticting access to the APIM virtual IP address.

<h4 class="exercise-start">
    <b>Exercise</b>: Restricting Backend API Access
</h4>

Navigate to the *Overview* section of your APIM service and locate the *Virtual IP (VIP) Addresses*:

<img src="images/chapter5/vip.png" class="img-medium" />

Copy the VIP.

Next, navigate back tot he Access Restrictions area of your backend API web app. Add an *Allow* rule for the VIP you copied:

<img src="images/chapter5/ip-restriction.png" class="img-medium" />

Click *Add rule*. You'l notice that the restrictions now show that all traffic is denied with the exception of the APIM VIP you just set:

<img src="images/chapter5/deny.png" class="img-medium" />

#### Testing the restrictions

To test the changes, navigate to your backend API URL directly from your browser. You should receive an error message.

<img src="images/chapter5/error403.png" class="img-medium" />

Now, return to Postman and re-run your last HTTP GET request and verify that it succeeds.

This concludes the exercise. 

<div class="exercise-end"></div>

In this chapter, you learned:
- How to use APIM to greate a secure API gateway for your REST API.
- How to restrict access to your REST API so only APIM can communicate with it.

