# tp1-terraform-ecommerce

## Question 1
- `service_name` configure le nom du conteneur et la variable d’environnement `SERVICE_NAME`.
- `external_port` configure le port d’accès depuis l’extérieur et l’URL générée par Terraform.

## Question 2
La ressource `docker_network.ecommerce` crée un réseau Docker isolé (`ecommerce-net`) permettant aux futurs microservices de communiquer entre eux. Elle est définie séparément du conteneur pour gérer indépendamment son cycle de vie : le réseau persiste même si un conteneur est modifié ou recréé, et plusieurs services peuvent s’y connecter sans dupliquer la configuration du réseau.

## Question 3
L’utilisation de Terraform change la manière de déployer le projet fil rouge par rapport à docker-compose ou à des scripts Bash. Avec docker-compose ou des scripts, le déploiement est impératif : il faut lancer des commandes manuellement et l’état réel dépend souvent de ce qui a été fait précédemment. Terraform adopte une approche déclarative : on décrit l’état souhaité de l’infrastructure (réseau, conteneurs, ports, variables), puis Terraform calcule automatiquement ce qui doit être créé, mis à jour ou détruit. Cela garantit un déploiement reproductible, cohérent et versionnable, ce qui facilite la maintenance du projet fil rouge et évite les dérives de configuration.

## Question 4
Pour généraliser ce TP et déployer automatiquement plusieurs microservices de l’e-commerce (catalogue, panier, commande, auth…) avec un seul `terraform apply`, il faut structurer l’infrastructure en plusieurs blocs cohérents :

### 1. Réseau commun
- Créer un unique réseau Docker (`docker_network.ecommerce`).
- Tous les microservices s’y connectent, ce qui permet la communication interne.

### 2. Une ressource par microservice
Pour chaque microservice :
- un `docker_image` (ou un build local du Dockerfile du service),
- un `docker_container` avec :
  - un nom unique (`catalog-service`, `cart-service`, etc.),
  - un port externe propre,
  - les variables d’environnement nécessaires (ex : `AUTH_SERVICE_URL=http://auth:5001`).

### 3. Utilisation de variables Terraform
- Ports externes paramétrables,
- Noms des microservices,
- URLs internes pour l’appel d’API.
Cela permet d’ajuster facilement l’infrastructure sans dupliquer du code.

### 4. Factorisation du code
Deux approches possibles :
- **Modules Terraform** : créer un module « microservice » réutilisable.
- **for_each** : générer automatiquement plusieurs conteneurs à partir d’une liste.

### 5. Résultat final
Un seul `terraform apply` permet de :
- créer le réseau,
- construire ou récupérer toutes les images Docker,
- déployer chaque microservice correctement configuré,
- exposer les ports publics nécessaires.

Cette approche offre un déploiement reproductible, cohérent et automatisé pour l’ensemble du projet e-commerce.
