### Configuring and Using an OpenId Connect Provider to use Application Passwords



### Usage of application passwords
  - A provider REST endpoint processes requests to create, list and delete (revoke) application passwords.
  - A user first authenticates to the provider and obtains a short-lived access token.
  - The user submits this token to the provider's application-passwords REST endpoint and obtains a long-lived application password.
  - The user supplies this application-password to a non-browser application.
  - The non-browser application can submit this application password to the provider's token endpoint using the OAuth ROPC flow, and obtain another short-lived access token.  
  - The access token can be submitted as a credential to access applications protected by an OpenIdConnect client.
  - The access token can be submitted to the provider's introspect endpoint to obtain information about the user. 
  - Short-lived access tokens can be requested repetitively as needed by the non-browser application until the application password expires or is revoked.



### Configuration:
  - Enable the use of application passwords by configuring or registering an OAuth client to specify appPasswordAllowed="true". Set the Oauth Provider parameter passwordGrantRequiresAppPassword to true.

### Optional Configuration:
  - Configure users or groups in the tokenManager role to administer the tokens and passwords of other users. 
  - Limit the maximum number of passwords or tokens a user can request by specifying appTokenOrPasswordLimit in the OAuthProvider configuration.  The default value is 100.
  - Customize lifetimes with the OauthProvider appPasswordLifetime parameters. The default value is 90 days.
 

 ## Example configuration 
  ```
     <openidConnectProvider id="OP" oauthProviderRef="Oauth"  signatureAlgorithm="RS256" jwkEnabled="true">
    </openidConnectProvider>
    
    <oauthProvider id="Oauth" passwordGrantRequiresAppPassword="true"   appPasswordLifetime="30d"   appTokenOrPasswordLimit="3" >
        <localStore>   
            <client name="RP" secret="thesecret" displayname="rp"  resourceIds="abc123"
                 scope="openid profile scope1 email phone address" enabled="true"
                 appPasswordAllowed="true"
                 preAuthorizedScope="profile">
                <redirect>https://localhost:19045/oidcclient/redirect/RP</redirect>              
             </client>          
        </localStore>
    </oauthProvider>     

   <oauth-roles>
        <authenticated>
            <special-subject type="ALL_AUTHENTICATED_USERS" />  
        </authenticated> 
        <tokenManager>
            <user name="adminuser" />
            <group name="admins" />
        </tokenManager>
   </oauth-roles>

  ```


### Requesting, listing, and deleting (revoking) application-passwords

The REST endpoint for application-passwords allows users to create, list, and delete (revoke) their application passwords. A user in the "tokenManager" OAuth role can act as an administrator to list and delete (revoke) the passwords of other users. 

