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
Q1) Pour fonctionner Weave cloud lance un agent installé sur le poste. C'est ce dernier qui initie le conteneur 
# Ceph
Q2) Un sytème de fichier distribué est un système de fichier permettant de partager des fichier entre hôtes via un réseau. L'hôte distant n'a jamais accès au système de fichier mais échange avec lui via un protocole.

Q3)


dépot git
nom:url envoyer maintenant
rendre dimanche 18/11 23h59
racine du dépot fichier 
	names.txt
		1 nom par ligne
