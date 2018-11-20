# Single Page JavaScript FHIR Client in Azure Web App

## TL;DR
Deploy template and single page JavaScript in to Azure Web App with the button below:

<a href="https://transmogrify.azurewebsites.net/azuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Locate the `wwwroot/config.json` file in the web app to adapt to your FHIR server settings.

## Details
Fast Healthcare Interoperable Resources ([FHIR](https://hl7.org/fhir)) is an evolving standard for healthcare interoperability. The SMART on FHIR framework describes specific [launch sequences](http://hl7.org/fhir/smart-app-launch/) for clients. The launch sequence includes an authorization flow that leads to the client obtaining an access token which must be presented when consuming the FHIR API.

When serving a FHIR client from an [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/), an alternative option is to leverage the built in [App Service Authentication](https://docs.microsoft.com/en-us/azure/app-service/app-service-authentication-overview) framework. One of the immediate advantages is that it allows the application to obtain a token as a secure client (with a secret) as opposed to using code flow without a secret as described in the SMART on FHIR specification. Moreover, since Azure App Service is easily integrated with Azure Active Directory, this approach helps to bridge known gaps between the SMART on FHIR standard and Azure Active Directory conventions. 

If Azure App Service Authentication is enabled, users will be asked to provide credentials and once signed in a token can be automatically obtained for a requested resource (a FHIR server). Subsequently the token information will be available at the `/.auth/me` endpoint. In the example client (see the [www](www/) folder) in this repository, the token is simply read from this endpoint and the JavaScript FHIR client is instanciate with this token.

To configure an App Service to obtain a token for a backend API (such as a FHIR server), an authority, a client id, a client secret, and a resource (audience) must be configured. The resource or audience is provided in the `additionalLoginParams` parameter of the App Service's `authsettings`. Since this cannot be set in the portal, it has to be done with a template or command line tools. This repository contains a [template](azuredeploy.json), which will set the appropriate settings on the App Service. The relevant resource from the template looks like this:

```json
{
    "apiVersion": "2015-08-01",
    "name": "authsettings",
    "type": "config",
    "dependsOn": [
        "[concat('Microsoft.Web/Sites/', variables('siteName'))]"
    ],
    "properties": {
        "enabled": true,
        "unauthenticatedClientAction": "redirectToLoginPage",
        "tokenStoreEnabled": true,
        "defaultProvider": "AzureActiveDirectory",
        "clientId": "[parameters('aadClientId')]",
        "clientSecret": "[parameters('aadClientSecret')]",
        "issuer": "[parameters('aadAuthority')]",
        "additionalLoginParams": [
            "[concat('resource=',parameters('aadAudience'))]"
        ],
        "isAadAutoProvisioned": false
    }
}        
```

To deploy this Azure App Service and an example single page FHIR client, use the button below:

<a href="https://transmogrify.azurewebsites.net/azuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Make sure that the Azure AD app registration has the appropriate reply url set for Azure App Service Authentication. It would be something like:

```
https://YOUR-SITE-NAME.azurewebsites.net/.auth/login/aad/callback
```

The application will deploy with it's FHIR `serviceUrl` parameter pointing to the public HAPI FHIR server and the bearer token will not be forwarded to the server (since HAPI does not require authentication). To change the settings to be appropriate for your FHIR server, locate the `config.json` file in the `wwwroot` folder of the Web App (e.g., using Kudu console). It will look something like this:

```json
{
    "serviceUrl": "https://hapi.fhir.org/baseDstu3",
    "defaultPatientId": "example",
    "disableAuth": true
}
```
Change the `serviceUrl` to point to your FHIR server, change the `defaultPatientId` to point to an existing patient, and switch `disableAuth` to `false` to enable authenticated communication with your FHIR server. The patient can can also be specified using the `patient=` URL parameter, e.g., `https://YOUR-SITE-NAME.azurewebsite.net/?patient=my-patient-id`

