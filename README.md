### 9. Personnaliser la page Web
**Question : quelle est la différence entre un montage bind et la copie d'un fichier dans une image Docker ?**
Avec la copie d'un fichier dans une image Docker, on doit reconstruire l'image alors qu'avec un montage Bind,  
si on modifie le fichier sur la machine, le conteneur voit la modification immédiatement sans reconstruire l'image.  

### 10. Créer un réseau Docker dédié  
Travail demandé :  
**Expliquez pourquoi le conteneur est connecté à ce réseau**  
Le conteneur est connecté au réseau tp-iac-nginx-network car cette connexion est déclarée dans le fichier main.tf,  
dans le bloc network_advanced.  
La ressource docker_network.app_network définit le réseau Docker et permet d'associer le conteneur à ce réseau.  

**Indiquez l'intérêt de déclarer le réseau dans Terraform**  
L'intérêt de déclarer le réseau Docker dans Terraform est de gérer l'infrastructure réseau automatiquement, de  
manière reproductible et cohérente, grâce au code (c'est le principe même de l'IaC).  

**Vérifiez que le réseau est supprimé lors de la destruction du projet**  
Oui, le réseau est bien supprimé lors de la destruction du projet.  

### 11. Limiter l'exposition du service  
**Question : quelle est la différence entre 127.0.0.1 et une écoute sur toutes les interfaces ?**  
Cela permet de renforcer la sécurité en empechant un autre ordinateur du réseau d'accéder à ce service  
via l'adresse IP réseau de l'hôte.    

### 13. Inspecter l'état Terraform  
Questions :  
**Quelles ressources apparaissent dans terraform state list ?**  
Ça affiche la liste des ressources que Terraform suit dans son fichier état.  

>terraform state list  
docker_container.web  
docker_image.nginx  
docker_network.app_network  


**Quelles informations affiche terraform state show ?**  
Ça affiche les attributs détaillés d'une ressource précise enregistrée dans l'état Terraform.  


**Pourquoi l'état Terraform doit-il être protégé ?**  
L'état de Terraform peut contenir des données sensibles (mots de passe, jetons d'accès, clé secrètes etc..)  
et des informations sur l'infrastructure (adresses IP, identifiants etc..).  
Il faut donc le protéger pour éviter qu'il soit modifié ou supprimé accidentellement.  


**Pourquoi faut-il éviter de publier cet état dans un dépôt public ?**  
Publier cet état dans un dépôt public peut exposer ces informations sensibles à n'importe qui.  
Donc pour des raisons de sécurité évidentes on ne le fait pas.  


### 14. Observer une dérive de configuration  
3. Analysez le plan et les différences détectées.  
Changes to Outputs:  
  ~ container_name = "tp-iac-dev" -> "tp-iac-nginx"  
  ~ network_name   = "tp-iac-dev-network" -> "tp-iac-nginx-network"  
  ~ website_url    = "http://localhost:8081" -> "http://localhost:8080"  


### 15. Versionner avec Git
Travail demandé : expliquez comment l'historique Git facilite la revue, la collaboration et le retour sur une modification.
Git est un outil de versionning qui possède un historique et des commandes spéciales pour revenir à la version d'avant si
on veut effectuer un retour sur une modification.  
Grace à d'autres outils comme GitHub il permet de faciliter la revue et la collaboration en permettant à d'autres utilisateurs
d'apporter des modifications au code.  

### 19.Exercices complémentaires  

**Exercice 1 :**   
Changes to Outputs:  
  ~ container_name = "tp-iac-nginx" -> "tp-iac-web"  
  ~ network_name   = "tp-iac-nginx-network" -> "tp-iac-web-network"  

Le com du conteneur a été changé donc son nom ainsi que le nom du réseau associé changent aussi.  


**Exercice 2 :**  
Changes to Outputs:  
  ~ website_url    = "http://localhost:8080" -> "http://localhost:8082"  


**Exercice 3 :**  
variable "network_name" {  
  description = "Nom du réseau Docker"  
  type        = string  
  default     = "tp-iac-web-network"  

  validation {  
    condition     = can(regex("^[a-zA-Z0-9][a-zA-Z0-9_.-]*$", var.network_name))  
    error_message = "Le nom du réseau contient des caractères non autorisés."  
  }  
}  

