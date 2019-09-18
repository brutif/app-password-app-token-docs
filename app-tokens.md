### Configuring and Using an OpenId Connect Provider to use Application Tokens



### Usage of application tokens
  - A provider REST endpoint processes requests to create, list and delete (revoke) application tokens.
  - A user first authenticates to the provider and obtains a short-lived access token.
  - The user submits this token to the provider's application-tokens REST endpoint and obtains a long-lived application token.
  - The user supplies this application-token to a non-browser application.  
  - The access token can be submitted as a credential to access applications protected by an OpenIdConnect client.
  - The access token can be submitted to the provider's introspect endpoint to obtain information about the user. 
  - The long-lived access token is valid until it expires or is revoked.


### Configuration:
  - Enable the use of application tokens by configuring or registering an OAuth client to specify appTokenAllowed="true". 

### Optional Configuration:
  - Configure users or groups in the tokenManager role to administer the tokens and passwords of other users. 
  - Limit the maximum number of passwords or tokens a user can request by specifying appTokenOrPasswordLimit in the OAuthProvider configuration.  The default value is 100.
  - Customize lifetimes with the OauthProvider appTokenLifetime parameters. The default value is 90 days.
 

 ## Example configuration 
  ```
     <openidConnectProvider id="OP" oauthProviderRef="Oauth"  signatureAlgorithm="RS256" jwkEnabled="true">
    </openidConnectProvider>
    
    <oauthProvider id="Oauth" appTokenLifetime="30d" appTokenOrPasswordLimit="3" >
        <localStore>   
            <client name="RP" secret="thesecret" displayname="rp"  resourceIds="abc123"
                 scope="openid profile scope1 email phone address" enabled="true" 
                 appTokenAllowed="true"
                 preAuthorizedScope="profile" >        
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


### Requesting, listing, and deleting (revoking) application tokens

The REST endpoint for application-tokens allows users to create, list, and delete (revoke) their application tokens. A user in the "tokenManager" OAuth role can act as an administrator to list and delete (revoke) the tokens of other users. 

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


#### Using the app-tokens endpoint:
 
  - Request app-tokens:
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user. 
      - app_name - a convenience name to help the user remember the purpose of the token.     
      - used_by - optional. The client id that will be using the app-token.           
         - If specified, only that client can introspect the app-token. 
         - If not specified, the first client to use the token is the only client that can introspect the app-token.    

      ```   
      POST /oidc/endpoint/OP/app-tokens HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0
      Content-Type: application/x-www-form-urlencoded
      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri

      app_name=mytestapp1&used_by=client_04

      HTTP/1.1 200 OK
      {"app_token":"xuZtiWoJIiXfGxY9lJgWzryVuoSQgFWUEs8f4IGh","app_id":"iHoDV5mgbCfJdNxL3TTucA2ewM0VmjYaLxpvFAB2","created_at":"1556733579572","expires_at":"1564509579572"}
      ```


  - List app-tokens:
      - Note that list will only show tokens associated with the client id specified.  Tokens for this user, but created for different client ids, will not be shown. 
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user.      
      - user_id -optional request parameter.  User whose application passwords will be listed.  Only valid if the authenticated user is in the tokenManager role. 

      ```
      GET /oidc/endpoint/OP/app-tokens?user_id=testuser HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0
      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri
      Content-Length: 14
      Content-Type: application/x-www-form-urlencoded

      
      HTTP/1.1 200 OK
      {"app-tokens":[{"user":"testuser","name":"mytestapp2","app_id":"iHoDV5mgbCfJdNxL3TTucA2ewM0VmjYaLxpvFAB2","created_at":1556733579572,"expires_at":1564509579572}]}
      ```

  - Delete (revoke) app-tokens:     
      - The authorization header contains the base64 encoded client id and secret.
      - The access_token header contains  the short-lived access token that identifies an authenticated user.      
      - user_id -optional request parameter.  User whose passwords or tokens will be deleted.  Only valid if the authenticated user is in the tokenManager role.

      ```
      DELETE /oidc/endpoint/OP/app-tokens?user_id=testuser HTTP/1.1
      Host: localhost:29443
      Authorization: Basic UlA6dGhlc2VjcmV0

      Accept: application/json
      access_token: 2cuQ5GqI9GorM7XXVl62t2UvkFjA9J5iX0YWXQri

      HTTP/1.1 200 OK
      ```



### Using the application token:
   - The application token is a normal OAuth access token with long lifetime, and can be used in all the OAuth and OpenId Connect flows that accept such tokens.
   - Access tokens can be submitted as an authentication credential to OpenIdConnect clients that are configured to accept them.
   - Aaccess tokens can be  submitted to the provider introspect endpoint to obtain information about the user who initially created the token.

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
