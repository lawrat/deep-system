# Deep-in-System Project: Server Configuration and Services Setup

## Prérequis

- **VM Ubuntu Server**
  Telecharger Ubuntu server depuis le site officiel https://ubuntu.com/download/desktop

Mettre à jour les dépôts et installer VirtualBox :

```bash
sudo apt update
sudo apt install virtualbox
```

- **Disque de 30 Go** partitionné comme suit :
  - `/` : 15 Go (Système principal)
  - `/home` : 5 Go (Utilisateurs)
  - `swap` : 4 Go (Mémoire virtuelle)
  - `/backup` : 6 Go (Sauvegardes)

## Étape 1 : Installation de la VM et Partitionnement

### 1.1 Installation de la VM

Téléchargez l'ISO d'Ubuntu Server et créez une VM avec les spécifications suivantes :

- Processeur : 2 vCPU
- Mémoire : 2 Go
- Disque dur : 30 Go (Partitionnement manuel)

### 1.2 Partitionnement du Disque

Durant l'installation d'Ubuntu, partitionnez manuellement :

- `/` : 15 Go (ext4)
- `/home` : 5 Go (ext4)
- `swap` : 4 Go
- `/backup` : 6 Go (ext4)

### 1.3 Configuration du Hostname

- Définissez le nom d'hôte avec la commande suivante :

```bash
sudo hostnamectl set-hostname {votre_login}-host
```

## Étape 2 : Configuration Réseau

### 2.1 Configuration d'une IP statique

Modifiez le fichier Netplan (`/etc/netplan/01-netcfg.yaml`) pour définir une IP privée statique :

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Appliquez les modifications :

```bash
sudo netplan apply
```

### 2.2 Vérification de la Connexion Internet

Testez la connexion :

```bash
ping -c 5 google.com
```

## Étape 3 : Sécurité

### 3.1 Désactivation de la Connexion Root via SSH

Éditez le fichier de configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

- Modifiez `PermitRootLogin` à `no`.

Redémarrez SSH :

```bash
sudo systemctl restart sshd
```

### 3.2 Modification du Port SSH

Changez le port SSH à `2222` dans le fichier `/etc/ssh/sshd_config` :

```bash
Port 2222
```

Redémarrez SSH :

```bash
sudo systemctl restart sshd
```

### 3.3 Configuration du Pare-feu (UFW)

Activez et configurez UFW :

```bash
sudo ufw default deny incoming
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw enable
```

## Étape 4 : Gestion des Utilisateurs

### 4.1 Création d'Utilisateurs

Créez les utilisateurs `luffy` et `zoro` :

```bash
sudo adduser luffy
sudo usermod -aG sudo luffy
sudo adduser zoro
```

### 4.2 Configuration SSH pour `luffy`

Générez et configurez les clés SSH pour `luffy` :

```bash
ssh-keygen -t rsa
ssh-copy-id luffy@192.168.1.100
```

## Étape 5 : Installation des Services

### 5.1 Serveur FTP (vsftpd)

Installez et configurez vsftpd :

```bash
sudo apt-get install vsftpd
```

Configurez un utilisateur `nami` avec accès à `/backup` :

```bash
sudo adduser nami
sudo usermod -d /backup nami
```

### 5.2 Serveur MySQL

Installez MySQL :

```bash
sudo apt-get install mysql-server
```

Sécurisez l'installation :

```bash
sudo mysql_secure_installation
```

Créez une base de données et un utilisateur pour WordPress :

```bash
CREATE DATABASE wordpress_db;
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost' IDENTIFIED BY 'password';
```

## Étape 6 : Installation de WordPress

### 6.1 Installation

Installez WordPress :

```bash
sudo apt-get install wordpress
```

Configurez la base de données dans `wp-config.php`.

## Étape 7 : Sauvegardes Automatisées

### 7.1 Configuration des Sauvegardes via cron

Ajoutez un job cron pour la sauvegarde de la base de données :

```bash
crontab -e
```

Ajoutez cette ligne :

```bash
0 0 * * * mysqldump -u wp_user -p'password' wordpress_db | gzip > /backup/wordpress_db_$(date +\%F).sql.gz
```

## Étape 8 : Exportation de la VM

### 8.1 Exportation au format OVA

Exportez la VM au format `.ova` via votre hyperviseur, puis générez un fichier de contrôle :

```bash
sha1sum deep-in-system.ova > deep-in-system.sha1
```

Vérifier l'empreinte : Ensuite, assurez-vous que le fichier SHA1 a bien été généré :

```bash
cat deep-in-system.sha1 | cat -e
```

## Pusher maintenant deep-in-system.sha1 et README.md dans votre repository

## Conclusion

Ce projet a permis de configurer un serveur complet avec partitionnement, services, gestion des utilisateurs et sécurité. Les compétences acquises incluent la configuration réseau, la gestion des services FTP et MySQL, ainsi que la mise en place d'un environnement WordPress sécurisé.
