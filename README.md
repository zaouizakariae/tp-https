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

en attendant on demmarre l amchine **Client** :

```cmd
./demarreVM.sh client
```

![image](https://github.com/user-attachments/assets/73828400-a25b-4ef6-a534-c7320c32be0e)

maintenant qu on a demarrer la machine client on doit changer l'add ip du eth1 et innstaller un browser afin d acceder aux sites webs:

![image](https://github.com/user-attachments/assets/a90dcdd5-05a5-40c5-a1a0-5407590b8d20)

# Audit en boîte noire

tout d'abord on doit s'assurer que toutes les machines virtuelles (VM) et l'environnement Vagrant sont en cours d'exécution. on va essayer de pinger les sites web (10.16.10.80 à 10.16.10.84).

```
ping 10.16.10.811
```

le ping n'a pas reussi pour les quatre sites au debut . je me suis connecter en utilisant ssh aux serveurs webs 3 et 4 afin de donner des adds ip aux interfaces reseaux.

maintenant on passe a l audit

![image](https://github.com/user-attachments/assets/f6d8f733-3a52-4ee5-9445-2d141df48088)

![image](https://github.com/user-attachments/assets/39db21e4-6af3-450e-83e9-de9bc528dd6e)

openssl s_client -connect 10.16.10.80:443 -prexit -showcerts -state -status -tlsextdebug -verify 10
![image](https://github.com/user-attachments/assets/a6d8487e-43a9-49ca-9556-756186ba238d)

![image](https://github.com/user-attachments/assets/c8a2bac6-dd4c-4781-b122-dd391e2fea79)

![image](https://github.com/user-attachments/assets/54b3023f-39ea-46be-a461-8978fbdd6357)

![image](https://github.com/user-attachments/assets/d2708b2e-4252-4064-92cb-855dea83144d)

![image](https://github.com/user-attachments/assets/33ff74d7-f62b-4b41-9435-6a082ed342ac)

![image](https://github.com/user-attachments/assets/a5b36c3f-bf37-4fd7-9531-7ffe26b8b8f1)

![image](https://github.com/user-attachments/assets/1acd43e7-b61e-4760-94aa-e64ff4010eef)

![image](https://github.com/user-attachments/assets/04a1ae51-d6df-4b23-b06c-14c55ccfa221)

![image](https://github.com/user-attachments/assets/1507b008-0650-462a-97e2-eb4698d0a2ef)





