**Ecercice 4 :**  
Listage des ressources Terraform avec "terraform state list"  
Dans docker_container qui se situe dans main.tf, on trouve :  
image = docker_image.nginx.image_id  
networks_advanced {  
    name = docker_network.app_network.name  
}  
Donc des références directes entre l'image, le réseau et le conteneur.  
Le conteneur a besoin d'une image pour définir son système de fichiers et son environnement.  
Le conteneur doit être connecté à un réseau Docker qui doit exister à l'avance.  

**Exercice 5 :**  
Après avoir arreté le conteneur Docker et lancé la commande terraform plan, le message  
"docker_container.web will be created" apparaît.  

**Exercice 6 :**  
Quand on essaie de push avec une erreur de formatage (ici dans main.tf), on obtient une erreur  
dans nos GitHub Actions :  
 validate  
Terraform exited with code 1.

**Exercice 7 :**  
> terraform plan -destroy  
Changes to Outputs:  
  - container_name = "tp-iac-web" -> null  
  - network_name   = "tp-iac-web-network" -> null  
  - website_url    = "http://localhost:8082" -> null  

Vérification de la destruction des ressources avec terraform state list qui ne doit rien afficher.  
Vérifications supplémentaires avec docker ps -a et docker network ls  
Si on supprime manuellement l'état de Terraform, celà ne supprime pas les conteneurs/images/réseaux Docker.  
De plus, Terraform perd le suivi de ces ressources et des conflits peuvent apparaître.  


### 20. Questions de compréhension  
**1. Quelle est la différence entre terraform init, terraform plan et terraform apply ?**  
terraform init : met en place terraform sans proposer de modification  
terraform plan : propose des modifications, mais sans les appliquer  
terraform apply : applique les modifications proposées  

**2. Quel est le rôle d'un provider Terraform ?** 
Le provider Terraform permet la création de serveurs, réseaux et stockages cloud.  

**3. À quoi sert le fichier terraform.tfstate ?**  
Le fichier terraform.tfstate permet de suivre l'état actuel de terraform ainsi que l'accès au ressources dont il dépend.  

**4. Pourquoi utilise-t-on des variables et des outputs ?**  
On utilise des variables et des outputs pour avoir un contrôle sur ce qu'on envoie en entrée et reçois en sortie plutôt que 
de tout générer automatiquement.  
Celà permet de par exemple choisir un nom spécifique qu'on contrôle avec une RegEx pour valider.  
Afficher une description personnalisée etc..  

**5. Qu'est-ce qu'une dérive de configuration ?**  
Il s'agit d'une différence entre l'état réel du provider (ici Terraform) et l'état souhaité.  

**6. Quelle différence existe-t-il entre Infrastructure as Code et configuration manuelle ?**  
Gestion manuelle :  
- Clics en console : Processus lent et sujet aux erreurs.  
- Serveurs flocons : Configurations uniques et non documentées.  
- Aucun historique : Impossible de savoir qui a modifié quoi.  

Infrastructure as Code :
- Fichiers texte : Définition claire en code informatique.  
- Reproductible : Environnements identiques en 1 commande.  
- Versionné sur Git : Traçabilité et retours en arrière faciles.  

**7. Quel est l'intérêt de Git dans un projet IaC ?**  
Git permet de versionner le code et de pouvoir facilement revenir en arrière dans les modifications.  
GitHub est un ajout qui permet de collaborer entre plusieurs développeurs.  

**8. Quelle est la différence entre un contrôle CI et un déploiement CD ?**  
CI : vérifier que les modifications apportées au code sont correctes et peuvent être intégrées sans introduire de problème.  
CD : préparer ou réaliser la livraison de ces modifications vers un environnement cible, comme la préproduction ou la production.  

**9. Pourquoi faut-il examiner le plan avant d'appliquer une modification ?**  
Il faut examiner le plan avant d'appliquer une modification pour être sûre que chaque modification proposées  
est réellement voulue.  
Une modification non désirée peut avoir des répercutions importantes sur l'infrastructure.  

**10. Quelles précautions faut-il prendre avec les fichiers d'état et les secrets ?**  
Il ne faut jamais commiter les fichiers d'états et les secrets sur un dépôt publique.  
GitHub permet justement de gérer les secrets pour avoir un endroit sécurisé où les stocker.  
