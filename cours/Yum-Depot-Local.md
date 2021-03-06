Dépôt de paquets Yum local
==========================

Document écrit par Nicolas Kovacs <info@microlinux.fr>

Une grande administration régionale m'a contacté récemment pour mettre en place
l'un de leurs serveurs Intranet sous Linux. La particularité de cette machine,
c'est qu'elle n'est pas reliée à Internet, ce qui pose un problème dans la
mesure où je n'ai plus accès aux dépôts de paquets CentOS en ligne. La solution
consiste ici à mettre en place un dépôts de paquets local.

À des fins de démonstration et pour bien expliquer la procédure, je pars d'un
système CentOS 7.0 (installation minimale effectuée à partir du fichier
`CentOS-7.0-1406-x86_64-Minimal.iso`), que je vais successivement mettre à jour
par le biais d'un dépôt local.


Désactiver les dépôts en ligne
-----------------------------

Étant donné que nous ne disposons pas de connexion Internet, la première chose
à faire, c'est de désactiver les dépôts en ligne pour éviter les messages
d'erreur à répétition du gestionnaire de paquets. Éditer
`/etc/yum.repos.d/CentOS-Base.repo` et désactiver les dépôts `[base]`,
`[updates]` et `[extras]`. Le dépôt `[centosplus]` est déjà désactivé dans la
configuration par défaut.

<pre>
[base]
name=CentOS-$releasever - Base
enabled=<strong>0</strong>
...
[updates]
name=CentOS-$releasever - Updates
enabled=<strong>0</strong>
...
[extras]
name=CentOS-$releasever - Extras
enabled=<strong>0</strong>
...
[centosplus]
name=CentOS-$releasever - Plus
enabled=<strong>0</strong>
</pre>

Voilà ce que ça donne.

<pre>
# <strong>yum check-update</strong> 
Modules complémentaires chargés : fastestmirror
Aucun dépôt n'est activé.
</pre>


Récupérer les fichiers ISO
--------------------------

La [page de téléchargement de CentOS](https://www.centos.org/download/) offre
plusieurs fichiers ISO au choix.

  * *DVD ISO*

  * *Everything ISO*

  * *Minimal ISO*

Pour la mise en place d'un dépôt local, on choisira le fichier *Everything ISO*.
Voici tous les fichiers correspondants publiés depuis la sortie de CentOS 7.0.
Chacun pèse près de 7 Go et contient l'ensemble des paquets officiels fournis
par la distribution.

  * `CentOS-7.0-1406-x86_64-Everything.iso`

  * `CentOS-7-x86_64-Everything-1503-01.iso`

  * `CentOS-7-x86_64-Everything-1511.iso`

  * `CentOS-7-x86_64-Everything-1611.iso`

On notera au passage que les développeurs de CentOS ont un peu de mal à se
mettre d'accord sur une convention de nommage consistante des fichiers.

Pour ma part, je range ces fichiers dans un endroit approprié comme par exemple
`/root/iso`. Évidemment, sur une machine de production, on supprimera les
anciennes versions au fur et à mesure pour ne pas encombrer le serveur
inutilement.


Monter l'ISO
------------

Créer un point de montage approprié.

<pre>
# <strong>mkdir -v /mnt/iso</strong> 
mkdir: création du répertoire « /mnt/iso »
</pre>

Monter l'ISO avec les options qui vont bien.

<pre>
# <strong>mount -o loop CentOS-7.0-1406-x86_64-Everything.iso /mnt/iso/</strong> 
mount: /dev/loop0 est protégé en écriture, 
       sera monté en lecture seule
</pre>


Configurer le dépôt local
-------------------------

Créer un fichier `/etc/yum.repos.d/local.repo` et l'éditer comme ceci.

<pre>
[LocalRepo]
name=Local Repository
baseurl=file:///mnt/iso
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
</pre>

Nettoyer le cache.

<pre>
# <strong>yum clean all</strong> 
Modules complémentaires chargés : fastestmirror
Nettoyage des dépôts : LocalRepo
Cleaning up everything
Cleaning up list of fastest mirrors
</pre>

À partir de là, Yum peut être utilisé normalement comme avec les dépôts en ligne.


Mise à jour
-----------

Démonter l'ancien ISO.

<pre>
# <strong>umount /mnt/iso/</strong> 
</pre>

Monter le nouvel ISO.

<pre>
# <strong>mount -o loop CentOS-7-x86_64-Everything-1503-01.iso /mnt/iso/</strong> 
mount: /dev/loop0 est protégé en écriture, 
       sera monté en lecture seule
</pre>

Vider le cache.

<pre>
# <strong>yum clean all</strong> 
Loaded plugins: fastestmirror
Cleaning repos: LocalRepo
Cleaning up everything
Cleaning up list of fastest mirrors
</pre>

Afficher les mises à jour.

<pre>
# <strong>yum check-update</strong> 
</pre>

Une bonne pratique consiste à s'assurer qu'on dispose bien de tous les paquets
de l'installation minimale, étant donné qu'il peut y avoir quelques menus
changements entre les différentes versions.

<pre>
# <strong>yum group install "Minimal Install"</strong> 
</pre>

Il ne reste plus qu'à lancer la mise à jour.

<pre>
# <strong>yum update</strong> 
</pre>


