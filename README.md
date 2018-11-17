# TP1 Cloud : Architectures complexes cloud-like
## Prérequies
### Vagrantfile
Ajouter un disque 10 GB
```
unless File.exist?("ddplus0#{i}.vdi")
    vb.customize ['createmedium', '--filename', "ddplus0#{i}.vdi", '--size', 10 * 1024]
end
vb.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--port', 1, '--device', 1, '--type', 'hdd', '--medium', "ddplus0#{i}.vdi"]
```
## Docker Swarm
### Configurer le docker swarm
### Installer docker-component
```
sudo mkdir -p /opt/bin
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /opt/bin/docker-compose
sudo chmod +x /opt/bin/docker-compose
```
### Lancer le service applicatif
#### Mettre les images disponible
Sur chaque VM récupérer la correction du tp précédent (docker-component.yml, python-app/ et proxy)
modifier docker-component.yml
-`image: m1.swarm:5000/python-app:latest`
+`image: python-app:latest`

-`image: m1.swarm:5000/app-proxy:latest`
+`image: app-proxy:latest`

Monter les images
`docker-compose build`
### Lancer les services
`docker stack deploy python_dirty_app -c docker-compose.yml`
# Weave Cloud
```
sudo curl -L git.io/scope -o /opt/bin/scope
sudo chmod a+x /opt/bin/scope
scope launch --service-token={TOKEN}
```
**Q1 : comment ce conteneur a-t-il pu lancer des conteneurs sur votre hôte ?**
Pour fonctionner Weave cloud lance un agent installé sur le poste. C'est ce dernier qui initie le conteneur 
# Ceph
**Q2 : expliquer rapidement le principe d'un système de fichiers distribué** (distributed filesystem)
Un sytème de fichier distribué est un système de fichier permettant de partager des fichier entre hôtes via un réseau. L'hôte distant n'a jamais accès au système de fichier mais échange avec lui via un protocole.

**Q3 : proposer une façon d'automatiser le déploiement cette conf CEPH** (Vagrant ? Swarm stack ? autres ?) Par ex, si on veut rajouter un noeud ?
Un moyen simple d'automatiser cette configuration serait de copier les fichiers de conf sur notre hote et les déplacer dans nos VM via le provisionning de notre Vagrantfile

# Registry - Part 1
**Q4 : où est lancé le service réellement ? (sur quel hôte, et comment on fait pour savoir ?)** Combien y'a-t-il de conteneur(s) lancé(s) ?
Un docker ps sur les hosts permet de voir qu'ici le registry est lancé sur la machine core-02
```
core@core-02 ~ $ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
edfe2290fbf0        registry:2               "/entrypoint.sh /etc…"   4 minutes ago       Up 4 minutes        5000/tcp            registry.1.ixaquyhkux1i8g3qkiteg1vzp
```

**Q5 : pourquoi le service est accessible depuis tous les hôtes ?** Documentez vous sur internet.
Un docker service ls permet de voir que si le régistry est effectivement lancé sur une seule machine il fonctionne comme un service du docker swarm, accessible donc à toutes les machines qu'on y a intégré
# Keepalived
**Q6 : que fait le `--net=host` de la commande exactement ? Je veux voir une utilisation de `nsenter` en réponse à cette question.** Pourquoi avoir besoin de ça sur Keepalived ? 
`--net=host` permet au conteneur keepalived de créer un pont avec avec le réseau du swarm. Accès indispensable à l'application pour servir d'IP Virtuelle.
nsenter est une application permettant d'acceder à un proccessus directement via le PID de ce dernier.
```
core@core-02 ~ $ PID=$(docker inspect --format {{.State.Pid}} keepalived)
core@core-02 ~ $ nsenter --target $PID --mount --uts --ipc --net --pid
core-02:/# ip a | grep "eth1"
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP qlen 1000
    inet 192.168.100.102/24 brd 192.168.100.255 scope global eth1
    inet 192.168.100.100/32 scope global eth1
```
**Q7 : le principe de priorité au sein de Keepalived et le fonctionnement simplifié de `vrrp`** (schéma si vous voulez)
vrrp est un protocole permettant la haute disponibilité en plaçant un groupe de routeur derrière une IP Virtuelle et une MAC Virtuelle. vrrp créé des routeur virtuel associé à cette IP, les routeur ont un numéro de priorité (allant de 1 à 255) permmettant de déterminer celui prenant la main (le maitre) et ceux placé en attente (les secours) qui pourront reprendre le rôle de maitre en cas de panne. Le routeur avec la priorié la plus haute est choisis pour être le maitre.
# Show me your metrics
**Q8 : ce qu'est un 'collector' dans ce contexte**
Un collector est un outil permettant d'agréger un grand nombre d'information sur un système et les présenter sous différents format.
