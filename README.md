# Serveur HKMP avec Docker Compose / Portainer

Prérequis : Docker et Docker Compose ou Portainer sur une machine Linux `amd64` (architecture publiée pour l'image utilisée). Utiliser un environnement **Docker Standalone** dans Portainer : cette stack utilise `depends_on` avec `service_completed_successfully`.

## Déployer avec Portainer depuis Git

1. Publier ce dépôt sur un hébergeur Git accessible depuis Portainer (GitHub, GitLab, etc.). Le dépôt ne contient pas les données du serveur.
2. Dans Portainer : **Stacks → Add stack → Git repository**. Indiquer l'URL du dépôt, la branche `main` et le chemin du fichier Compose `compose.yaml` (à la racine). Définir éventuellement la variable `HKMP_PORT` (par défaut `2222`), puis cliquer sur **Deploy the stack**.
3. Vérifier les logs du service `hkmp` : `Initializing server addon: HkmpPouch 2.0.0` et `Starting NetServer on port 2222` (ou le port choisi). L'image du serveur est épinglée sur HKMP v2.4.3.

Le service `pouch` télécharge automatiquement la DLL serveur **HkmpPouch v2.0.0r** dans le volume Docker `hkmp_data` si elle n'est pas déjà présente, puis démarre le service `hkmp`. Le premier démarrage nécessite un accès à GitHub. La configuration et les sauvegardes sont persistées dans ce même volume, même après recréation des conteneurs. Ne pas supprimer le volume si tu veux conserver ces données.

## Jouer

Ouvrir/rediriger le port **2222/UDP** vers la machine Docker sur le pare-feu et le routeur si les joueurs se connectent depuis Internet. Communiquer l'adresse IP publique (ou le nom de domaine) et le port 2222 aux joueurs. Sur le même réseau local, utiliser l'IP locale.

Dans Hollow Knight avec **HKMP et HkmpPouch** installés via Lumafly, se connecter à cette adresse et à ce port depuis le menu du mod, ou avec `connect <adresse> 2222 <pseudo>` dans le chat HKMP. Vérifier que la version du mod HKMP est compatible avec le serveur v2.4.3 et que HkmpPouch annonce l'addon réseau v2.0.0.

Les mods **MagicUI, MapChanger, MenuChanger, MultiplayerEvents, Satchel et Vasi** sont installés dans Hollow Knight via Lumafly ; ce ne sont pas des DLL serveur HKMP à copier dans le volume. MultiplayerEvents utilise HkmpPouch pour communiquer entre joueurs. Les autres joueurs qui veulent rejoindre ce serveur doivent aussi installer **HKMP et HkmpPouch** ; MultiplayerEvents reste un mod client optionnel.

## Utilisation locale (Docker Compose)

Depuis ce dossier :

```sh
docker compose up -d
docker compose logs -f hkmp
```

Le serveur crée notamment `/HKMP/serversettings.json` dans le volume. Pour envoyer une commande à la console du serveur :

```sh
docker compose exec hkmp hkmp list
docker compose exec hkmp hkmp 'set pvp true'
```

Pour utiliser un autre port en local, créer un fichier `.env` contenant `HKMP_PORT=3333` **avant** de lancer Compose ; dans Portainer, définir `HKMP_PORT` dans les variables d'environnement de la stack. Ouvrir le même port en UDP et donner ce port aux joueurs. Pour arrêter le serveur local : `docker compose down`.

Si tu utilisais l'ancienne version de ce dossier avec `./data`, les fichiers existants restent dans ce dossier (ignoré par Git). Pour les transférer sur la **même machine** vers le nouveau volume local (avec le nom de projet Compose `hkmp`), depuis ce dossier :

```sh
docker compose down
docker volume create hkmp_hkmp_data
docker run --rm -v "$PWD/data:/from:ro" -v hkmp_hkmp_data:/to alpine:3.18.4 sh -c 'cp -a /from/. /to/'
docker compose up -d
```

Si le serveur doit tourner dans Portainer sur cette même machine, utiliser à la place le nom du volume créé par sa stack (`<nom-de-stack>_hkmp_data`) et ne pas redémarrer l'ancien serveur sur le même port UDP. Sur une autre machine, transférer les fichiers séparément ; ils ne sont pas dans le dépôt Git.

Sources : [serveur HKMP et commandes](https://github.com/Extremelyd1/HKMP#standalone-server), [image Docker et volume de données](https://hub.docker.com/r/maximalmax90/hkmpserver), [fonctionnement de HkmpPouch](https://github.com/PrashantMohta/HkmpPouch/blob/main/ReadMe.md).
