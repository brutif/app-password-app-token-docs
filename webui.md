
The following web UI pages are available:

```https://host:port/[provider id]/clientManagement``` allows administrators in the clientAdmin role to manage clients.


```https://host:port/[provider id]/personalTokenManagement``` allows users to manage their application passwords and application tokens.

```https://host:port/[provider id]/usersTokenManagement``` allows administrators in the tokenManager role to list and revoke application passwords and application tokens of other users. 

The personalTokenManagement and usersTokenManagement pages require the OAuth configuration attributes ```internalClientId``` and ```internalClientSecret``` to be set to the id and secret of a valid OAuth client. These will be used to generate an access token for for the web UI  to call the REST APIs. 

 ## Example configuration 
  ```
    <openidConnectProvider id="OP" oauthProviderRef="Oauth"  signatureAlgorithm="RS256" jwkEnabled="true">
    </openidConnectProvider>
    
    <oauthProvider id="Oauth" 
       appTokenLifetime="30d" 
       appPasswordLifetime="30d"
       appTokenOrPasswordLimit="3" 
       passwordGrantRequiresAppPassword="true"
       internalClientId="c845821d837f42e3ae4c7b1dbcfbc3b7"
       internalClientSecret="kTdnSwBjna8uf3nOucNifvb6a1MSXtdbVPEnBNJPx2bZeAEmYk7sbaTKmDqy">
       <databaseStore dataSourceRef="OAuthDataSource" />
    </oauthProvider>

  ```
