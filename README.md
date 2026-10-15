# docker-hive : Hadoop + Hive + Hue en une commande

Une stack Docker Compose qui monte un **mini-cluster Hadoop complet** (HDFS, YARN, Hive avec metastore PostgreSQL, Hue) sur un simple PC, pour apprendre et faire ses TP Big Data **sans la VM Cloudera QuickStart**.

Elle a servi pendant le bloc projet Big Data de 4e année d'ingénieur informatique au **CESI** (octobre 2025), à la place de la VM Cloudera, trop lourde, plus maintenue et pénible à installer. Si tu es étudiant CESI (ou d'ailleurs) et que tu galères avec Hadoop, ce dépôt est fait pour toi.

> ⚠️ **Environnement de développement et d'apprentissage uniquement.** Les images sont anciennes (Hadoop 3.1.3, Hive 2.3.2), les mots de passe sont ceux par défaut et les permissions HDFS sont désactivées. Ne jamais exposer cette stack sur Internet.

## Ce qu'il y a dedans

| Service | Rôle | Port(s) sur ta machine |
|---|---|---|
| `namenode` | HDFS : annuaire des fichiers, interface web | [9870](http://localhost:9870) (web), 8020 (RPC HDFS) |
| `datanode1`, `datanode2` | HDFS : stockage des blocs (2 nœuds, pour voir la réplication) | 9864 / 50075, 9866 / 50076 |
| `resourcemanager` | YARN : ordonnanceur des jobs | [8088](http://localhost:8088) |
| `nodemanager` | YARN : exécution des jobs | 8042 |
| `historyserver` | Historique des jobs YARN | 8188 |
| `hive-server` | HiveServer2 (JDBC / beeline / Talend) | 10000 |
| `hive-metastore` + `hive-metastore-postgresql` | Catalogue des tables Hive | 9083 |
| `hue` + `hue-postgres` | Interface web pour écrire du SQL Hive et parcourir HDFS | [8888](http://localhost:8888) |

## Démarrage rapide

Prérequis : Docker Desktop (ou Docker Engine + Compose v2) et **8 Go de RAM libres** de préférence.

```bash
git clone https://github.com/calaxo/Docker-hive.git
cd Docker-hive
docker compose up -d
```

Le premier lancement télécharge les images (plusieurs Go). Les services s'attendent entre eux : compte **1 à 2 minutes** avant que tout soit prêt.

Vérifier que tout tourne :

```bash
docker compose ps
```

Puis ouvrir :
- **Hue** : http://localhost:8888 (au premier accès, le compte que tu crées devient administrateur) ;
- **HDFS** : http://localhost:9870 (onglet *Datanodes* : les 2 datanodes doivent être *In service*) ;
- **YARN** : http://localhost:8088.

Arrêter : `docker compose down`. Tout effacer, données comprises : `docker compose down -v`.

## Utilisation

### Requêtes Hive en ligne de commande (beeline)

```bash
docker exec -it hive-server beeline -u jdbc:hive2://localhost:10000
```

```sql
CREATE TABLE test (id INT, nom STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
SHOW TABLES;
```

### Mettre un fichier dans HDFS

```bash
docker cp mon_fichier.csv namenode:/tmp/
docker exec -it namenode hdfs dfs -mkdir -p /data
docker exec -it namenode hdfs dfs -put /tmp/mon_fichier.csv /data/
docker exec -it namenode hdfs dfs -ls /data
```

Puis dans Hive : `LOAD DATA INPATH '/data/mon_fichier.csv' INTO TABLE test;`

### Connecter Talend (Talaxie 8) à Hive et HDFS

C'était le besoin du projet CESI : faire tourner des jobs Talend sur le PC hôte contre le cluster Docker.

1. **Fichier hosts** : HDFS renvoie au client les *noms* des conteneurs (`namenode`, `datanode1`…), que ton PC ne connaît pas. Ajoute le contenu de [`host fichier conf`](host%20fichier%20conf) à ton fichier hosts :
   - Windows : `C:\Windows\System32\drivers\etc\hosts` (éditeur lancé en administrateur) ;
   - Linux / macOS : `/etc/hosts`.
2. **Configuration Hadoop** : [`hadoop-conf-docker.jar`](hadoop-conf-docker.jar) contient les fichiers `core-site.xml`, `hdfs-site.xml`, `yarn-site.xml`… générés depuis le conteneur `namenode`. Dans Talend, crée une connexion Hadoop **personnalisée** (*custom*) et donne-lui ce jar.
3. **Hive** : connexion HiveServer2 sur `hive-server:10000` (ou `localhost:10000`).

[Talaxie](https://deilink.fr/#/download) est le fork communautaire de Talend Open Studio.

## Problèmes fréquents

- **Un conteneur redémarre en boucle** : souvent un manque de RAM. Augmente la mémoire allouée à Docker Desktop (Paramètres → Resources) ou ferme d'autres applications.
- **« port is already allocated »** : un autre logiciel utilise déjà le port (8088 et 8080 sont souvent pris). Change le port de gauche dans `docker-compose.yml`, par exemple `"18088:8088"`.
- **Hive ne répond pas tout de suite** : le metastore attend HDFS et PostgreSQL. Patiente une minute, puis regarde `docker compose logs hive-metastore`.
- **Talend ou un client sur le PC n'arrive pas à écrire dans HDFS** : vérifie le fichier hosts (étape 1 ci-dessus).

## Limites connues

- Images `bde2020/*` de 2019-2020, plus mises à jour. Leur provenance exacte et leur sécurité ne sont pas garanties : à réserver à un usage local.
- `gethue/hue:latest` peut évoluer et casser la compatibilité un jour. Si Hue ne démarre plus, fixe une version dans `docker-compose.yml`.
- Mots de passe par défaut (`hive`/`hive`, `hue`/`hue`, clé secrète Hue d'exemple dans `z-hue-overrides.ini`) : à changer si la stack est accessible depuis un autre poste.

## Origine et crédits

Basé sur [Vivekpawar07/Docker-hive](https://github.com/Vivekpawar07/Docker-hive), lui-même dérivé de [big-data-europe/docker-hive](https://github.com/big-data-europe/docker-hive) et [docker-hadoop](https://github.com/big-data-europe/docker-hadoop).

Ajouts de ce fork : cluster à **2 datanodes**, **Hue** (avec sa base PostgreSQL et sa configuration `z-hue-overrides.ini` reliée à HDFS et Hive), le **jar de configuration** et le **fichier hosts** pour Talend, et le nettoyage des données d'exemple.

Contributeurs du projet d'origine :
* Ivan Ermilov [@earthquakesan](https://github.com/earthquakesan) (maintainer)
* Yiannis Mouchakis [@gmouchakis](https://github.com/gmouchakis)
* Ke Zhu [@shawnzhu](https://github.com/shawnzhu)
