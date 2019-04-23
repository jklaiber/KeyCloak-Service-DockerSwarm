# Keycloak - Auth - Controller
**Usefull Links:**  
* [Github keycloak proxy](https://github.com/ibuetler/docker-keycloak-traefik-workshop)
* [Medium keycloak proxy](https://medium.com/docker-hacks/how-to-apply-authentication-to-any-web-service-in-15-minutes-using-keycloak-and-keycloak-proxy-e4dd88bc1cd5)  

## Keycloak Auth Deployment
```
docker stack deploy --compose-file docker-compose.yml keycloak
```

## Add a service to keycloak

### First Step
In keycloak, create a new client and give it a name.  

Click on the "Credentials" tab in the keycloak configuration and copy the **Secret** value.    


### Second Step
You must set a keycloak aware reverse proxy in front of the deployment. The keycloak aware proxy will do the authentication and only web requests from authenticated users will be sent to your deployment.  

**Example:** have also a look at the `test-compose.yml` file
```yaml
version: "3.7"
services:
  auth-proxy:
    image: keycloak/keycloak-gatekeeper:5.0.0
    hostname: servicetest-proxy.example.com
    deploy:
      mode: replicated
      replicas: 1
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 1
        failure_action: rollback
      labels:
        traefik.port: 3000
        traefik.frontend.rule: Host:servicetest.example.com
        traefik.protocol: http
        traefik.frontend.passHostHeader: 'true'
        traefik.docker.network: transit_servicetest_traefik
    command: >
      --discovery-url=https://auth.example.com/auth/realms/master
      --skip-openid-provider-tls-verify=true
      --client-id=servicetest
      --listen=0.0.0.0:3000
      --client-secret=<Your-Client-Secret>
      --enable-refresh-tokens=true
      --redirection-url=https://servicetest.example.com
      --encryption-key=<Your-Encryption-Key>
      --upstream-url=http://app:7681/
    networks:
      - transit_servicetest_traefik
      - internal

  # Here should be your deployment...
  # Be aware to set the following label: "traefik.enable=false"
```
**Important:** The proxy and the deployment must be in the transit network of traefik.
