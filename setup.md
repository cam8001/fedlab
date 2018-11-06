# Setup
## Create Azure Free Account
Follow manufacturer's instructions :)

## Populate Default Domain
__Record__ your domain name as $AZD and the sub-domain component as $AZDSUB
|Key|Sample
|---|---
|$AZD|`craigaroachicloud.onmicrosoft.com`
|$AZDSUB|`craigaroachicloud`

### Add User
* Name: Alice Armstrong
* User name: alice@$AZD
* Profile: arbitrary
* Directory role: User
***
__Record__ user's generated password as $ALICEPWD

__Note__ Password will changed on first login. Now `AUTOcoot+3`

### Add Group
* Group Type: Security
* Group Name: OperationsManagers
* Group Description: arbitrary
* Membership type: Assigned
***

### Add User to Group
__Select__ the Group *OperationsManagers*

__Add__ members
* Alice Armstrong
***

## Create Cognito User Pool
__Navigate__ to Cognito in Console, then Create Pool

__Enter__ your chosen pool name and record as $CUP
`$CUP=BlueCup`

* Sign in: Username
* Required attributes: email
* Custom attributes: skip
* Password Strength: your choice
* Allow user sign-up: true
* MFA: off
* Verification: email
* Message customizations: skip
* Remember your user's devices
* App Clients: skip for now
* Custom workflows: skip
***

__Save__ your user pool.

__Record__ your Pool Id and ARN as $CUPID and $CUPARN
|Key| Sample
|---|---
|$CUPID |ap-southeast-2_0ShBpWf3u
|$CUPARN|arn:aws:cognito-idp:ap-southeast-2:167635472246:userpool/ap-southeast-2_0ShBpWf3u

## Configure App Integration for Cognito User Pool
__Select__ your Cognito User Pool called $CUP

__Navigate__ App Integration | Domain name

__Determine__ a domain name prefix for your user pool. It will need to be unique within your selected AWS region. A concatenation of $AZDSUB and $CUP is a good choice. You can click Check Availability button.

__Enter__ your chose domain prefix and record as $CUPDP
|Sample $CUPDP
|---
|`craigaroachicloud-bluecup`

__Save Changes__

__Record__ your fully qualified Cognito user pool domain name $CUPFQDN
|Sample $CUPFQDN
|---
|https://craigaroachicloud-bluecup.auth.ap-southeast-2.amazoncognito.com

__Construct__ your SAML endpoint using this domain name. After authenticating a user, AAD will respond with a HTTP Redirect to this endpoint using the SAML POST binding method. This method encodes the SAML Assertion as HTML FORM data, which the user's browser POSTs to the endpoint. To construct the endpoint URL, append `/saml2/idpresponse`. Record the endpoint URL as $CUPSAML
> $CUPSAML = $CUPFQDN + `/saml2/idpresponse`

|Sample $CUPSAML
|---
|https://craigaroachicloud-bluecup.auth.ap-southeast-2.amazoncognito.com/saml2/idpresponse

__Construct__ your SAML Service Provider (SP) Entity ID - also known as the Audience URI - by prefixing your Pool Id with `urn:amazon:cognito:sp:`. Record the Entity ID as $CUPSPENTITY
> $CUPSPENTITY = `urn:amazon:cognito:sp:` + $CUPID

|Sample $CUPSPENTITY
|---
|`urn:amazon:cognito:sp:ap-southeast-2_0ShBpWf3u`


__Navigate__ to Cognito | $CUP | General Settings | App Clients

|Field|Value
|---|---
|App client name| *TeamCalendar*
|Generate client secret| true
__Create app client__

__Record__ App client id as $CUPCID

|Key| Sample
|---|---
|$CUPCID |2n6pup059ng2scg8n1crb1v5j6
|$CUPCSECRET|kb5a9vmn2ua7qu70jghgv5m3bsopqheaeaumsadgmpbthc8sf6s


***
https://your_domain/login?response_type=code&client_id=your_app_client_id&redirect_uri=your_callback_url


https://craigaroachicloud-bluecup.auth.ap-southeast-2.amazoncognito.com/login?response_type=code&client_id=2n6pup059ng2scg8n1crb1v5j6&redirect_uri=https://localhost

***


## Add Application to AAD
__Navigate__ to AAD | Default Directory | Manage | Enterprise Applications

__Select__ New Application, then __Select__ Non-gallery application
* Name: TeamCalendar
***
__Add__

__Select__ Manage | Single sign-on, then __Select__ SAML

__Navigate__ to Basic SAML Configuration, then __Select__ Edit

|Attribute|Value|Sample
|---|---|---
|Identifier (Entity ID) | $CUPSPENTITY | urn:amazon:cognito:sp:ap-southeast-2_0ShBpWf3u
|Reply URL (ACS URL) | $CUPSAML | https://craigaroachicloud-bluecup.auth.ap-southeast-2.amazoncognito.com/saml2/idpresponse

__Save__

__Navigate__ to User Attributes & Claims, then __Select__ Edit

__Click__ on Claim `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

__Change__ Source Attribute from `user.mail` to `user.userprincipalname`


__Navigate__ to SAML Signing Certificate, then __Record__ App Federation Metadata URL as $AZDFM
|Sample $AZDFM
|---
|`https://login.microsoftonline.com/23033791-d560-4912-8b87-c6e361fa3e17/federationmetadata/2007-06/federationmetadata.xml?appid=9be99ae6-3547-4ba7-b65d-7e0e9c1b4365`
Optionally, __Download__ the Federation Metadata XML

__View__ Metadata document URL in browser, or downloaded XML


__Navigate__ to *TeamCalendar* | Manage | Users and groups

__Select__ Add User

__Find__ *Alice Armstrong*, then __Select__

__Select__ then __Assign__

__Confirm__ *Alice Armstrong* has been assigned to *TeamCalendar*

__Logout__ of AAD
***

## Add AAD a Federated Identity Provider for Cognito User Pool

__Navigate__ to Cognito | $CUP | Federation | Identity Providers

__Select__ SAML

__Navigate__ to Metadata Document

*Either* __Enter__ $AZDFM *OR* __Upload__ Federation Metadata XML

__Enter__ Provider name `AAD`

__Create Provider___


## Configure Cognito App Client for ALB

__Navigate__ to Cognito | $CUP | App Integration | App Client Settings

__Navigate__ to $CUPCID

|Field|Value|Sample
|---|---
|Enabled Identity Providers| AAD | -
|Callback URL| $ALBURL + `/oauth2/idpresponse` | https://www2.craigroachnz.net/oauth2/idpresponse
|OAuth2 - Allowed Flows| Authorization Code Grant | -
|OAuth2 - Allowed Scopes| email, openid | -
