# java-rest-api-b2c
This is a Java Spring Boot sample REST API that can use Azure AD B2C as its Identity Provider.

## Background
In the [samples for MSAL J](https://github.com/AzureAD/microsoft-authentication-library-for-java) there is a WebApp called [msal-b2c-web-sample](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples/msal-b2c-web-sample) that is a WebApp that uses Azure AD B2C for its authentication. The sample has the possibility of calling a WebAPI to illustrate the use of JWT access token. If you read the Microsoft documentation for B2C, the WebAPI mentioned is a node.js sample. If you are looking for a way to build a Java WebAPI instead, this is that sample.

## Where this sample is derived from
MSAL (Microsoft Authentication Library) is a client side library for aquiring tokens from OIDC/OAuth sources. It is not a library to use on the WebApi side to validate access tokens received. So if you're looking for a Microsoft MSAL sample for how to build a WebApi, you will not find it. But, you can easily create a WebApi out of the sample code in the MSAL samples called [msal-obo-sample](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples/msal-obo-sample). This sample demonstrates how you use the On-Behalf-Of authentication flow, but if you remove the OBO functionality that calls downstreams APIs, what remains is a WebApi and a WebApi that can be used together with Azure AD B2C. 

## How to test
First you must create your Azure AD B2C instance and either use the User Flows or built your Custom Policies with the Identity Experience Framework. Then you git clone and build the webapp [msal-b2c-web-sample](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples/msal-b2c-web-sample) according to the official Microsoft instructions for the sample

You need to update the application.properties file with your B2C tenant details
```XML
b2c.tenant=yourtenant.onmicrosoft.com
b2c.host=yourtenant.b2clogin.com
b2c.redirectUri=http://localhost:8080/msal4jsample/secure/aad
b2c.api=http://localhost:8081/api/items
b2c.api-scope=https://yourtenant.onmicrosoft.com/your-api-name/demo.read
b2c.authority.base=https://yourtenant.b2clogin.com/tfp/yourtenant.onmicrosoft.com/
b2c.sign-up-sign-in-authority=https://yourtenant.b2clogin.com/tfp/yourtenant.onmicrosoft.com/b2c_1a_signup_signin/
b2c.edit-profile-authority=https://yourtenant.b2clogin.com/tfp/yourtenant.onmicrosoft.com/b2c_1a_edit_profile/
b2c.reset-password-authority=https://yourtenant.b2clogin.com/tfp/yourtenant.onmicrosoft.com/b2c_1a_resetpassword/
b2c.clientId=...webapp client_id...
b2c.secret=...webapp secret/key...
```

Then you git clone this sample and update its application.properties file. 

```XML
server.port=8081
logging.level.org.springframework.oauth2=DEBUG
logging.level.org.springframework.jwt=DEBUG
logging.level.org.springframework.cache=DEBUG
security.oauth2.client.authority=https://yourtenant.b2clogin.com/tfp/yourtenant.onmicrosoft.com/
security.oauth2.client.client-id=...api client_id...
security.oauth2.client.client-secret=...api secret/key...
security.oauth2.resource.id=...api for client_id...
security.oauth2.scope.access-as-user=https://yourtenant.onmicrosoft.com/your-api-name/demo.read
security.oauth2.resource.jwt.key-uri=https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/discovery/v2.0/keys?p=b2c_1a_signup_signin
security.oauth2.issuer=https://yourtenant.b2clogin.com/yourtenant-guid/v2.0/
```

If you build and run the two samples simultaneously, you will have an all Java sample of a WebApp authenticating using Azure AD B2C calling a Java WebAPI with the B2C access token

## Things to pay attention to

**Scope** value to be set in application.properties for the WebApp is the long ***Value*** in portal.azure.com and have a format like this: https://yourtenant.onmicrosoft.com/your-api-name/demo.read. It is easy to get wrong, so copy-n-paste the value from the portal. In application.properties for the **WebApi** you also need to add the long value.

**Secrets** from B2C may contain characters like " and $. In application.properties, you don't need to escape them as everything after the = sign is treated like the settings value by Java. But if you set the secret value in environment variables in bash, powershell or in the env section of the vscode launch.json file, remember to escape the secret characters " and $

**RedirectUri** The error handeling in the WebApp around the login part isn't perfect, so if you have a B2C config error, like the RedirectUri misspelled, you will just land in the error page with no explanation. If you encounter these types of errors, run the page in Chrome with DevTools open and look at the real response from B2C to see the real error.

**Packaging** The WebApp builds to a WAR file that you deploy to Tomcat. If you prefer to run it as a .jar file or debug it in vscode, just change war to jar in the pom.xml file
```XML
<packaging>jar</packaging>
```