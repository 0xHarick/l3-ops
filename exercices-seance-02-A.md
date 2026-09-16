## Exercice 1 — Durcir un serveur Linux

**Objectif.** Partir d'une machine nue, joignable par mot de passe, et la rendre
conforme à l'exigence D-06 : accès par clés uniquement, aucun travail en root,
trois ports ouverts et correctifs de sécurité appliqués seuls.

**Prérequis.** VirtualBox installé sur votre poste et l'image ISO d'**Ubuntu
Server 24.04 LTS** (l'édition *Server*, pas *Desktop* : pas d'interface graphique,
c'est exactement ce qu'est un vrai serveur). Un membre de l'équipe pilote, les
autres suivent à l'écran — c'est une manœuvre où l'on se coupe l'accès en silence.

> **Pourquoi une VM et pas un vrai serveur ?** Parce qu'on va délibérément vous
> faire frôler l'erreur qui détruit un serveur, et qu'ici elle ne coûte rien. Les
> gestes, eux, sont **exactement** ceux que vous appliquerez au vrai serveur du
> projet : la procédure que vous écrivez aujourd'hui est celle que vous rejouerez
> le jour où D-06 portera sur une machine en ligne.

### Étape 0 — Créer la machine

Créez une VM Linux / Ubuntu 64 bits, **2 Go de mémoire**, **2 processeurs**,
**15 Go de disque**, et installez Ubuntu Server depuis l'ISO. Pendant
l'installation, cochez **« Install OpenSSH server »** : c'est votre seule porte
d'entrée, et sans écran ni clavier, une machine sans SSH ne sert à rien.

Configurez **deux cartes réseau** dans les paramètres de la VM :

| Carte | Mode | À quoi elle sert |
|---|---|---|
| Adaptateur 1 | **NAT** | donne à la VM un accès à Internet — indispensable pour `apt` |
| Adaptateur 2 | **Réseau privé hôte** | rend la VM **joignable depuis votre poste**, en `192.168.56.x` |

Les deux sont nécessaires, et pour des raisons opposées : le NAT laisse sortir mais
n'a aucune adresse par laquelle entrer ; le réseau privé hôte donne cette adresse.
Relevez-la dans la VM avec `ip a`, et notez-la : **c'est l'adresse de votre serveur**
pour tout le reste de l'exercice.

Dans ce montage, **votre poste joue le rôle de l'extérieur** : c'est de lui que vous
vous connecterez, et c'est de lui que vous vérifierez à l'étape 6.

> *Variante, si le réseau de la salle l'autorise :* mettez l'adaptateur 2 en
> **« Accès par pont »**. La VM reçoit alors une adresse sur le réseau local, et un
> coéquipier peut la scanner depuis sa propre machine — c'est encore plus proche de
> la réalité.

### Étape 0 bis — L'instantané, votre filet

**Avant de toucher à quoi que ce soit**, prenez un instantané (*snapshot*) de la VM
et nommez-le `avant-durcissement`. Si vous vous coupez l'accès à l'étape 3 ou 4,
vous restaurez l'instantané au lieu de tout réinstaller.

> ⚠️ **Le filet ne dispense pas du geste.** À partir de l'étape 3, vous modifiez la
> porte d'entrée de la machine : **gardez en permanence une session SSH ouverte**, et
> testez chaque changement depuis un **second terminal**. Prenez cette habitude
> maintenant, pendant qu'une erreur ne coûte rien — sur le serveur du projet, il n'y
> aura pas d'instantané à restaurer.

Reprenez un instantané après chaque étape réussie. Sinon, restaurer pour réparer
l'étape 4 vous ferait aussi perdre les clés déposées à l'étape 1.

### Les étapes

1. **Une paire de clés par personne.** Chaque membre de l'équipe crée la sienne sur
   son propre poste (`ssh-keygen -t ed25519`), puis dépose sa clé **publique** sur
   la VM. À la fin, `~/.ssh/authorized_keys` contient autant de lignes que l'équipe
   compte de membres. Aucune clé privée ne circule, ni par message, ni par clé USB.

2. **Le compte d'exploitation.** Installez Docker dans la VM
   (`sudo apt install docker.io`), puis créez l'utilisateur `deploy`, membre des
   groupes `sudo` et `docker`. C'est avec lui que l'équipe se connectera désormais,
   et c'est lui que le pipeline utilisera au module 6. Vérifiez que `sudo`
   fonctionne et que `docker ps` répond sans `sudo`.

3. **Fermer la porte aux mots de passe.** Dans `/etc/ssh/sshd_config` :
   `PasswordAuthentication no` et `PermitRootLogin no`, puis rechargez le service.
   Appliquez le protocole de l'encadré ci-dessus.

4. **Le pare-feu en liste blanche.** UFW : tout ce qui entre est refusé par défaut,
   puis 22, 80 et 443 sont autorisés — **dans cet ordre, et `allow 22` avant
   `enable`**. Terminez par `ufw status verbose`.

5. **Les correctifs automatiques.** Installez et activez `unattended-upgrades`.

6. **Vérifiez depuis l'extérieur.** Depuis **votre poste**, jamais depuis la VM :
   scannez les ports ouverts de `192.168.56.x`, puis tentez volontairement une
   connexion par mot de passe
   (`ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no deploy@192.168.56.x`).
   Elle **doit** échouer.

### Ce que vous rendez

Un fichier `docs/serveur.md` dans le dépôt, apporté par une pull request, contenant :

- la configuration réseau de la VM — les deux adaptateurs et l'adresse obtenue ;
- le nom de vos instantanés et le moment où chacun a été pris ;
- la sortie de `ufw status verbose` ;
- la sortie de votre scan de ports **depuis le poste hôte** ;
- la ligne d'erreur renvoyée par la tentative de connexion par mot de passe ;
- **le nombre de clés** présentes dans `authorized_keys`, et à qui elles appartiennent.

Écrivez-le comme une **procédure**, pas comme un compte rendu : c'est ce document
que vous suivrez pour durcir le vrai serveur du projet, et il deviendra plus tard
une section de votre runbook (D-11).

### Ce qui sera regardé

Que la vérification vienne **de l'extérieur de la machine** : un serveur qui se
déclare sûr à lui-même ne prouve rien. Et qu'aucune capture ne montre un `#` de
prompt root.

> **Pièges connus.** `ufw enable` avant `ufw allow 22/tcp` — ici vous restaurez
> l'instantané, sur un vrai serveur vous auriez perdu la machine. Oublier
> l'adaptateur « réseau privé hôte » : la VM sort sur Internet mais reste
> invisible depuis votre poste, et aucun `ssh` n'aboutira. Installer l'édition
> *Desktop* d'Ubuntu : lourde, lente, et sans rapport avec un serveur. Déposer la
> clé publique dans le compte `root` au lieu de `deploy`. Enfin, croire que
> `ufw status` dit la vérité sur un port publié par Docker : il ne la dit pas,
> voir l'exercice 2 et la section 4 du support.