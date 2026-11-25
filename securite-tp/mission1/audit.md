# Mission 1 : audit de l'image initiale quiz-docker:initial

| Point vérifié | Résultat | Risque |
|---|---|---|
| Application | Fonctionne sur http://localhost:8092 | Aucun |
| Image de base | `node:18-alpine` | Alpine est léger ; Node 18 n'est plus maintenu |
| Utilisateur | `root` (`whoami` et `docker top`) | L'appli a tous les droits dans le conteneur |
| Taille | 243 Mo sur disque, 60,2 Mo compressée | Grosse image = grande surface d'attaque |
| Couches | Les plus lourdes : `RUN npm install` et `RUN npm run build && npm install -g serve` (détail dans `couches.txt`) | Les outils de construction restent dans l'image finale |
| Vulnérabilités (Docker Scout) | 13 critiques, 92 hautes, corrigeables (détail dans `scout-cves.txt`) | Failles connues exploitables |

Points à corriger dans les missions suivantes : utilisateur non root, image de
base maintenue, image finale plus légère (sans les outils de construction),
mise à jour des paquets vulnérables.