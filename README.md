Load-Balancing Microsoft Dynamics NAV in Docker Swarm Mode
===

Example Docker Stack for dynamically load balancing Microsoft Dynamics NAV inside a Docker Swarm.

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

The ports are published on the host system with default port numbers. ClickOnce deployment is available at [`http://myhost:8080/nav/`](http://myhostsys:8080/nav/).

Default user is `myuser` with password `Test123!`

# Details

Details on the load balancer can be found in the corresponding [repository](https://github.com/lippertmarkus/windows-swarm-lb). You can customize the configuration via the [`nginx.conf`](./nav-lb/nginx.conf) file to e.g. add SSL or change other settings. 

For encrypting a password with a password key file you can use the following snippet:
```powershell
$SecurePasswordString = Read-Host "Enter a password" -AsSecureString
$Key = Get-Content -Path 'my.key'
$Encrypted = ConvertFrom-SecureString -SecureString $SecurePasswordString -Key $Key
Write-Host $Encrypted
```
The key file can be used for `securepassword` and `databaseSecurepassword` of the `nav` service.