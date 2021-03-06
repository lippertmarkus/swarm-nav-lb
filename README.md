Load-Balancing Microsoft Dynamics NAV in Docker Swarm Mode
===

Example Docker Stack for dynamically load balancing Microsoft Dynamics NAV inside a Docker Swarm. For more details see [https://www.axians-infoma.de/navblog/load-balancing-navbc-in-docker-swarm/](https://www.axians-infoma.de/navblog/load-balancing-navbc-in-docker-swarm/)

# Components

The Docker stack consists of three services: 
- [Custom load balancer](https://github.com/lippertmarkus/windows-swarm-lb) based on [OpenResty](https://openresty.org/en/) for dynamic load balancing with automatic reconfiguration on failures or scaling. OpenResty is a platform for extending the features of the popular webserver [nginx](https://nginx.org) via various libraries and LUA scripts. This way the internal Docker DNS server can be used for retrieving available instances without any external service discovery.
- Scalable NAV service using official [Microsoft Dynamics NAV](https://store.docker.com/community/images/microsoft/dynamics-nav) image with overwritten healthcheck interval for faster failovers
- SQL server using official [Microsoft SQL-Server image](https://store.docker.com/community/images/microsoft/mssql-server-windows-express)

The stack also uses Docker secrets for the NAV password key file.

# Getting started

For a simple single-node swarm:

0. Set up a Docker Swarm Mode cluster
1. Build NAV image inside `nav` folder: `docker build -t swarm/nav .` (uses newest NAV image per default)
2. Build load-balancer image inside `nav-lb` folder: `docker build -t swarm/nav-lb .`
3. Adapt `docker-stack.yml`:
   - Set `PublicDnsName` of service `nav` to match the DNS name of your host system
   - Set `volumes` and environment variable `attach_dbs` of service `navdb` according to the location and naming of your databases files
4. Deploy the stack: `docker stack deploy --compose-file docker-stack.yml mystack`

Web- and windows client as well as the webservices are available and load balanced. The ports are published on the host system with default port numbers. ClickOnce deployment is available at [`http://myhost:8080/nav/`](http://myhostsys:8080/nav/).

Default windows user is `myuser` with password `Test123!`. After making sure the NAV user was sucessfully created you can login with the windows account `NAV\myuser` and the password. Consider using a gMSA for simplified access, uncomment the following lines:
```yml
#credential_spec:
#    file: nav.json
```

Windows authentication is used as default as NavUserPassword authentication uses an AntiForgeryToken cookie in the Webclient. When this cookie is sent to another instances during a failover it causes a `502 Bad Gateway` error and you must delete the AntiForgeryToken cookie by hand.

# Details

Details on the load balancer can be found in the corresponding [repository](https://github.com/lippertmarkus/windows-swarm-lb). You can customize the configuration via the [`nginx.conf`](./nav-lb/nginx.conf) file to e.g. add SSL or change other settings. 

For encrypting a password with a password key file you can use the following snippet:
```powershell
$SecurePasswordString = Read-Host "Enter a password" -AsSecureString
$Key = Get-Content -Path 'my.key'
$Encrypted = ConvertFrom-SecureString -SecureString $SecurePasswordString -Key $Key
Write-Host $Encrypted
```
The encrypted password can be used for `securepassword` and `databaseSecurepassword` of the `nav` service.
