# cassandra_admin
Notes concernant la préparation de la certification Apache Cassandra Admin

CONFIGURING CLUSTERS: YAML
cassandra.yaml est le fichier de configuration principal. Emplacement par défaut /etc/dse/cassandra/. (package installation)
Il est nécessaire de redémarre le node après chaque modification pour prise en compte de la nouvelle configuration.

Minimal settings:
- cluster_name
- listen_address
- native_transport_address (c'est l'adresse IP utilisé par le client pour se connecter au node et / ou au cluster)
- seeds (adresse(s) node(s) qui sont contactés lorsque un nouveau node rejoint le cluster )

Commonly Used settings:
- endpoint_snitch (SimpleSnitch par défaut, il est obligatoire si on veut un cluster pour être topologie aware)
- initial_token (désactivé par défaut) et num_token (128 par défaut) : (si on souhaite utiliser kes virtual nodes "vnodes" pour distribuer la donnée dans le cluster)
- commitlog_directory (défaut /var/lib/cassandra/commitlog)
- data_file_directories (défaut /var/lib/cassandra/data)
- hints_directory (défaut /var/lib/cassandra/hints)
- saved_caches_directory (défaut /var/lib/cassandra/saved_caches) (key and row cache files)

Notable settings:
hinted_handoff_enable (défaut true)
max_hint_window_in_ms (défaut 10800000 [3 hours])
row_cache_size_in_mb (défaut à 0 désactivé) (cache in memory)
file_cache_size_in_mb (défaut 4096 désactivé) (maximum memoru to use when pooling ss table buffers)
memtable_heap_space_in_mb
memtable_offheap_space_in_mb

CLUSTER SIZING (Estimation process)
- débit (throughput): mesurer le débit des données en "mouvement " par période de temps (GB/S)
- taux de croissance (growth rate)
- latence (latency) 

CASSANDRA STRESS
1. cassandra-stress est un tool pour simuler une charge de travail utilisateur
Utiliser pour :
  - déterminer la performance du schéma
  - comprendre comment scaler la base
  - optimiser le modèle de données et paramètres
  - déterminer la capacité de sa production
 A réaliser avant de passer en production

2. configuration (fichier yaml)
description schéma / description colonnes / description batch / description query

3. exemple de commande:
cassandra-stress user profile=/home/ubuntu/labwork/TestProfile.yaml ops\(insert=100,user_by_email=100\) -node ds210-node1

LINUX COMMAND
- top
- dstat = montre toutes les ressources systèmes en un unique tableau
  dstat -am
  option -a donne toutes les stats par défaut
  option -am donne toutes les stats par défaut + la mémoire
  
- nodetool (couteau suisse de Cassandra)
  - info => pour avoir les infos sur un node spécifique
  - compactionhistory => liste et décrit les actions de compaction qui ont eu lieu
  - gcstats (java carbage collection)
  - gossipinfo => pour vérifier si tous les nodes ont le même schéma
  - ring => montre info concernant les tokens dans le ring
  - tablestats => montre info concerant keyspaces / tables flushé sur disque -H pour une sortie human readeable
  - tablehistograms => fournit des stats concernant une spécifique table (utile pour comprendre les latences sur les tables)
  - tpstats (thread pool stats) => donne les infos concernant les thread pools
      - active = le pool est en train de traiter une requête
      - pending = la requête est e nattente dans le poom
      - completed = la requête a été traitée
      - blocked = la file était pleine et donc la requête a été rejectée

SYSTEM & OUTPUT LOGS
- /var/log/cassandra/system.log (INFO messages et plus)
- /var/log/cassandra/debug.log (tous les messages)
Il est possible de modifierl'empalcement des logs en ajoutant -Dcassandra.logir=<specify path here > dans le fichier /etc/dse/cassandra/jvm.options
7 niveau de logging: OFF / ERROR / WARN / INFO / DEBUG / TRACE / ALL
2 manière de configurer le logging:
  - en utilisant le fichier logback.xml
  - en utilisant la commande nodetool setlogginglevel (seulement temporaire jusqu'au prochain redémarrage) 
  
JVM GARBAGE COLLECTION LOGGING
aide pour savoir exactement ce que fait la JVM
Pour trouver:
  - quand GC s'est produit
  - combien de mémoire GC a besoin
  - combien de mémoire est disponible dans la heap
 Activer GC logging:
 - statique en éditant jvm.options
 - dynamique en utilisant jinfo

-Xloggc:<log_file_name_goes_here>

ADDING / REMOVING NODES
bootstrapping est le process pour ajouter un node dans le cluster:
- le node en cours de rejoindre le cluster contact un SEED node
- le seed node communique l'info dans le cluster, incluant les token range vers le nouveau node

Lancer la commande nodetool cleanup après un bootstrap sur les autres nodes
