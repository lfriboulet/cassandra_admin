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
