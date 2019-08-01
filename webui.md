
The following web UI pages are available:

```https://host:port/[provider id]/clientManagement``` allows administrators in the clientAdmin role to manage clients.


```https://host:port/[provider id]/personalTokenManagement``` allows users to manage their application passwords and application tokens.

```https://host:port/[provider id]/usersTokenManagement``` allows administrators in the tokenManager role to list and revoke application passwords and application tokens of other users. 

The personalTokenManagement and usersTokenManagement pages require the OAuth configuration attributes ```internalClientId``` and ```internalClientSecret``` to be set to the id and secret of a valid OAuth client. These will be used to generate an access token for for the web UI  to call the REST APIs. 

