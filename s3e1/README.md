# S3E1 - automatisation du provisionning d'un serveur de wikis

On souhaite utiliser dokuwiki (une application de gestion de Wiki) pour créer
plusieurs espaces communautaires

* recettes.wiki
* politique.wiki

On souhaite utiliser Ansible pour faciliter et automatiser les multiples
installations.


## Etapes communes

### 1. Mettre en place les DNS pour les deux sites hors de la VM

* Sous Unix/Linux : `vim /etc/hosts`
* Sous MS-Windows : `vim /c/Windows/System32/Drivers/etc/hosts`
      
Le contenu à ajouter est le suivant:

```
127.0.0.1       recettes.wiki 
127.0.0.1       politique.wiki
```
 
### 2. Installer Apache et PHP

Installer Apache et PHP avec le gestionnaire de paquets du système

    apt-get install apache2 php7.4 

:warning: Attention à bien utiliser la version présente sur votre systeme
(7.0 ? 7.3 ? 7.4 ? autre ?)

### 3. Télécharger dokuwiki 

Télécharger la dernier version stable de dokuwiki et mettre le fichier
téléchargé dans `/usr/src/dokuwikiXXXXXXX.zip`

    wget -O /usr/src/dokuwiki.tgz \
      https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz 

### 4. Dézipper dokuwiki

On extrait l'archive de dokuwiki dans `/usr/src/dokuwiki`

       cd /usr/src
       tar xavf dokuwiki.tgz
       mv dokuwiki-XXXXXXXXXXXX dokuwiki

:warning: Adapter la commande en fonction de la version de Dokuwiki

### 5. Créer les virtualhosts

Avec la méthode décrite à la section suivante, on cherche à créer deux virtualhosts :

* Un virtualhost pour recettes.wiki
* Un virtualhost pour politique.wiki


## Etapes pour créer un virtualhost XXX

### 1. Créer un dossier pour les données du site XXX

Les données seront placées dans `/var/www/XXX`

    mkdir -p /var/www/XXX/

### 2. Installer dokuwiki dans le site XXX

Copier le contenu de dokuwiki dans `/var/www/XXX`

    rsync -av /usr/src/dokuwiki/ /var/www/XXX/

### 3. Autoriser apache à écrire dedans

Pour autoriser apache à écrire dans le dossier, il faut changer la propriété du
dossier et l'attribuer l'utilisateur `www-data`.

    cd /var/www
    chown -R www-data:www-data XXX/

### 4. Créer un fichier de configuration pour XXX

Créer un fichier de configuration pour apache dans `/etc/apache2/site-available/XXX.conf` inspiré de `000-default.conf`.

* indice n°1 : vous pouvez utiliser la commande `sed` de Unix
* indice n°2 : vous pouvez utiliser un `template`
* indice n°3 : voir pouvez aussi ajouter/des modifier des lignes avec ansible avec `lineinfile`

:warn: Attention, il est important que le nom du fichier
`/etc/apache2/site-available/XXX.conf` ne comporte pas de point '.' avant le
`.conf`

* ... sinon Apache n'est pas capable de le prendre en compte. 
* penser donc à réécrire le nom XXX si besoin (ex: `politique.wiki` &rArr; `politique-wiki` )

### 5. Activer la configuration 

Pour activer un virtualhost dans apache2:

    a2ensite XXX
    systemctl reload apache2

### 6. Ajouter le DNS pour XXX dans `/etc/hosts` de la VM :

      127.0.0.1   XXX

### 7. Tester

* En local depuis la VM, utiliser un navigateur en ligne de commande, comme w3m
  ou lynx pour vérifier que le site fonctionne bien
* Depuis votre Host, vérifier que la redirecton de port est bien en place et
  utiliser votre navigateur (ex: Firefox) pour accéder au site.