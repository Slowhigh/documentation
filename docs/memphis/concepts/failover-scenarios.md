---
title: Failover Scenarios
description: This section describes how memphis behaves when there is a failure in reading, writing, or when there is potential data loss.
---

# Failover Scenarios

## Introduction

This section describes the exact behavior of Memphis in different failure scenarios based on three parameters:&#x20;

* Ability to read
* Ability to write
* Potential data loss

This section also aims to educate and explain Memphis's cluster mode's internals so its users can make design decisions that suit their workloads perfectly.

## Initial state

The test environment is based on a station with **three replicas** (mirrors) over **three memphis brokers** in cluster mode.

(\*) The small circle within the brokers emphasizes if they are a ["Leader" or "Follower."](station#leaders-and-followers)

<figure><img src="/assets/initial_state.jpeg" alt=""><figcaption></figcaption></figure>

## Scenarios

### 1. Memphis brokers

Single broker failure, leader or follower. Produce / Consume is working usually. No data loss.

<div>

<figure><img src="/assets/broker_1_(1).jpeg" alt=""><figcaption><p>Broker 2 is down</p></figcaption></figure>

 

<figure><img src="/assets/broker_2.jpeg" alt=""><figcaption><p>Broker 3 is down</p></figcaption></figure>

 

<figure><img src="/assets/broker_3.jpeg" alt=""><figcaption><p>Broker (Leader) 1 is down</p></figcaption></figure>

</div>

Two out of three replicas/brokers are down. \
Produce/Consume will be stopped until at least one replica/follower is available. No data loss.

<div>

<figure><img src="/assets/broker_4.jpeg" alt=""><figcaption></figcaption></figure>

 

<figure><img src="/assets/broker_5.jpeg" alt=""><figcaption></figcaption></figure>

</div>

The entire cluster is down.&#x20;

Produce/Consume will be stopped until at least one leader and replica/follower are available. No data loss.

<figure><img src="/assets/broker_6.jpeg" alt=""><figcaption></figcaption></figure>

### 2. Kubernetes Workers

The kubernetes worker that holds one of the "followers" is down.\
The cluster will wait for the broker to be "Ready."\
Produce / Consume is working usually. No data loss.

<div>

<figure><img src="/assets/k8s_1_(1).jpeg" alt=""><figcaption></figcaption></figure>

 

<figure><img src="/assets/k8s_2.jpeg" alt=""><figcaption></figcaption></figure>

</div>

The kubernetes worker that holds the "leader" is down.\
The "leader" role has been taken over by broker 2.\
Producers / Consumers will might require a reconnect. No data loss.

<figure><img src="/assets/k8s_3_(1).jpeg" alt=""><figcaption></figcaption></figure>

The kubernetes workers that hold the "followers"/"Leader and a follower" are down.\
No quorum for stream. No cluster leader. The station is temporarily unavailable.\
Produce/Consume will be stopped until at least one replica/follower is available. No data loss.

<div>

<figure><img src="/assets/k8s_4.jpeg" alt=""><figcaption></figcaption></figure>

 

<figure><img src="/assets/k8s_5.jpeg" alt=""><figcaption></figcaption></figure>

</div>

The Kubernetes cluster is down; therefore, the Memphis cluster is down.

Produce/Consume will be stopped until at least one leader and replica/follower are available. No data loss.

<figure><img src="/assets/k8s_6.jpeg" alt=""><figcaption></figcaption></figure>

### 3. Kubernetes volumes (PVC)

One of the K8S worker's PVs owned by one of the followers is down.

The affected broker and PV require a manual restart.

<div>

<figure><img src="/assets/pv1.jpeg" alt=""><figcaption></figcaption></figure>

 

<figure><img src="/assets/pv2.jpeg" alt=""><figcaption></figcaption></figure>

</div>

The leader's volume is down.\
Data may not be available during the new leader election process.\
Producing messages is suspended until a new leader is elected.\
The affected broker and PV require a manual restart.

<figure><img src="/assets/pv3.jpeg" alt=""><figcaption></figcaption></figure>

Both followers' volumes are down.\
No quorum for stream. No cluster leader. The station is temporarily unavailable. \
Produce/Consume will be stopped until at least one replica/follower is available. No data loss.

<figure><img src="/assets/pv4.jpeg" alt=""><figcaption></figcaption></figure>

Both leader and follower volumes are down.\
No quorum for stream. No cluster leader. The station is temporarily unavailable. \
Produce/Consume will be stopped until at least one replica/follower is available. No data loss.

<figure><img src="/assets/pv5.jpeg" alt=""><figcaption></figcaption></figure>

All volumes are down; therefore, the Memphis cluster is down.

Produce/Consume will be stopped until at least one leader and replica/follower are available. No data loss.

<figure><img src="/assets/pv6_(1).jpeg" alt=""><figcaption></figcaption></figure>
