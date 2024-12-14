# Projet Ansible : Déploiement de GitLab et PostgreSQL

## 📖 Description du Projet

Ce projet utilise **Ansible** pour automatiser le déploiement d’une infrastructure composée de :  
- **GitLab** : Une plateforme de gestion de code source et de collaboration d’équipe.  
- **PostgreSQL** : Un système de gestion de bases de données relationnelles.  

L’objectif principal est de garantir une configuration automatisée, reproductible et centralisée, afin de simplifier la gestion de l’infrastructure.

---

## 📂 Structure des Rôles Ansible

### 🔧 Rôle : GitLab

**Variables :**  
- `gitlab_domain` : Spécifie le nom de domaine pour accéder à GitLab.  
- `gitlab_storage_path` : Définit le chemin de stockage des données de GitLab.  

**Tâches principales :**  
1. Téléchargement du script d’installation officiel de GitLab.  
2. Exécution du script pour configurer les dépôts nécessaires.  
3. Installation de GitLab Enterprise Edition (EE).  
4. Configuration des paramètres suivants :  
   - `external_url` pour définir le domaine.  
   - Chemin de stockage des données.  
5. Application des configurations via `gitlab-ctl reconfigure`.  

**Handler :**  
Un handler est défini pour redémarrer GitLab après toute modification des configurations.

---

### 🔧 Rôle : PostgreSQL

**Variables :**  
Aucune variable spécifique n’est définie pour ce rôle.

**Tâches principales :**  
1. Installation de PostgreSQL.  
2. Configuration du fichier `pg_hba.conf` :  
   - Autorisation des connexions locales avec la méthode `trust`.  
   - Modification pour permettre les connexions distantes (`0.0.0.0/0`).  
3. Création des bases de données suivantes :  
   - `all`, `dev`, `stage`, `prod`.  
4. Création de l’utilisateur `vagrant` avec :  
   - Tous les privilèges sur la base `all`.  
   - La propriété des bases `dev`, `stage` et `prod`.  

**Handler :**  
Un handler est configuré pour redémarrer PostgreSQL si des modifications sont apportées à la configuration.

---

## 📄 Fichiers Importants

### 1. Inventaire (`inventory.yml`)

Ce fichier définit les hôtes cibles et leurs paramètres. Voici un exemple de configuration :  

#### Structure de l’inventaire :  
- **`gitlab-server`** : Contient les paramètres de configuration pour le serveur GitLab.  
- **`postgres-server`** : Définit les paramètres pour le serveur PostgreSQL.  

```yaml
all:
  children:
    gitlab:
      hosts:
        gitlab-server:
          ansible_host: 192.168.224.130
          ansible_user: root
    postgres:
      hosts:
        postgres-server:
          ansible_host: 192.168.224.131
          ansible_user: root
          ansible_python_interpreter: /usr/bin/python3
```
---

## 2. Variables GitLab (`roles/gitlab/vars/main.yml`)

Ce fichier contient les variables utilisées pour configurer GitLab :  

- `gitlab_domain` : Définit le nom de domaine pour accéder à l'instance GitLab.  
- `gitlab_storage_path` : Spécifie le chemin où seront stockées les données GitLab.  

Exemple de contenu du fichier :  

```yaml
gitlab_domain: gitlab.vlne.lan
gitlab_storage_path: "/var/opt/gitlab/git-data"
```
---

## 🚀 Instructions d’Utilisation

### Prérequis :  

1. **Installation d’Ansible** : Assurez-vous qu'Ansible est installé sur votre machine de contrôle.  
   - Sous Linux : Vous pouvez l’installer via `apt`, `yum`, ou `dnf` selon votre distribution.  
   - Sous macOS : Utilisez `brew install ansible`.  

2. **Accès SSH** : Configurez un accès SSH entre votre machine de contrôle et les hôtes cibles. Assurez-vous que les clés SSH sont en place pour permettre une authentification sans mot de passe.  

### Commande d’exécution :  

Pour lancer le playbook et déployer GitLab et PostgreSQL :  

```bash
ansible-playbook -i inventory.yml deploy_gitlab.yml
```
Cette commande exécute toutes les tâches nécessaires pour configurer l’infrastructure définie dans les rôles, y compris :  

1. **Installation de GitLab** :  
   - Téléchargement et exécution du script d’installation officiel.  
   - Configuration du domaine (`external_url`) et du chemin de stockage.  
   - Redémarrage pour appliquer les changements.  

2. **Installation et configuration de PostgreSQL** :  
   - Installation du serveur PostgreSQL.  
   - Modification des paramètres de connexion pour autoriser l’accès local et distant.  
   - Création des bases de données nécessaires et de l’utilisateur `vagrant` avec les permissions adéquates.  

---

### Vérification après l’exécution  

#### Étape 1 : Modifier le fichier `hosts` local  

Pour accéder à GitLab via le domaine configuré, vous devez ajouter l’adresse IP du serveur GitLab à votre fichier `hosts` local :  

- Sous **Linux** : ouvrez le fichier `/etc/hosts` avec une commande comme :  
  ```bash
  sudo nano /etc/hosts
  ```

Sous **Windows** : éditez le fichier `C:\Windows\System32\drivers\etc\hosts` en utilisant un éditeur de texte avec des droits administrateur.  

Ajoutez la ligne suivante à la fin du fichier :  

  ```yaml
  192.168.224.130 gitlab.vlne.lan
  ```

#### Étape 2 : Accéder à GitLab  

Une fois cette modification effectuée, ouvrez un navigateur et accédez à :  
[http://gitlab.vlne.lan](http://gitlab.vlne.lan)  
Vous devriez voir la page d’accueil de GitLab.

