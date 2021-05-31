# cassandra_admin
Notes concernant la préparation de la certification Apache Cassandra Admin

CONFIGURING CLUSTERS: YAML
cassandra.yaml est le fichier de configuration principal. Emplacement par défaut /etc/dse/cassandra/. (package installation)
Il est nécessaire de redémarrer le node après chaque modification pour prise en compte de la nouvelle configuration.

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

Dynamic snitch
Dynamic snitch surveille les latences en lecture pour éviter de lire à partir de hists qui sont "lents".
Pour le configurer voir ce lien :
https://cassandra.apache.org/doc/latest/operating/snitch.html#dynamic-snitching

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
  - cleanup => déclenche immédiatement un cleanup des keyspaces qui n'appartiennet plus à un node.
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
  ADDING
bootstrapping est le process pour ajouter un node dans le cluster:
- le node en cours de rejoindre le cluster contact un SEED node
- le seed node communique l'info dans le cluster, incluant les token range vers le nouveau node

Lancer la commande nodetool cleanup après un bootstrap sur les autres nodes
  
  REMOVING
  
  REPLACING

COMPACTION
  
  LEVELED COMPACTION
  
  SIZE TIERED COMAPCTION
  
  TIME WINDOW COMPACTION
  
  Adapter pour les time series data model
  
  REPAIR
  repair => synchronisation des répliques.
  Repair s'assure que que toutes les répliques ont le même nombre de de partitions
  Repair se produit:
  - si nécessaire en lecture (via un quorum)
  - aléatoire via la la proproiété de table read_repair_chance_or_dcloal_read_repair_chance
  - manuellement via la commande nodetool repair

  NODESYNC
  Node sync se comporte comme un repair continue en arrrière plan (disponivle seulement sur DSE)
  
  SSTABLESPLIT
  A utiliser pour splitter 1 large sstables en plusieurs sstables (par exemple suite à compaction qui a produit de trop larges sstables)
  Pour réalsier cette tâche, i lfauut stopper le daemon cassandra
  
  MULTI DATACENTER CONCEPTS
  
  CQL COPY
  Pour importer et exporter des données délimités depuis et vers Apache Cassandra
  
  SSTABLEDUMP
  permet de visualiser la donnée brute d'une SSTable en format TEXTE (foramt JSON)
  
  SSTABLE LOADER
  fournit la possibilité" de :
  - bulk load donnée externe dans un cluster
  - charger des pre existing SSTABLE
  
  Prérequis:
  Le cassandra.yaml doit etre dans le classpath
  - au moins un node dans le cluster est déclaré en tant que SEED
  - configuration obligatoire de ces paramètres:
    - cluster_name
    - listen_address
    - storage_port
    - rpc_address
    - rpc_port
  
  SPARK FOR DATA LOADING
  A tester
  
  DSE DSBULK
  Disponible uniquement sur DSE
  Déplace des données (cassandra) depuis / vers file system
  - utilise 2 formats CSV ou JSON
  Exemple: dsbulk load -url file1.csv -k ks1 -t table1
  
  BACKUP FUNDAMENTALS
  en utilisant snapshot
  
  
  BACKUP DETAILS
  
  
  JVM Settings
  Fichier jvm.options
    - MAX_HEAP_SIZE (Maximum de 8GB)
  - HEAP_NEWSIZE => configurer 100MB par coeur
  
  Heap Dump
  - utilise pour investiguer sur des porblèmes de forte utilisation mémoire ou OutOfMemoryErrors
  - montre exactement quels objects sont le plus consommés dans la heap
  - eclipse fournit un tool pour anamlyser les Heap Dumps (Eclipse Memory Analyzer Tool)
  
TUNING THE KERNEL
  - activer ntp (important car apache cassandra utilsie les timestamp pour retourner la valeur la plus récente)
  - User ressource limits (limits.conf) => tout désactiver (*-nofile, *-memelock ....)
  - swap a désactiver / le retirer du fstab / modifier le paramètre swappiness
  Remarque: il est préfèrable qu'un node crah au lieu de swapper
  - network kernel settings (/etc/sysctl.conf) (voir recommendation)
  
 HARDWARE SELECTION
  - Persistent storage type : éviter SAN storage / NAS et NFS
    Il est FORTEMENT recommandé d'utiliser du SSD (pas de HDD)
  - Memory
    - production: 16 à 64GB (minimum 8GB)
    - dev: pas moins de 4GB
  - CPU
    - production: 16 core CPU
    - dev: 2 à 4 core CPU 
  - Number of nodes
  - network
   - bind les interfaces OS sur différentes NIC
   - bandwidth recommandée : 1GB/s
  
  CLOUD
  ...
  
  SECURITY CONSIDERATIONS (contexte DSE de datastack, voir si possible en cassandra open source ? )
  - authentication (dse.yaml) interne / ldap et kerberos
  - authorization : Permissions sont alter / authorize / create / drop / modify / select
  - encryption 
  
  
  ANTI PATTERN
  - queue anti pattern: Étant donné que Cassandra utilise un moteur de stockage structuré par journal, les suppressions ne suppriment pas immédiatement toutes les traces d'une row. Au lieu de cela, Cassandra écrit un marqueur de suppression appelé tombstone qui supprime les anciennes données jusqu'à ce qu'elles puissent être compactées.
Cassandra peut être amené à lire énormément d'anciennes données avant de lire les entrées encore "en vie"
  
  SNITCH
  2 fonctions :
 -  il en apprend suffisamment à Cassandra sur la topologie du réseau pour acheminer efficacement les demandes.
 - il permet à Cassandra de répartir les réplicas autour du cluster pour éviter les échecs corrélés. Pour ce faire, il regroupe les machines en «datacenters» et «racks». Cassandra fera de son mieux pour ne pas avoir plus d'une réplique sur le même «rack» (qui peut ne pas être un emplacement physique).
  
  CLUSTERING KEY
  colonne permettant d'identifier demanière unique les rows dans une partition et de déterminer l'ordre de tri (ASC par défaut)
