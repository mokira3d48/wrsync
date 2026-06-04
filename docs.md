En suivant les bonnes pratiques et les recommandations du shell scripting de qualite production, ecris moi un script bash qui
  utilise rsync sous le capo pour envoyer des fichiers et dossiers sur un server ssh. Mais ce scripte doit avoir l'interface 
  de cli suivante : 
```bash
wrsync ./mon_dossier_ou_fichier monvps:~/mondossierdeprojet
```
  il copie mon_dossier_ou_fichier et son contenu (s'il s'agit d'un dossier `-r` avant `mon_dossier`) de mon repertoire courant vers `monvps` qui represente le nom du fichier de config que j'ai
  cree dans ~/.config/wrsync/monvps.conf et qui comporte toute les configurations : username, IP de l'ordinateur distant, le
  port de connexion au serveur SSH et autres configuration necessaires pour faire fonctionner `rsync` sous le capo en toute 
  securite.
 
Dans l'exemple ci-dessus, le fichier mon_dossier_ou_fichier sera copie dans le repertoire `mondossierdeprojet` qui se trouve dans le `/home/username/` de la machine distant.

Dans l'exemple suivant, `mon_dossier_ou_fichier` sera copie dans le dossier systeme `opt` de la machine distant.

```bash
wrsync ./mon_dossier_ou_fichier monvps:/opt/mondossierdeprojet
  ```
