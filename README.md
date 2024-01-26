# msdUmbrella

Ce repository "chapeaute" les packages de la démo microservices webMethods:
-   https://github.com/staillansag/msdOrders pour le microservice orders
-   https://github.com/staillansag/msdNotifications pour le microservice notifications
-   https://github.com/staillansag/msdPerformance pour le microservice performance

Dans une approche microservice, on va chercher à gérer une image pour chaque microservice. Chaque conteneur de microservice tournera sur un MSR dédié.
Mais on peut également suivre une approche plus pragmatique et rassembler les packages dans une même image. Chaque conteneur contiendra alors un MSR et les 3 packages.

##  Setup de l'environnement de développement

Niveau environnement de développement, on peut également choisir d'avoir un MSR par microservice.  Même si le MSR consomme relativement peu de ressources (disons 1 coeur et 1 Go de mémoire), je préfère quand même rassembler tous mes packages sur un même MSR de développement, déployé sur mon poste local et branché au Designer.  

Pour installer ce MSR on a plusieurs options:
1.  l'installer SAG
2.  télécharger le bundle Designer + MSR: https://tech.forums.softwareag.com/t/webmethods-service-designer-download/235227
3.  utiliser Docker

La troisère option est un peu plus complexe, mais elle a l'énorme avantage de permettre de développer avec l'image de base des microservices:
-   Tous les packages SAG, les packages des frameworks et les drivers sont pré-installés
-   L'environnement de développement est standardisé, tous les développeurs développent et testent sur le même runtime MSR

Grâce à Docker Compose, on peut même facilement déployer une base de données et un UM en local.  Un docker-compose.yml est fournit pour ce faire, pour déployer:
-   une base Postgres conteneurisée
    -   avec un user "postgres" ayant un mot de passe "Password123@" qui peut être modifé ou variabilisé
    -   en gérant la persistance des données grâce à un volume Docker
-   un realm UM conteneurisé
    -   instancié à partir de l'image produit officielle SAG
    -   avec de la persistance de la configuration et des données grâce à des volumes Docker
    -   la license est injectée par le biais d'un autre volume Docker, pointant vers l'emplacement local ./license/um-license.xml (il faut donc positionner un fichier de license UM à cet emplacement)
-   un runtime MSR conteneurisé
    -   instancié avec l'image de base générée par le biais de https://github.com/staillansag/msdBase
    -   la configuration se fait avec un fichier properties injecté par le biais d'un volume Docker, pointant vers l'emplacement local ./application.properties (voir l'exemple application.properties.example fournit dans ce repo)
    -   la license est injectée par le biais d'un volume Docker, pointant vers l'emplacement local ./license/msr-license.xml (il faut donc positionner un fichier de license MSR à cet emplacement)
    -   la persistance des packages est également gérée par des volumes Docker, pointant vers des répertoires locaux

Procédure pour faire le setup:
1.  Créer un répertoire msdemo (ou n'importe quel autre nom)
2.  Se positionner dans ce répertoire et cloner ce repository: `git clone https://github.com/staillansag/msdUmbrella.git`
3.  Aller dans msdUmbrella et renommer le fichier application.properties.example et le modifier (seules les propertiers globalvariable.SMTP* doivent être modifiées, le reste ne bouge pas)
4.  Toujours dans msdUmbrella, créer un répertoire license et y positionner les fichiers um-license.xml et msr-license.xml
5.  Cloner les repositories des packages dans le répertoire msdemo (et pas dans msdUmbrella)
```
git clone https://github.com/staillansag/msdOrders.git
git clone https://github.com/staillansag/msdNotifications.git
git clone https://github.com/staillansag/msdPerformance.git
```
6.  Retourner dans msdUmbrella et démarrer le stack Docker Compose: `docker compose up -d` (dans certains environnements, ce sera docker-compose) et vérifier l'output, qui doit ressembler à ceci:
```
msdUmbrella % docker compose up -d
[+] Running 4/4
 ✔ Network msdumbrella_sag  Created                                                                                                                                                                                                      0.1s
 ✔ Container postgresql     Started                                                                                                                                                                                                      0.0s
 ✔ Container umserver       Started                                                                                                                                                                                                      0.1s
 ✔ Container msdemo         Started
```
7.  Docker expose le MSR au port 15555, il faut donc connecter le Designer sur ce port (par défaut, le user est Administrator et le mdp est manage)

Pour arrêter le stack docker: `docker compose down`  
Pour consulter les logs du MSR (server.log): `docker log msrdemo`  
Pour se connecter au conteneur MSR: `docker exec -it msrdemo sh`  

Pas besoin de gérer la création des connection factories, queues et topics UM. Dans application.properties, le paramètre `jms.DEFAULT_IS_JMS_CONNECTION.jndi_automaticallyCreateUMAdminObjects=true` indique au MSR qu'il doit les créer lui-même automatiquement.  


##  Déploiement d'un "macroservice"