#### Obtaining an initial access token
An OAuth access token is required to use the REST endpoints. It can be obtained from an OpenID Connect Client, or requested directly from the OpenID Connect provider. 
   - If your application is protected by an OpenID Connect client, your application can use the [Propagation Helper API](https://www.ibm.com/support/knowledgecenter/en/SSAW57_liberty/com.ibm.websphere.javadoc.liberty.doc/com.ibm.websphere.appserver.api.oidc_1.0-javadoc/com/ibm/websphere/security/openidconnect/PropagationHelper.html) to obtain the access token that it obtained from the provider. 
   - You can request an access token from the Provider directly using an implicit flow 
       - The authorization header contains the base64 encoded user id and password. 
       - The scope parameter must be a preAuthorizedScope in the client configuration  

implicit flow example:       
```     
GET /oidc/endpoint/OP/authorize?response_type=token&scope=profile&client_id=RP&redirect_uri=https://localhost:19045/oidcclient/redirect/RP HTTP/1.1
Host: localhost:29443
Authorization: Basic Ymlnd2lnOnRlc3R1c2VycHdk
User-Agent: curl/7.54.0
Accept: application/json 

HTTP/1.1 302 Found
X-Powered-By: Servlet/3.1
Pragma: no-cache
Location: https://localhost:19045/oidcclient/redirect/RP#scope=profile&access_token=2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri&token_type=Bearer&expires_in=7199     
```


#### Using the app-passwords endpoint:
 
  - Request app-password:
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user.
      - app_name - a convenience name to help the user remember the purpose of the password.
      - used_by - optional. The client id that will be exchanging the app-password.
         - If specified, only that client can exchange the app-password for a short-lived access token.        

      ```   
      POST /oidc/endpoint/OP/app-passwords HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0
      Content-Type: application/x-www-form-urlencoded
      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri

      app_name=mytestapp2&used_by=RP

      HTTP/1.1 200 OK
      {"app_password":"jAM4RMHBGFprIoBCwywSp9yhPVh90pFLlQC09ipu","app_id":"lvhzjTlZD2GKlEVFJoAiZB8B1kqh3ASl6ZW11PSx","created_at":"1556727785401","expires_at":"1564503785401"}
      ```


  - List app-passwords:
      - Note that list will only show passwords associated with the client id specified.  Passwords for this user, but created for different client ids, will not be shown. 
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user.
      - app_id - optional request parameter.  The app_id of an application password.  If omitted, all application passwords will be listed.
      - user_id -optional request parameter.  User whose application passwords will be listed.  Only valid if the authenticated user is in the tokenManager role. 

      ```
      GET /oidc/endpoint/OP/app-passwords?user_id=testuser HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0
      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri
      Content-Length: 14
      Content-Type: application/x-www-form-urlencoded

      
      HTTP/1.1 200 OK
      {"app-passwords":[{"user":"testuser","name":"mytestapp","app_id":"4nTuMZ0aocXZcdXMS2KbtZ1QRmZzR7If3oBqDJWO","created_at":1556727593634,"expires_at":1564503593634},{"user":"testuser","name":"mytestapp2","app_id":"lvhzjTlZD2GKlEVFJoAiZB8B1kqh3ASl6ZW11PSx","created_at":1556727785401,"expires_at":1564503785401}]}
      ```

  - Delete (revoke) app-passwords:     
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user.
      - app_id - optional URL fragment appended after app-passwords.   The app_id of an application password.  If omitted, all passwords for the  user and client id will be revoked.
      - user_id -optional request parameter.  User whose passwords or tokens will be deleted.  Only valid if the authenticated user is in the tokenManager role.

      ```
      DELETE /oidc/endpoint/OP/app-passwords/lvhzjTlZD2GKlEVFJoAiZB8B1kqh3ASl6ZW11PSx?user_id=testuser HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0

      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri

      HTTP/1.1 200 OK
      ```


### Using the app-password:
   - App-passwords are submitted to the token endpoint using the ROPC flow to obtain a short-lived access token. 
   - The authorization header contains the base64 encoded client id and secret.  
   - passwordGrantRequiresAppPassword must be set to true in the provider configuration.

   ```
      POST /oidc/endpoint/OP/token HTTP/1.1
      Host: localhost:29443
      User-Agent: curl/7.54.0
      Authorization: Basic UlA6dGhlc2VjcmV0
      Content-Type: application/x-www-form-urlencoded
      Accept: application/json

      grant_type=password&scope=profile&username=testuser&password=s1Ft8uUV0V2s2XtI7dgMmtBUobjDAyaxLOU5saHm

      HTTP/1.1 200 OK

      {"access_token":"UtwHp5v8fBHhCrNwN6uO1aZMIpXkPdHIsxiIOtHw","token_type":"Bearer","expires_in":7199,"scope":"profile","refresh_token":"LI6cdchT9uBlAEschVC0gZCgNKysMrO35QVilDj8XOKQMUhCzY"}
   ```

### Using the short-lived access token:
   - The short-lived access token is a normal OAuth access token and can be used in all the OAuth and OpenId Connect flows that accept such tokens.
   - Short-lived access tokens can be submitted as an authentication credential to OpenIdConnect clients that are configured to accept them.
   - Short-lived access tokens can be  submitted to the provider introspect endpoint to obtain information about the user who initially created the application password that was used to obtain the short-lived token.   

   ```
      POST /oidc/endpoint/OP/introspect HTTP/1.1
      Host: localhost:29443
      User-Agent: curl/7.54.0
      Authorization: Basic UlA6dGhlc2VjcmV0
      Content-Type: application/x-www-form-urlencoded
      Accept: application/json

      token=UtwHp5v8fBHhCrNwN6uO1aZMIpXkPdHIsxiIOtHw

      HTTP/1.1 200 OK

      {"sub":"testuser","grant_type":"resource_owner","realmName":"OpBasicRealm","scope":"profile","uniqueSecurityName":"testuser","groupIds":["testers"],"iss":"https:\/\/localhost:29443\/oidc\/endpoint\/OP","active":true,"exp":1556738944,"token_type":"Bearer","iat":1556731744,"client_id":"RP"}
   ```