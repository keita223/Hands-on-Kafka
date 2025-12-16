# Commandes Kafka - Lab 2

## Méthode 1 : Commandes directes (depuis l'hôte)

### Task 1 — Démarrage de Kafka

```bash
# Aller dans le dossier du projet
cd Hands-on-Kafka

# Démarrer Kafka et Zookeeper
docker-compose up -d

# Vérifier que les services sont actifs
docker-compose ps
```

### Task 2 — Topics

```bash
# Créer le topic "test-topic"
docker-compose exec kafka kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

# Inspecter la configuration du topic
docker-compose exec kafka kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092

# Lister tous les topics
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092
```

### Task 3 — Producer

```bash
# Démarrer un producteur pour envoyer des messages au topic "test-topic"
docker-compose exec kafka kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic

# Tapez vos messages ligne par ligne puis Ctrl+C pour quitter
```

### Task 4 — Consumer

```bash
# Démarrer un consommateur pour lire les messages depuis le début
docker-compose exec kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --from-beginning

# Lire uniquement les nouveaux messages (sans --from-beginning)
docker-compose exec kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic
```

---

## Méthode 2 : En se connectant au conteneur Kafka (utilisée)

### Étape 1 : Se connecter au conteneur

```bash
# Démarrer Kafka
docker-compose up -d

# Entrer dans le conteneur Kafka
docker-compose exec kafka bash
```

### Étape 2 : Utiliser les commandes Kafka depuis le conteneur

```bash
# Créer un topic
kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

# Lister les topics
kafka-topics --list --bootstrap-server localhost:9092

# Décrire un topic
kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092

# Démarrer un consumer (lire depuis le début)
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --from-beginning

# Démarrer un producer (envoyer des messages)
kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
# Ensuite tapez vos messages ligne par ligne

# Consumer avec consumer group
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --group mon-groupe
```

### Étape 3 : Quitter le conteneur

```bash
exit
```

---

## Task 6 — Consumer Groups (plusieurs terminaux)

### Terminal 1 : Se connecter et démarrer premier consumer

```bash
docker-compose exec kafka bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --group groupe-test
```

### Terminal 2 : Se connecter et démarrer deuxième consumer

```bash
docker-compose exec kafka bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-topic --group groupe-test
```

### Terminal 3 : Se connecter et démarrer producer

```bash
docker-compose exec kafka bash
kafka-console-producer --bootstrap-server localhost:9092 --topic test-topic
# Tapez des messages et observez la distribution entre les consumers
```

## Commandes utiles

```bash
# Voir les logs d'un service
docker-compose logs kafka
docker-compose logs zookeeper

# Arrêter les services
docker-compose down

# Arrêter et supprimer les volumes
docker-compose down -v
```
