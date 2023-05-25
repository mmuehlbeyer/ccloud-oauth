# ccloud-oauth

## Prerequisits

* Access to Azure AD
* Access to Confluent Cloud 
* curl 
* Confluent CLI 
* Azure CLI for 

## Azure AD

### Create Azure AD tenant for testing (optional) 

Create a seperate tenant for testing purposes or use and existing what fits your need best.
   ![tenant01.png](assets/tenant01.png)
   ![tenant02.png](assets/tenant02.png)

### Create App Registration

Navigate to the newly created tenant, should look similar to
       ![tenant03.png](assets/tenant03.png)


Choose "App registrations"  
       ![appreg01.png](assets/appreg01.png)


Now we can create our application which will be a client for Kafka
Choose a proper name leave everything else to its defaults and click register.
       ![appreg02.png](assets/appreg02.png)


After the registrtration, the application will be assigned an ID
       ![appreg03.png](assets/appreg03.png)

Switch to certificates and & secrets on the navigation pane and create a new client secret.  
       ![secret01.png](assets/secret01.png)  

Do not forget to copy the secrets as they will be hidden soon.  
       ![secret02.png](assets/secret02.png)


Now we have to switch to "Expose an API" to set appliaction URL.  
       ![api01.png](assets/api01.png)
just keep the default and click "save"
       ![api02.png](assets/api02.png)


To ensure that Azure AD returns version 2 token we need to do a small change in the Manifest.

Click on "Manifest" in the left-hand bar and look for ```accessTokenAcceptedVersion``` in the Manifest.
By default the value is ```null``` which should be changed to ```2```
       ![manifest01.png](assets/manifest01.png)

Do not forgot to ```save``` your changes ;-)


Finally we could switch back to Overview and check the Endpoints.
       ![endpoints01.png](assets/endpoints01.png)


E.g the OAuthv2.0 token endpoint is 

https://login.microsoftonline.com/cf91a0a7-813e-42d3-9f6b-a0cda8062713/oauth2/v2.0/token


### Create a token via curl

We can get a JWT token with the following command.  

Adapt 
```client_id, api, client_secret , token URL ```
accordingly

```client_id``` can be obtained from the overview pane of our application  
```api``` can be obtained from "Expose an API"    
```client_secret``` hopefully saved during generation above ;-)  
```token URL```can be taken from Endpoints (see above for details)

generic example

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" 
-d 'client_id=$YOUR_CLIENT_ID\
&scope=$YOUR_API/.default\
&client_secret=$YOUR_CLIENT_SECRET\
&grant_type=client_credentials'\
'$YOUR_OAUTH2_ENDPOINT'
```


example with values taken from the demo above

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=5172d8ab-3278-418a-ac65-a70edf7a7211&scope=api://5172d8ab-3278-418a-ac65-a70edf7a7211/.default&client_secret=Lez8Q~Ed1wNZm6i6K9vuRBdrYykocIu3DvbqtbII&grant_type=client_credentials' 'https://login.microsoftonline.com/cf91a0a7-813e-42d3-9f6b-a0cda8062713/oauth2/v2.0/token'
```


the command should result in a JSON Web Token like this:

```bash
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":3599,"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ii1LSTNROW5OUjdiUm9meG1lWm9YcWJIWkdldyJ9.eyJhdWQiOiI1MTcyZDhhYi0zMjc4LTQxOGEtYWM2NS1hNzBlZGY3YTcyMTEiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vY2Y5MWEwYTctODEzZS00MmQzLTlmNmItYTBjZGE4MDYyNzEzL3YyLjAiLCJpYXQiOjE2ODQ5OTU3MzYsIm5iZiI6MTY4NDk5NTczNiwiZXhwIjoxNjg0OTk5NjM2LCJhaW8iOiJFMlpnWU1nM05hek5TOWoydUdkQnJYcUtNTC8xamViVHF3NTlzcnJ1c1d1R1N3Rlh2Qm9BIiwiYXpwIjoiNTE3MmQ4YWItMzI3OC00MThhLWFjNjUtYTcwZWRmN2E3MjExIiwiYXpwYWNyIjoiMSIsIm9pZCI6IjA4M2NmMDc5LTBiODUtNDA4MC04OGI2LTU0ODgzZmZiYjMzOSIsInJoIjoiMC5BVXNBcDZDUnp6NkIwMEtmYTZETnFBWW5FNnZZY2xGNE1vcEJyR1duRHQ5NmNoRkxBQUEuIiwic3ViIjoiMDgzY2YwNzktMGI4NS00MDgwLTg4YjYtNTQ4ODNmZmJiMzM5IiwidGlkIjoiY2Y5MWEwYTctODEzZS00MmQzLTlmNmItYTBjZGE4MDYyNzEzIiwidXRpIjoiWVVhZG9MX09Sa3FRNFdlSVpsTmxBQSIsInZlciI6IjIuMCJ9.MHkVZqNkA_-d0vEBjQx-mS99vA679eyD2YZU2BnKJ64OUwakj6KhD93fqNLBvOKdUnrGkLXDcEELDMrGQn3UzPLCffOhgterYtLBoXaN3hNLE_6EIosPkZT28aTpnDwJV2B-H3UhepYa3-6sAmBfb6nvQj5C3_-tqmHkizLnbjLGzl9Q3Ubct9SyepmXlRLfQVPlqyIBaf9tPHc89_pU_BCjoVQeafbSVPK1mt1HmVpQ29TrenHb6zAoYRt1AtBs-cSgbxDBUl45I26mDuteI4rzufVBe6H7x-Zk8oXp8fFombR1U6B8DGoBSCWV7sv2giVcAxLM4zgH4lIt36a1Tg"}
```


### Configure Confluent Cloud for Azure AD IDP

Let's configure the Confluent Cloud part


Login to https://confluent.cloud and go to **Accounts & access** in the burger menu in the top right
and select **Identity providers** tab

Select **Add provider** 
       ![provider01.png](assets/provider01.png)

Choose Azure AD then press Next
       ![provider02.png](assets/provider02.png)

In the next screen, enter a name for this provider, a description of the provider, and the Tenant ID (get if from the overview pane) from Azure. 
Pressing the "Import from Tenant ID" button will populate the Issuer and JWKS URI fields.
       ![provider03.png](assets/provider03.png)


As the provider is created we have to create the Identity Pools.
Click on the new provider.
       ![provider04.png](assets/provider04.png)


The  pool needs a name, an Identity Claim and a Filter.
The Identity claim is the field of the token that we consider to be the name of the application logging in,
for the purposes of audit. 
It defaults to "claims.sub" (subject) but in this example we have set it to "claims.azp" (Authorised Party).

The filter specifies which identities will be included in this pool by inspecting the contents of the token.
The next step is to assign permission to the identity pool using RBAC. "Import permissions" will apply the RBAC permissions of an existing service account. "Add new permissions" will allow new role bindings to be applied to this Identity Pool.  
       ![ip01.png](assets/ip01.png)

