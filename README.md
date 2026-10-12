###9. Personnaliser la page Web
***Question : quelle est la différence entre un montage bind et la copie d'un fichier dans une image Docker ?***
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

