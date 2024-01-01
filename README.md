A-Description du projet :

-Le projet app-deploy est un exemple pour mettre en place une IaC(Infrastructure as Code).
Avant de déployer une application dans Kubernetes il faut écrire des manifests. En effet, Mon projet app-deploy respecte  les bonnes pratiques (personnalisation des fichiers YAML  de nos ressources k8s en utilisant l'outil kustomize ).
kustomize est un outil Kubernetes qui  permet de personnaliser les fichiers YAML bruts de nos ressources k8s d'origine à des fins multiples (ex: différents environnements, différentes variables/ressources informatique, etc ...).




B-Integration du projet :

Pour integrer le projet app-Deploy dans vos projets, il faut respecter les etapes suivantes:

1- Creer un  projet  'app' qui va contenir l'application

2- Modifier le fichier Dockerfile pour decrire la recette de construction du  projet 'app':

Exemple:

- FROM quay.io/wildfly/wildfly
- ARG WAR_FILE=target/*.war
- ARG VERSION
- LABEL "app.version"="${VERSION}"
- LABEL "app.contact"="test@gmail.com"
- COPY ${WAR_FILE} "/opt/jboss/wildfly/standalone/deployments/app-binary.war"
- COPY "./docker/modules/com/ops" "/opt/jboss/wildfly/modules/com/ops"
- COPY "./docker/config/standalone.xml" "/opt/jboss/wildfly/standalone/configuration/standalone.xml" 

3- Creer a la racine du projet 'app' le fichier .gitlab-ci.yaml qui est le point de départ de tout pipeline Gitlab pour definir des jobs et des stages:
Les bonnes pratiques à repecter au niveau de la definition des stages:

-Pour deployer l'image de l'appli en dev: faudrait creer un stage 'deploy-dev' pour pousser l'image scratch (generée du build de la branche dev: snapshot ) dans dockerHub et mettre à jour la versoin de l'appli dans le projet 'app-deploy' (environnement - dev).

-Pour deployer l'image de l'appli en uat: faudrait creer un stage 'deploy-uat' pour pousser l'image stable (generée du build de la branche uat: release ) dans dockerHub et mettre à jourla versoin de l'appli dans le projet 'app-deploy' (environnement - uat) 

-Pareil pour les autres environnement




C- ArgoCd

Pour plus d'information sur l'outil ArgoCd:
-L'outil ArgoCd  permet de contrôler le déploiement d'une application sur les differents environnements. Il a besoin de deux  paramètres d'entrée:
-Le projet 'app-deploy' 
-Le docker Hub
Si la version de l'appli change dans 'app-deploy', argoCd fait la synchronisation automatique pour deployer la bonne version sur le bon environnement.
