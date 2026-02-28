# Rapport Lab 2 — Hands-On Kafka (CLI)

## 1. Commandes utilisées et observations

---

### Task 1 — Démarrage de Kafka

**Commandes :**
```bash
docker-compose up -d
docker-compose ps
```

**Observations :**
- `docker-compose up -d` démarre deux conteneurs : `zookeeper` et `kafka`.
- Zookeeper démarre en premier car Kafka en dépend (`depends_on`).
- `docker-compose ps` confirme que les deux services ont le statut `Up`.
- Le broker Kafka écoute sur le port `9092`.

---

### Task 2 — Topics

**Commandes :**
```bash
kafka-topics --create --topic transactions --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
kafka-topics --describe --topic transactions --bootstrap-server localhost:9092
kafka-topics --list --bootstrap-server localhost:9092
```

**Observations :**
- La commande `--describe` révèle la structure du topic `transactions` :
  - **3 partitions** (0, 1, 2), chacune assignée au broker 1.
  - `Leader: 1`, `Replicas: 1`, `Isr: 1` — il n'y a qu'un seul broker, donc pas de réplication réelle.
- Kafka crée aussi un topic interne `__consumer_offsets` pour suivre les offsets des groupes.

---

### Task 3 — Producers

**Commandes :**
```bash
kafka-console-producer --bootstrap-server localhost:9092 --topic transactions
# Messages envoyés :
# 1,103,171.07,2024-01-01 10:00:20
# 2,104,337.10,2024-01-01 10:00:26
# 3,105,115.75,2024-01-01 10:00:34
```

**Observations :**
- Chaque ligne saisie constitue un message Kafka distinct.
- Les messages sont distribués entre les 3 partitions par round-robin (pas de clé définie).
- Le producer attend l'entrée de l'utilisateur en continu jusqu'à `Ctrl+C`.
- Pour charger le fichier CSV entier :
  ```bash
  docker-compose exec -T kafka kafka-console-producer --bootstrap-server localhost:9092 --topic transactions < transactions.csv
  ```
  → Les 200 lignes du fichier sont envoyées immédiatement en tant que messages.

---

### Task 4 — Consumers

**Commandes :**
```bash
# Lecture depuis le début
kafka-console-consumer --bootstrap-server localhost:9092 --topic transactions --from-beginning

# Lecture des nouveaux messages uniquement
kafka-console-consumer --bootstrap-server localhost:9092 --topic transactions
```

**Observations :**
- Avec `--from-beginning` : tous les messages déjà publiés sont relus depuis l'offset 0.
- Sans `--from-beginning` : le consumer attend uniquement les nouveaux messages (offset actuel).
- À chaque redémarrage **sans group-id**, Kafka assigne un group-id temporaire aléatoire → le consumer repart toujours de la position par défaut (fin du topic).
- Les messages des 3 partitions sont lus dans un ordre non garanti entre partitions (mais ordonné **au sein de chaque partition**).

---

### Task 5 — Offsets

**Commandes :**
```bash
# Démarrer un consumer avec un group-id explicite
kafka-console-consumer --bootstrap-server localhost:9092 --topic transactions --group groupe-offsets --from-beginning

# Vérifier l'état des offsets du groupe
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group groupe-offsets
```

**Observations :**
- La commande `--describe` affiche pour chaque partition :
  - `CURRENT-OFFSET` : dernier offset consommé par le groupe.
  - `LOG-END-OFFSET` : dernier offset écrit dans la partition.
  - `LAG` : nombre de messages non encore consommés (`LOG-END-OFFSET - CURRENT-OFFSET`).
- Après consommation complète, `LAG = 0` pour toutes les partitions.
- Si le consumer est redémarré **avec le même group-id**, il reprend là où il s'était arrêté (offsets sauvegardés dans `__consumer_offsets`).
- Si le consumer est redémarré **avec `--from-beginning` ET le même group-id**, les offsets déjà commités sont ignorés → les messages sont rejouables uniquement si on réinitialise les offsets :
  ```bash
  kafka-consumer-groups --bootstrap-server localhost:9092 --group groupe-offsets --topic transactions --reset-offsets --to-earliest --execute
  ```

---

### Task 6 — Consumer Groups

**Configuration (3 terminaux) :**
- Terminal 1 et 2 : consumers du même groupe `groupe-test` sur le topic `transactions` (3 partitions).
- Terminal 3 : producer envoyant des messages.

**Commandes :**
```bash
# Terminals 1 et 2
kafka-console-consumer --bootstrap-server localhost:9092 --topic transactions --group groupe-test

# Terminal 3
kafka-console-producer --bootstrap-server localhost:9092 --topic transactions
```

**Observations :**
- Avec 2 consumers et 3 partitions, Kafka distribue les partitions :
  - Consumer 1 → partitions 0 et 1 (ou 0 et 2)
  - Consumer 2 → partition 2 (ou 1)
- Chaque message n'est reçu que par **un seul consumer** du groupe.
- La distribution est équitable mais pas forcément égale (dépend du partitionnement).
- Si un 3ème consumer rejoint le groupe → chaque consumer gère exactement 1 partition.
- Si un 4ème consumer rejoint → l'un d'eux reste **inactif** (plus de consommateurs que de partitions).

---

## 2. Réponses aux questions de réflexion

### Pourquoi les topics sont-ils considérés comme des flux logiques ?

Un topic est une catégorie nommée dans laquelle les messages sont publiés. Il représente un flux de données continu et ordonné (par partition), indépendant du nombre de producers ou consumers qui y accèdent. Le topic abstrait les détails physiques (nombre de brokers, partitions) pour ne présenter qu'un canal logique de communication.

---

### Pourquoi Kafka utilise-t-il des partitions ?

Les partitions permettent le **parallélisme** :
- Un topic peut être distribué sur plusieurs brokers (scalabilité horizontale).
- Plusieurs consumers d'un même groupe peuvent lire en parallèle, chacun sur une partition différente.
- L'ordre des messages est garanti **au sein d'une partition**, mais pas entre partitions.
- Les partitions permettent aussi la **réplication** pour la tolérance aux pannes.

---

### Quel est le rôle des offsets ?

L'offset est un identifiant entier séquentiel assigné à chaque message dans une partition. Il sert à :
- **Suivre la position** d'un consumer dans le flux (ce qu'il a déjà lu).
- **Permettre le replay** : un consumer peut revenir à un offset précédent pour relire des messages.
- **Assurer la résilience** : si un consumer plante, il peut reprendre depuis son dernier offset commité, sans perte ni double traitement (exactement-une-fois avec les bons paramètres).

---

### Pourquoi chaque partition ne permet-elle qu'un seul consumer par groupe ?

Parce que l'ordre des messages dans une partition doit être préservé. Si deux consumers du même groupe lisaient la même partition simultanément, l'ordre de traitement ne serait plus garanti et la gestion des offsets deviendrait incohérente. La règle **une partition = un consumer par groupe** simplifie la coordination et garantit la cohérence.

---

### Que se passe-t-il s'il y a plus de consumers que de partitions ?

Les consumers excédentaires restent **inactifs** (idle). Ils sont connectés au groupe mais aucune partition ne leur est assignée. Ils servent de **standby** : si un consumer actif tombe en panne, Kafka déclenche un **rebalance** et réassigne sa partition à l'un des consumers en attente, assurant ainsi la continuité du traitement.
