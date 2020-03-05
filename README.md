# Configure Axway API Management with external OAuth provider

- [Configure Axway API Management with external OAuth provider](#configure-axway-api-management-with-external-oauth-provider)
  - [Introduction](#introduction)
  - [Scenarios](#scenarios)
  - [External OAuth flow](#external-oauth-flow)
  - [Okta Endpoints](#okta-endpoints)
  - [Implementation steps -Scenario 1 (Client Credentials)](#implementation-steps--scenario-1-client-credentials)
    - [Register your application with Okta](#register-your-application-with-okta)
    - [Creating Custom Scopes](#creating-custom-scopes)
    - [Configure Okta Token Information Policy](#configure-okta-token-information-policy)
    - [APIManager Configuration](#apimanager-configuration)
    - [Testing](#testing)
  - [Implementation steps -Scenario 2 (Authorization code)](#implementation-steps--scenario-2-authorization-code)
    - [Register your application with Okta](#register-your-application-with-okta-1)
    - [Testing](#testing-1)

## Introduction

This document is intended to explain the implementation of external Oauth2 for authentication and authorization,by setting up API Manager/Gateway as Resource server and Okta as External Authorization server.

## Scenarios

This document details the steps required to access APIM protected API using External OAuth inbound security.

**Scenario 1 -- Client Credentials OAuth grant type**

Client credentials is recommended for machine-to-machine authentication. Application will need to securely store ClientID and Secret and pass to Okta in exchange for an access token.

**Scenario 2 \-- Authorization code grant type**

If you are building a server-side (or \"web\") application that is capable of securely storing secrets, then the authorization code flow is the recommended method for controlling access to it.

## External OAuth flow

-   The external OAuth is done with a 3d party IDP.

-   Client application registered in the IDP passes client id and secret to IDP.

-   IDP validates the id and secret , and generates token.

-   Subsequent request to IDP is happening through APIGW.

-   The client sends Access token in every subsequent calls for invoking APIs protected with OAuth.

-   APIGW makes Introspect Access Token API request to generate OAuth token attributes such as scope, clientid and valid. This helps to validate the access token to pass the API request call.

## Okta Endpoints

| **Endpoint** |  **Use** |
|----------------------|---------------|
| [/authorize](https://developer.okta.com/docs/api/resources/oidc/#authorize) | Interact with the resource owner and obtain an authorization grant.|
| [/token](https://developer.okta.com/docs/api/resources/oidc/#token) | This endpoint returns access tokens, ID tokens, and refresh tokens, depending on the request parameters.|
| [/introspect](https://developer.okta.com/docs/api/resources/oidc/#introspect) | Returns information about a token.|

[Okta doc]: <https://developer.okta.com/docs/api/resources/oidc>



## Implementation steps -Scenario 1 (Client Credentials)

### Register your application with Okta

- Create an account in Okta developer console

  <https://developer.okta.com/>

- Choose "Create Free account"


![image001](images/image001.png)

- Once creating your account, create a new application by choosing **Service** for client credentials grant type.

  ![](images/image004.jpg)

- Name the Application (eg. "My Service App") in the next step.

  ![](images/image005.png)

Application is setup now and client credentials are generated.

### Creating Custom Scopes

- Choose **API->Authorization Servers** and edit default Authorization Server.

  ![](images/image007.png)

  *Note the Issuer URI as it is different for every account.*

- Click on the **Add Scope** button to add a Custom Scope. In this example, **resource.READ** custom scope is added.

  ![](images/image009.png)

  ![](images/image010.png)



### Configure Okta Token Information Policy

- Add Okta certificate to Certificate repository.
  

  ![](images/image012.png)
  
- Here is the flow in APIGW which does validate access token.

  ![](images/image014.png)

- The following screenshots show the configuration of all the filters used in the policy.

  ![](images/image015.png)



​		![](images/image017.png)

​		

​		![](images/image018.png)

​		![](images/image020.png)



![](images/image022.png)		

------

![](images/image023.png)



​	![](images/image024.png)



​	![](images/image025.png)



- Add the policy to **Server Settings->API->Manager->Oauth Token information Policies**

  ![](images/image026.png)



### APIManager Configuration

- Virtualize an API in API Manager and protect it using OAuth (External) in Inbound Security

  ![](images/image028.png)

- Edit OAuth Security Profile and specify the properties as shown below,

  ![](images/image030.png)

  The attributes specified for **oauth.token.\*** are generated in the External Oauth policy.

  ![](images/image031.png)

- Register the client Application also in API Manager and specify the client id (copied from Okta developer console).

  ![](images/image032.png)

### Testing

- The following screen shows the *Postman* testing configuration.

  ![](images/image034.png)

  Scope setting: Custom scope defined when registering application in Okta

- Once you click **Request Token**, *Access token* is generated as shown in the following screen.

  ![](images/image036.png)

- Use the *Access token* to access the API protected by External Oauth.

  ![](images/image038.png)

  ------

  ![](images/image040.png)

​		

## Implementation steps -Scenario 2 (Authorization code)

### Register your application with Okta

- Follow steps given in 5.1 to create an account.


- Choose Add Applications

  ![](images/image042.png)

- Create a Web application

  ![](images/image044.png)

- Since testing is done through Postman, *Callback URI* is given as Postman callback URI.

  ![](images/image046.png)

- Client credentials are generated at this stage. Save the client credentials for future use.

  ![](images/image048.png)

- Also safe the *Issuer URI* from default Authorization Server.

  ![](images/image050.png)

- Follow steps given in 

  [sections]: CreatingCustomScopes	"Creating Custom Scopes"

   for Custom scope. 

  [section]: ConfigureOktaTokenInformationPolicy	"Configure Okta Token Information Policy"

   for configuring Okta token information policy and 

  [section]: APIManagerConfiguration	"API Manager Configuration"

  for API Manager configuration.


- Register the client application also in API Manager and specify the client id (copied from Okta developer console) as shown below.

  ![](images/image052.png)

### Testing

- The following screen shows the Postman testing configuration.

  ![](images/image054.png)

  **state** is an arbitrary alphanumeric string that the authorization server will return as reference ID when redirecting the user-agent back to the client. This is used to help prevent cross-site request forgery.

- Once **Request Token** is clicked and if the user does not have an existing session, this will open the Okta Sign-in Page as shown below

  ![](images/image056.png)

  Sign in with Okta account.

- Token is generated.

  ![](images/image058.png)

- Use the token to access the API.

  ![](images/image060.png)

  ------

  ![](images/image062.jpg)