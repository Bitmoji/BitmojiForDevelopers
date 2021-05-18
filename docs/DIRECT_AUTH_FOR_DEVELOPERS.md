![image](https://user-images.githubusercontent.com/31291049/109557561-44a8b900-7aa6-11eb-8390-9371c293448b.png)

# Bitmoji Direct Authorization
Bitmoji Direct Authorization allows users to connect their Bitmoji to 3rd party experiences outside of Bitmoji and Snapchat. Users control access to their Bitmoji through the Bitmoji iOS and Android apps.

## Overview
Bitmoji Direct Authorization provides a standard OAuth2 flow for requesting and accessing Bitmoji user data. Apps which are integrated with Bitmoji may request access using www.bitmoji.com/connect/ on both mobile devices and on desktop web. Users approve access to their Bitmoji within Bitmoji apps, and can manage connected apps within the Connected Apps settings screen.

See [RFC6749](https://tools.ietf.org/html/rfc6749) and [RFC7636](https://tools.ietf.org/html/rfc7636) for more details about OAuth2 and Proof Key for Code Exchange.

Once a user has connected an app, their data may be accessed by that app using the Direct API. The Direct API provides data such as the avatar, sticker packs, and search.

## Bitmoji App Support
|Platform|Version|Supported|Release Date|Notes|
|--------|-------------|-----|---|---|
| Android | >= 11.19 | :white_check_mark: | Mar 15 2021 | |
| iOS | >= 11.19 | :white_check_mark: | Mar 15 2021 |  |
| iOS | 11.17 | :warning: | Mar 1 2021 | Connected Apps screen not available |
| iOS | 11.15 | :warning: | Feb 17 2021 | Connected Apps screen not available |

## Configuring Your App
Developers must have a Snapchat account and register in the Snap Kit Developer portal.

In the portal developers may create an Org and an App. The App ID, Organization ID, and OAuth2 Client IDs in the portal will be used by Direct Authorization. The developer must inform the Bitmoji Partnerships Team of the App ID they wish to use with Direct Auth. Bitmoji engineers will then configure Direct Authorization and the Direct API to allow access for the developer.

## Usage
Their are two user flows for connecting a Bitmoji to an app: On mobile devices and on desktop web. Both flows are initiated using www.bitmoji.com/connect/.

#### Making an Authorization Request
Developers may make an OAuth2 Authorization Request to initiate the connection flow.

Method | URL
---|---
GET | https://www.bitmoji.com/connect/

Query Parameter | Description
---|---
client_id | OAuth2 Client ID identifier for your app available in the developer portal.
response_type | OAuth2 response type. Must be `code`.
redirect_uri | OAuth2 Redirect URI. Values must be registered with the Bitmoji team.
state | Optional OAuth2 state.
code_challenge | OAuth2 PKCE code challenge.
code_challenge_method | OAuth2 PKCE code challenge method. Must be `S256`.
package_name | Optional package name for redirection on Android using intents. Must be registered with the Bitmoji team.

Redirect URIs for use on iOS are strongly recommended to use Universal Links. On Android App Links are recommended, and custom URI schemes should be used in combination with the package name of the app receiving the redirect.

#### Handling the Authorization Response
When the user approves the access request, a `GET` request will be made to the redirect URI with the OAuth2 Authorization Response parameters.

Query Parameter | Description
---|---
state | OAuth2 state matching the value in the authorization request, if used.
code | OAuth2 authorization code.

#### Retrieving an Access Token
The Authorization Code value `code` may then be exchanged to retrieve an access token.

Method | URL
---|---
POST | https://bitmoji.api.snapchat.com/direct/token

Payload Value | Description
---|---
grant_type | OAuth2 grant type. Must be `authorization_code`.
code | OAuth2 authorization code from the authorization response.
client_id | Client identifier used in the authorization request.
redirect_uri | Redirect URI used in the authorization request.
code_verifier | OAuth2 PKCE code verifier.

The response will be JSON containing an Access Token Response.

Key | Description
---|---
access_token | May be used to access the Direct API.
refresh_token | May be used to obtain a new access token.
expires_in | Length of time until the access token expires in seconds.
token_type | `bearer`

#### Refreshing an Access Token
The access tokens may be refreshed. The response will be JSON containing an Access Token Response.

Method | URL
---|---
POST | https://bitmoji.api.snapchat.com/direct/token

Payload Value | Description
---|---
client_id | Client identifier used in the authorization request.
grant_type | OAuth2 grant type. Must be `refresh_token`.
refresh_token | A refresh token.

#### Revoking an Access Token
The access tokens or refresh tokens may be revoked. Revoking a token causes all tokens associated with the authorization session to be disabled.

Method | URL
---|---
POST | https://bitmoji.api.snapchat.com/direct/token/revoke

Payload Value | Description
---|---
token | Either an access token or refresh token.

### Mobile
When an iOS or Android mobile app initiates the connection flow the request will be handled by the Bitmoji app, if installed. Otherwise the request will be redirected to the App Store or Play Store to download Bitmoji.

Conceptual flow,
![image](https://user-images.githubusercontent.com/31291049/109555125-242b2f80-7aa3-11eb-8fce-fc9990fde908.png)


### Desktop Web
In a desktop web browser a scannable QR code will be shown. Users may scan the code with their phone cameras, then complete the connection flow in the Bitmoji app, if installed. If the app is not installed they will be redirected to the app store.

Conceptual flow,
![image](https://user-images.githubusercontent.com/31291049/109555088-1b3a5e00-7aa3-11eb-9e95-618d78b584d0.png)

### Limits
Access tokens expire in 1 hour.

Refresh tokens expire in 90 days.

### Debugging
OAuth errors are logged to the device console on iOS and Android.

#### iOS
Example 1. Sending wrong client Id,
```
[Bitmoji Direct Auth] ["message": Bad Request, "statusCode": 400, "error_description": invalid client_id "clientId", "error": invalid_client]
```
Example 2. Wrong code_challenge was send as part of request,
```
[Bitmoji Direct Auth] ["statusCode": 400, "message": Bad Request, "error_description": "code_challenge" length must be 43 characters long, "error": invalid_request]
```

#### Android
Example 1. Sending an invalid redirect URI,
```
2021-03-12 12:30:35.860 21758-21758/com.bitstrips.imoji I/ConsentPresenter: {"statusCode":400,"error":"invalid_request","message":"Bad Request","error_description":"invalid redirect_uri \"bad\""}
```
Example 2. Sending an invalid package name,
```
2021-03-12 12:34:24.997 21758-21758/com.bitstrips.imoji I/ConsentPresenter: {"statusCode":400,"error":"invalid_request","message":"Bad Request","error_description":"invalid package_name \"bad\""}
```

## Accessing User Data
Once an access token is obtained, requests for Bitmoji user data may be made.

#### The Avatar ID
Each Bitmoji user has an Avatar ID which uniquely identifies their avatar. The Avatar ID for a user changes whenever the user updates their avatar. For example, changing an outfit or updating an attribute will result in a new Avatar ID.

Method | URL
---|---
GET | https://bitmoji.api.snapchat.com/direct/avatar

Header | Description
---|---
authorization | Must be `bearer ${access_token}`

The response will be JSON.

Key | Description
---|---
id | The avatar ID for the user.

#### Rendering the Avatar
An image URL containing the avatar may be composed such as `https://sdk.bitmoji.com/me/sticker/${avatarId}`.

For example,
![avatar](https://sdk.bitmoji.com/me/sticker/AXdZZnpaOGt5A_sNnlgH1jR7lGZQJg).

#### Error Handling
Access tokens may expire or be revoked when a user disconnects their Bitmoji from the connected app. Developers are expected to handle these cases appropriately.

When using access tokens to access user data, these errors will return 401 Unauthorized status codes. Information about the specific error is included in the body of the JSON response.

Key | Description
---|---
statusCode | The HTTP status code.
message | The HTTP status code name.
error | An error code for the type of authorization error.
error_description | A message for the authorization error.

Error code | Description
---|---
expired_token | The token has expired. A refresh token may be used to attempt to retrieve a new token.
revoked_token | The token is revoked. The user has disconnected from the app. The user must authorize the app to connect again.
invalid_token | The token is malformed.

Example
```
% curl -s 'https://bitmoji.api.snapchat.com/direct/avatar' -H "authorization: bearer $token" | jq
{
  "statusCode": 401,
  "message": "Unauthorized",
  "error": "token_expired",
  "error_description": "The token has expired. A new token may be obtained using a refresh token."
}
```