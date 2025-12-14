#  Lab 2 — Hands-On Kafka (CLI)

In this lab, you will interact directly with **Apache Kafka** using its **command-line tools**.
The goal is to understand Kafka’s **core concepts** by experimenting and exploring the CLI.

You are expected to search the documentation and discover the appropriate commands by yourself.

---

## Learning Objectives

By the end of this lab, you should understand and be able to explain:

- What a **topic** is and why it is a logical stream
- How **partitions** enable parallelism
- What a **producer** does
- What a **consumer** does
- How **offsets** track message positions
- What a **consumer group** is and how it shares workload
- What a **broker** is

---

##  Kafka Installation

Kafka must be run using **Docker**.

### Prerequisites
- Docker Desktop installed and running
- Basic familiarity with the terminal

You should be able to:
- Start Kafka using Docker
- Verify that Kafka and its dependencies are running

---

## Lab Tasks

### Task 1 — Start Kafka
- Start Kafka using Docker
- Verify that the Kafka broker is running

---

### Task 2 — Topics
- Create a topic named `transactions`
- Inspect the topic configuration
- Identify how many partitions the topic has

---

### Task 3 — Producers
- Start a Kafka producer
- Send multiple messages to the `transactions` topic
- Observe how messages are written to Kafka

---

### Task 4 — Consumers
- Start a Kafka consumer
- Read messages from the `transactions` topic
- Restart the consumer and observe what happens

---

### Task 5 — Offsets
- Identify how Kafka tracks which messages have been consumed
- Observe the effect of restarting consumers with and without replaying old messages

---

### Task 6 — Consumer Groups
- Start multiple consumers using the same consumer group
- Produce new messages
- Observe how the messages are distributed

---

## Reflection Questions (Answer in Your Own Words)

- Why are topics considered logical streams?
- Why does Kafka use partitions?
- What is the role of offsets?
- Why does each partition only allow one consumer per group?
- What happens if there are more consumers than partitions?

---

##  Notes
- Do **not** write application code for this lab
- Use only Kafka command-line tools
- Take notes of commands you discover and what they do

---

## Deliverable
Submit a short document explaining:
- What commands you used
- What you observed
- Answers to the reflection questions
