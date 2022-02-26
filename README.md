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

**This (probably should) be the same for the `drone` host: make it have the same URL from inside as from outside the Docker network.

Otherwise webhooks fail:

```
gitea_1         | 2022/02/26 16:20:34 ...s/webhook/deliver.go:253:DeliverHooks() [E] deliver: Post "http://localhost:8080/hook?secret=N9Jrij0g8aiRZlRBTAgSkbnprODpyuZy": dial tcp 127.0.0.1:8080: webhook can only call allowed HTTP servers (check your webhook.ALLOWED_HOST_LIST setting), deny 'localhost(127.0.0.1:8080)'
```

Workaround: in `http://localhost:3000/<user>/<repo>/settings/hooks/<id>`, change URL from `http://localhost:8080` to `http://drone:80`.

## Gitea security settings

Another error

```
gitea_1         | 2022/02/26 16:24:05 ...s/webhook/deliver.go:253:DeliverHooks() [E] deliver: Post "http://drone:8080/hook?secret=N9Jrij0g8aiRZlRBTAgSkbnprODpyuZy": dial tcp 172.23.0.4:8080: webhook can only call allowed HTTP servers (check your webhook.ALLOWED_HOST_LIST setting), deny 'drone(172.23.0.4:8080)'
```

Add this to Gitea conf inside Gitea volume: `~/gitea/conf/app.ini`.

```
[webhook]
ALLOWED_HOST_LIST = drone
```

## ALSO in Gitea's repos...

In Drone server's SQLite db:

```
UPDATE repos SET repo_clone_url = 'http://gitea:3000/benoit/gitea-drone.git' WHERE repo_id = 1;
```

## Noob errors

* `DRONE_SERVER_HOST` is the host address from the outside (not for inside the Docker network)
* Forgot to put Drone Server & Client on the same network as Gitea & MySQL :sweat_smile:

## Ideas

To get same hostnames inside containers/VMs than outside

* Using Acrylic, localdns, dnsmasq
* SSH tunnel
* Reverse proxy from nginx/apache/caddy on localhost

## Stuff that helped

* [[SOLVED] Making drone listen on port different than 443](https://discourse.drone.io/t/solved-making-drone-listen-on-port-different-than-443/8274)
* [Drone Server / Gitea](https://docs.drone.io/server/provider/gitea/)

## Run Drone Server without Compose
```
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITEA_SERVER=https://try.gitea.io \
  --env=DRONE_GITEA_CLIENT_ID=05136e57d80189bef462 \
  --env=DRONE_GITEA_CLIENT_SECRET=7c229228a77d2cbddaa61ddc78d45e \
  --env=DRONE_RPC_SECRET=super-duper-secret \
  --env=DRONE_SERVER_HOST=drone:8080 \
  --env=DRONE_SERVER_PORT=:8080 \
  --env=DRONE_SERVER_PROTO=https \
  --publish=8080:8080 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```

## TODO

* Don't hardcode secrets in Compose file!

## Conclusion

It works!
