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

* Generate RPC secret with `openssl rand -hex 16`
* Don't hardcode secrets in Compose file!

## Conclusion

## Setup sur Alpine

### Setup initial Alpine

Installation `sudo`

```
apk update
apk add sudo
```

Ajout user `alpine`

```
adduser alpine
addgroup alpine wheel
echo '%wheel ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/wheel
```

### Simplification mdp root

De `Alpine31` à `alpine`.

### Génération et transfert des clés SSH

```
ssh-keygen -t ecdsa
```

Pas de passphrase, sauvegarder sous `~/.ssh/ssh-keygen -t ecdsa`

```
ssh-copy-id -i ~/.ssh/id_ecdsa_alpine_ipi.pub -p 4099 alpine@localhost
```

Puis login passwordless :

```
ssh -i ~/.ssh/id_ecdsa_alpine_ipi -p 4099 alpine@localhost
```

> **VM exportée à ce stade sous le nom `AlpineBaseStep01.ova`** (57 Mo)

**Suite à "step01"**, il reste :

Reste :

* [ ] Bash
* [x] Python3 (pour Ansible)
* [x] Inet static
* [ ] **Désactiver** inet static temporairement
* [ ] Prompt coloré
* [ ] **Hostnames distincts**
* [x] Clé pub contrôleur Ansible à transférer vers clone

### IP statique

Ajouter à `/etc/network/interfaces` (changer l'IP éventuellement)

```
auto eth1
iface eth1 inet static
  address 192.168.56.23
  netmask 255.255.255.0
```

### Copie de la clé de base de la VM vers le clone

Vu que ce sont des clones, ne suffisait-il pas (par hasard ?) de copier la clé publique déjà présente dans la VM, dans `authorized_keys` ?

Du contrôleur

```
ssh-copy-id -i .ssh/id_ecdsa.pub 192.168.56.23
```

> **VM exportée à ce stade sous le nom `AlpineBaseStep02.ova`** (73 Mo)

### Installation de bash et nano

### Gérer manuellement le supervise daemon

#### Lancer `start-stop-daemon` de Gitea

```
supervise-daemon gitea --start --pidfile /run/gitea.pid --user gitea --env GITEA_WORK_DIR=/var/lib/gitea --chdir /var/lib/gitea --stdout /var/log/gitea/http.log --stderr /var/log/gitea/http.log /usr/bin/gitea -- web --config /etc/gitea/app.ini
```

#### Tuer `start-stop-daemon` de Gitea

`/etc/init.d/gitea stop` ne fonctionne pas. Ceci fonctionne :

```
start-stop-daemon --signal KILL --pidfile /run/gitea.pid
```

Mais cela ne suffit pas, il faut encore tuer l'instance de Gitea :

```
kill -9 `ps a | grep gitea | head -n 1 | awk '{ print $1 }'`
```

Ou plus simplement :

```
killall gitea
```

## Remettre les VMs à zéro

### VM Gitea

Ancienne façon (éventuellement reboot si  `/etc/init.d/gitea stop` échoue) :

```
/etc/init.d/gitea stop
start-stop-daemon --signal KILL --pidfile /run/gitea.pid
kill -9 `ps a | grep gitea | head -n 1 | awk '{ print $1 }'`
sudo apk del gitea
sudo rm -rf /etc/gitea /var/lib/gitea
```

Nouvelle façon :

```
sudo rc-service gitea stop
sudo apk del gitea
sudo rm -rf /etc/gitea /var/lib/gitea /var/log/gitea
```

## VM Docker

```
sudo apk del docker
```

## Changelog

La bonne blague : le playbook que je me suis fait -censuré- à écrire pour résoudre le problème de Gitea 1.15 qui ne s'arrêtait pas normalement&hellip; n'est plus nécessaire si j'active les repos _edge_ d'Alpine, pour installer la 1.16 ! Les `rc-service gitea start` et `rc-service gitea stop` fonctionnent normalement avec cette version, plus la peine de tuer `supervise-daemon` et `gitea` manuellement.

## TODO

* Améliorations Gitea :

    * virer passwd hardcodés
    * créer un nombre de comptes utilisateurs
* Scripter clonage machine : [VBoxManage clonevm](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vboxmanage-clonevm.html)

## Liens

* [ansible: lineinfile for several lines?](https://stackoverflow.com/questions/24334115/ansible-lineinfile-for-several-lines)