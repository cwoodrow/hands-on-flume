# hands-on-flume

## netcat

Le fichier ci-dessous *console-logger/console-logger.config* permet de configurer un agent qui écoute sur netcat port 44444 et qui écrit les événements recus dans les logs.

```
console-logger.sources = netcat-source
console-logger.sinks = logger-sink
console-logger.channels = memory-channel

# Describe/configure the source
console-logger.sources.netcat-source.type = netcat
console-logger.sources.netcat-source.bind = localhost
console-logger.sources.netcat-source.port = 44444

# Describe the sink
console-logger.sinks.logger-sink.type = logger

# Use a channel which buffers events in memory
console-logger.channels.memory-channel.type = memory
console-logger.channels.memory-channel.capacity = 1000
console-logger.channels.memory-channel.transactionCapacity = 100

# Bind the source and sink to the channel
console-logger.sources.netcat-source.channels = memory-channel
console-logger.sinks.logger-sink.channel = memory-channel
```

Copier ce fichier dans le cluster **Hadoop**, pour l'exemple dans */usr/hdp/2.3.2.0-2950/flume/conf/*

Pour lancer l'agent, ouvrez une session SSH sur le cluster et exécutez la commande suivante :

```
bin/flume-ng agent -n console-logger -c conf/ -f conf/console-logger.conf -Dflume.root.logger=INFO,console
```

**NB : l'option -Dflume.root.logger=INFO,console permet de logger dans la console.**

Ouvrir une seconde session SSH sur le cluster.

Observez le résultat.

### Ajout d'un interceptor
Ajoutez à présent un interceptor (le [timestamp](https://flume.apache.org/FlumeUserGuide.html#timestamp-interceptor) interceptor est un bon choix).

Ovbservez le résultat.

## Agrégation de logs
Nous allons à présent agréger des logs d'un firewall fictif dans **HDFS**.

Le répertoire *server-logs/* contient deux fichiers :

* Un script python *generate_logs.py* qui permet de générer des logs dans le */var/log/eventlog-demo.log*
* Un fichier de configuration **Flume** qui lit les logs dans */var/log/eventlog-demo.log* et écrit les événements dans **HDFS**

Copiez les fichiers sur le serveur.

Lancez l'agent **Flume**.

Lancez le script de génération des logs :
```
python generate_logs.py
```

Observez le résultat dans **HDFS**.

Créer une table dans **HCatalog** :
```
hcat -e "CREATE TABLE FIREWALL_LOGS(time STRING, ip STRING, country STRING, status STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY '|' LOCATION '/flume/events';"
```

Vous pouvez à présent exécuter des requêtes sur la table **FIREWALL_LOGS**.

### Tutoriel en entier
Ce tutoriel est issu d'un tutoriel de chez Horton, pour faire ce tutoriel en entier :
http://fr.hortonworks.com/hadoop-tutorial/how-to-refine-and-visualize-server-log-data/
