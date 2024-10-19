# tp-https
# Environnement de TP

installation de vagrant deja faite sur la machine de l'universite

# Documentation

on utilise le box fournit par notre professeur en utilisant la commande suivante :

```CMD
  vagrant box add /chemin/vers/fsi-tp.box --name "fsi-tp"
```  

# Audit de Site Web Sécurité

on commence par telecharger l archive

```cmd
wget https://pageperso.lis-lab.fr/emmanuel.godard/enseignement/securite-des-infrastructures/04_https/https.zip
```

![image](https://github.com/user-attachments/assets/56059b18-f31e-4138-97c4-84316c4c51a2)

on demarre l'ensemble des machine en utilisant la commande suivante :

```cmd
./startAll.sh
```

en attendant on demmarre la machine **Client** :

```cmd
./demarreVM.sh client
```

![image](https://github.com/user-attachments/assets/73828400-a25b-4ef6-a534-c7320c32be0e)

maintenant qu on a demarrer la machine client on doit changer l'add ip du eth1 et innstaller un browser afin d acceder aux sites webs:

![image](https://github.com/user-attachments/assets/a90dcdd5-05a5-40c5-a1a0-5407590b8d20)

![image](https://github.com/user-attachments/assets/f6d8f733-3a52-4ee5-9445-2d141df48088)

# Audit en boîte noire

tout d'abord on doit s'assurer que toutes les machines virtuelles (VM) et l'environnement Vagrant sont en cours d'exécution. on va essayer de pinger les sites web (10.16.10.80 à 10.16.10.84).

```
ping 10.16.10.811
```

le ping n'a pas reussi pour les quatre sites au debut . je me suis connecter en utilisant ssh aux serveurs webs 3 et 4 afin de donner des adds ip aux interfaces reseaux.

maintenant on passe a l audit :

![image](https://github.com/user-attachments/assets/39db21e4-6af3-450e-83e9-de9bc528dd6e)

# Audit avec Wireshark

Wireshark permet de capturer le trafic réseau pour analyser la poignée de main SSL/TLS et les communications.

- Ouvre Wireshark sur la VM cliente.
- On Sélectionne l'interface réseau utilisée pour communiquer avec les sites web eth1.
- on accede au site web
- On laisse Wireshark capturer le trafic, puis on l'arrête.
- puis on analyse le trafic

pour web 0 : **Aucune activité inhabituelle :** Le trafic capturé semble montrer une communication HTTP typique, incluant des poignées de main TCP et des requêtes GET.
**Comportement standard :** Le site web sert du contenu HTML basique, compressé avec gzip, ce qui est normal pour minimiser la bande passante.
le site n' utilise pas https

![image](https://github.com/user-attachments/assets/1acd43e7-b61e-4760-94aa-e64ff4010eef)

pour web1 et 2 La communication entre 10.16.10.82 10.16.10.100 et 10.16.10.100 montre un trafic HTTPS typique, initié par un protocole ARP, suivi d'une poignée de main TCP et de l'établissement d'une session TLS.

pour web 3 et 4 Dans l'ensemble, la communication suit un protocole HTTPS standard avec un chiffrement moderne et des versions TLS sécurisées, démontrant un canal chiffré bien établi.

# Audit avec OpenSSL s_client

![image](https://github.com/user-attachments/assets/a6d8487e-43a9-49ca-9556-756186ba238d)

OpenSSL permet d'inspecter les certificats et les détails de la poignée de main.

- On exécute cette commande pour chaque site (on remplace HOSTNAME par les adresses IP de 10.16.10.80 à 10.16.10.84) :

```cmd
openssl s_client -connect 10.16.10.81:443 -prexit -showcerts -state -status -tlsextdebug -verify 10
```

les points a verifier:

- La chaîne de certificats (validité, autorités de certification utilisées).
- La suite de chiffrement négociée.
- La version SSL/TLS utilisée.
- Si l’OCSP stapling est pris en charge (vérifie la sortie -status).

pour web1 : 

- Certificat expiré et autosigne : Le certificat doit être renouvelé ou mis à jour, car il a expiré en 2019.
- OCSP Stapling : Il est recommandé de configurer l'OCSP stapling pour une meilleure vérification de l'état du certificat.
- Renégociation sécurisée : Activer la renégociation sécurisée peut renforcer la sécurité de la connexion TLS.

pour web2 : 

- Certificat expiré et autosigne : Le certificat doit être renouvelé ou mis à jour, car il a expiré en 2021.
- OCSP Stapling : Il est recommandé de configurer l'OCSP stapling pour une meilleure vérification de l'état du certificat.
- ALPN support : l implementation de ALPN (Application-Layer Protocol Negotiation) peut ameliorer la performance et la compatibilite avec les clients car il support les protocoles comme HTTP/2.

Détails du Certificat :

- Sujet : CN=web3.nuage, O=M2 FSI, OU=TP SIR, L=Marseille, ST=PACA, C=FR
- Émetteur : CN=godard TP, O=Master FSI, OU=TP SIR, ST=paca, C=FR
- Le certificat est auto-signé par godard TP.
- Erreurs de Vérification :

  - num=20 : Impossible d'obtenir le certificat de l'autorité locale, ce qui signifie que le système ne peut pas vérifier la chaîne de confiance.
  - num=21 : Impossible de vérifier le premier certificat, ce qui confirme que le certificat n'est pas de confiance (car il est auto-signé).
  - Le certificat a expiré : Le certificat a expiré le 12 octobre 2021.

pour web3 :

- Certificat Expiré : Le certificat a expiré le 12 octobre 2021. Les navigateurs modernes afficheront des avertissements de sécurité, réduisant la confiance des utilisateurs. Vulnérabilité potentielle aux attaques de type "homme du milieu" (MITM).

![image](https://github.com/user-attachments/assets/d2708b2e-4252-4064-92cb-855dea83144d)

- Certificat Auto-Signé : Le certificat est auto-signé par "godard TP" au lieu d'être émis par une autorité de certification reconnue. Manque de confiance dans l'identité du serveur, vulnérabilité aux attaques MITM.
- Chaîne de Confiance Incomplète : Impossible d'obtenir le certificat de l'autorité locale (erreur num=20). Les clients ne peuvent pas vérifier l'authenticité du certificat, ce qui peut conduire à des avertissements de sécurité.
- Configuration du Serveur : Le serveur utilise TLS 1.3 avec un chiffrement fort (TLS_AES_256_GCM_SHA384), ce qui est positif.

pour web4 :

- Version du protocole : Le serveur utilise TLS 1.3, qui est la version la plus récente et la plus sécurisée du protocole TLS. C'est un point positif.
- Certificat du serveur : Le certificat est émis pour "web4.nuage" avec plusieurs informations d'organisation.
Il est signé par une autorité de certification (CA) apparemment locale ou privée (Emmanuel Godard).
Le certificat est valide du 8 octobre 2024 au 8 octobre 2025 (validité d'un an, ce qui est une bonne pratique).
- Problèmes de vérification du certificat : L'erreur "unable to verify the first certificate" est signalée. Cela indique que la chaîne de confiance n'est pas complète ou que le certificat racine n'est pas reconnu.
- Pas de négociation ALPN.

# Audit avec testssl.sh

testssl.sh est un outil passif de vérification de la configuration SSL/T

-on installe testssl.sh

```cmd
git clone https://github.com/drwetter/testssl.sh.git
cd testssl.sh
```

- on exécute testssl.sh sur chaque site 

```cmd
./testssl.sh ip_add
```

![image](https://github.com/user-attachments/assets/c8a2bac6-dd4c-4781-b122-dd391e2fea79)

# Audit en boîte blanche




![image](https://github.com/user-attachments/assets/54b3023f-39ea-46be-a461-8978fbdd6357)


on analyse les fichiers service.conf

```cmd
<VirtualHost *:80>
	ServerAlias web0.nuage
	ServerAdmin webmaster@localhost
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
----------------------------------------------------------------------------------------
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
</VirtualHost>


<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile      /vagrant/cert.pem
    SSLCertificateKeyFile   /vagrant/key.pem

    # HTTP Strict Transport Security (mod_headers is required) (63072000 seconds)
    Header always set Strict-Transport-Security "max-age=63072000"
</VirtualHost>

SSLProtocol             all -SSLv3
----------------------------------------------------------------------------------------
<VirtualHost *:80>
    ServerAlias web2.nuage
    ServerAdmin webmaster@localhost

    RewriteEngine On
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>


<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile      /vagrant/cert.pem
    SSLCertificateKeyFile   /vagrant/key.pem

    # HTTP Strict Transport Security (mod_headers is required) (63072000 seconds)
    Header always set Strict-Transport-Security "max-age=63072000"
</VirtualHost>

SSLProtocol             all -SSLv3 -TLSv1 
SSLCipherSuite 	        HIGH:!aNULL:!MD5
---------------------------------------------------------------------------------------
<VirtualHost *:80>
	ServerAlias web3.nuage

	ServerAdmin webmaster@localhost

</VirtualHost>


<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile      /vagrant/cert.pem
    SSLCertificateKeyFile   /vagrant/key.pem

</VirtualHost>

SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite          HIGH:!aNULL:!MD5

SSLHonorCipherOrder     off
--------------------------------------------------------------------------------------
<VirtualHost *:80>
	ServerAlias web4.nuage

	ServerAdmin webmaster@localhost
        RewriteEngine On
        RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>


<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile      /vagrant/cert.pem
    SSLCertificateKeyFile   /vagrant/key.pem

    # HTTP Strict Transport Security (mod_headers is required) (63072000 seconds)
    Header always set Strict-Transport-Security "max-age=63072000"
</VirtualHost>

SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1 
SSLHonorCipherOrder     off

```



Serveur web0.nuage :

- Problème majeur : Absence totale de HTTPS.
- Vulnérable aux attaques d'interception (man-in-the-middle).
- Aucun chiffrement des données en transit.
- Risque élevé de vol d'informations sensibles.


Serveur web1 :

- Utilise encore SSLv3, qui est obsolète et vulnérable (POODLE attack).
- Pas de redirection automatique de HTTP vers HTTPS, permettant potentiellement des connexions non sécurisées.
- Absence de configuration de cipher suite, pouvant mener à l'utilisation de chiffrements faibles.


Serveur web2.nuage :

- Utilise encore TLSv1, qui est considéré comme obsolète et présente des vulnérabilités connues.
- Bien que la configuration soit meilleure que les précédentes, l'utilisation de TLSv1 reste un risque.


Serveur web3.nuage :

- Absence de HSTS, permettant potentiellement des attaques de type downgrade.
- Pas de redirection automatique de HTTP vers HTTPS, laissant la possibilité de connexions non sécurisées.
- SSLHonorCipherOrder est désactivé, ce qui pourrait permettre au client de choisir un chiffrement plus faible.


Serveur web4.nuage :

- SSLHonorCipherOrder est désactivé, laissant le choix du chiffrement au client, potentiellement moins sécurisé.
- Manque de configuration pour les en-têtes de sécurité supplémentaires comme X-Frame-Options, X-Content-Type-Options, etc.

# Bonnes pratiques 

- Utiliser uniquement HTTPS
- Rediriger tout le trafic HTTP vers HTTPS
- Implémenter HSTS (HTTP Strict Transport Security)
- Utiliser les versions récentes de TLS (TLSv1.2 et TLSv1.3)
- Configurer une suite de chiffrement forte
- Utiliser des certificats SSL/TLS valides et à jour
- Activer OCSP Stapling
- Implémenter la politique de sécurité du contenu (CSP)
- Utiliser les en-têtes de sécurité HTTP appropriés
- Désactiver les versions anciennes et non sécurisées de SSL/TLS
- Configurer le Perfect Forward Secrecy (PFS)


```cmd
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile      /etc/apache2/ssl/cert.pem
    SSLCertificateKeyFile   /etc/apache2/ssl/key.pem

    # Secure SSL Protocols
    SSLProtocol -all +TLSv1.2 +TLSv1.3

    # Secure Cipher Suite
    SSLCipherSuite HIGH:!aNULL:!MD5:!3DES:!RC4:!DHE
    SSLHonorCipherOrder on  # Enforce cipher order

    # Enable OCSP Stapling for certificate validation
    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/stapling_cache(128000)"

    # HTTP Strict Transport Security (HSTS)
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"

    # Additional Security Headers
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "DENY"
    Header always set X-XSS-Protection "1; mode=block"

    # Content Security Policy (CSP)
    Header always set Content-Security-Policy "default-src 'self';"

    # Redirection HTTP to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R=301,L]

    DocumentRoot /var/www/html
    ServerName web_secure.nuage
</VirtualHost>

<VirtualHost *:80>
   
	ServerAlias demo.nuage

	ServerAdmin webmaster@localhost
        RewriteEngine On
        RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]

</VirtualHost>
```


![image](https://github.com/user-attachments/assets/33ff74d7-f62b-4b41-9435-6a082ed342ac)

![image](https://github.com/user-attachments/assets/a5b36c3f-bf37-4fd7-9531-7ffe26b8b8f1)

![image](https://github.com/user-attachments/assets/1acd43e7-b61e-4760-94aa-e64ff4010eef)

![image](https://github.com/user-attachments/assets/04a1ae51-d6df-4b23-b06c-14c55ccfa221)

![image](https://github.com/user-attachments/assets/1507b008-0650-462a-97e2-eb4698d0a2ef)





















