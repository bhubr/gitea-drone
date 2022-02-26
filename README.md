# Gitea/Drone

This contains:

* The `docker-compose.yml` file for starting Gitea, MySQL, Drone Server and Drone Runner
* A `.drone.yml` file for testing stuff

Three folders are ignored:

* `mysql` hosting the Gitea db
* `gitea` containing the Gitea files
* `drone-server`

## :warning: **IMPORTANT** :warning:

Add this line to the hosts file (`/etc/hosts` on Linux/MacOS, `C:\WINDOWS\SYSTEM32\Drivers\etc\hosts` on Windows).

```
127.0.0.1	gitea
```

Otherwise check [How to access host port from docker container](https://stackoverflow.com/questions/31324981/how-to-access-host-port-from-docker-container)

## Noob errors

* `DRONE_SERVER_HOST` is the host address from the outside (not for inside the Docker network)
* Forgot to put Drone Server & Client on the same network as Gitea & MySQL :sweat_smile:

## TODO

* Don't hardcode secrets in Compose file!