A-Description du projet :

-Le projet app-deploy est un exemple pour mettre en place une IaC(Infrastructure as Code).
Avant de déployer une application dans Kubernetes il faut écrire des manifests. En effet, Mon projet app-deploy respecte  les bonnes pratiques (personnalisation des fichiers YAML  de nos ressources k8s en utilisant l'outil kustomize ).
kustomize est un outil Kubernetes qui  permet de personnaliser les fichiers YAML bruts de nos ressources k8s d'origine à des fins multiples (ex: différents environnements, différentes variables/ressources informatique, etc ...).




B-Integration du projet :

Pour integrer le projet app-Deploy dans vos projets, il faut respecter les etapes suivantes:
1- Creer un  projet  'app' qui va contenir l'application
2- Modifier le fichier Dockerfile pour decrire la recette de construction du  projet 'app':
Exemple:

FROM quay.io/wildfly/wildfly
ARG WAR_FILE=target/*.war
ARG VERSION
LABEL "app.version"="${VERSION}"
LABEL "app.contact"="nagmouchi.azza@gmail.com"
COPY ${WAR_FILE} "/opt/jboss/wildfly/standalone/deployments/app-binary.war"
#creer un package '/docker/modules/com/op' au niveau de la racine du projet 'app' en ajoutant #le fichier module.xml :
#creer un package '/docker/config' au niveau de la racine du projet 'app' en ajoutant le #fichier standalone.xml :
#Exemple du module xml:
#<module xmlns="urn:jboss:module:1.5" name="com.ops.config">
     #    <resources>
     #		<resource-root path="/config"/>
     #    </resources>
#</module>
#l'instruction suivante permet de copier ce fichier module.xml dans les modules de wildfly
COPY "./docker/modules/com/ops" "/opt/jboss/wildfly/modules/com/ops"
#l'instruction suivante permet de specifier la partie config à wildfly
#Dans le fichier de conf standalone.xml, on utilise des variables comme 'DB_CONNECTION_URL' ,#'WS-URL' ...
#Ces variables dependent de l'environnement d'execution (dev, int, pprod, prod), elles seront #instantiées
#dans le projet 'app-deploy'
COPY "./docker/config/standalone.xml" "/opt/jboss/wildfly/standalone/configuration/standalone.xml" 

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
