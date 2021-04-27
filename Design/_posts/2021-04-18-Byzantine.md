---
layout: post
title: Byzantine Fault Tolerance
---


When a node in a distributed system crashes, it stops completely. But there are other ways a node could be 'faulty'. Maybe a virus caused it to function differently or hackers programmed it to work differently. Any kind of issue with your node that causes it to function differently than expected is called a Byzantine fault.

* We can check for some changes in performances using SLIs or Service Level Indicators. These indicators could be some measure of performance, or uptime, or response time etc. 
* These SLIs have SLOs or service level objectives that state the target of your SLIs. For example, we want 99% of time, the response is under 500 millisecons
* These are then included in thr SLA or service level agreements between you and your customers. So this is the SLO plus consequences of what you will do if you dont meet SLO.

“I promise 99% uptime”
* How often do you check if your system is up? Sampling frequency
* What does it mean to be “up”? Domain of responsibility
* Over what time interval do you promise 99% uptime? Measurement interval


| Uptime | Downtime/month |

| 90% | 3 days |

| 99% | 7 hours |

| 99.9% | 43 minutes |

| 99.99% | 4 minutes |

| 99.999% | 25 seconds (5m/year)|

Need to have all the following 
* Multiple VMs
* Over multiple failure domains
* Automatic failover
* Monitoring
* Tolerance of planned outages
* Automatic machine provisioning (GCE)

#### The Two Generals problem

We have two generals of army A and B thats have to come to a consensus about fighting Army c. They lose if only one army fights C. So our aim is to get them both to not attack or attack.

How can you make sure the messages sent between these two are always in consensu given that there maybe hackers trying to corrupt messages.

A
"I want to attack
I will attack if B responds"

*sends message* -> If lost, its okay, neither attacks

B recieves message
*sends back* "I will attack if A responds" -> if lost, its okay, neither attacks

A recieves message, 
*sends back* Attack! -> If lost, A will attack but B won't.

Now what?

Can make a statistcal assumption that some number of messages will go through.

*The actual problem assumes the network is fail safe, but the actual nodes may fail.*

#### The Byzantine General Problem

5 generals, one is an imposter 

We dont really care what the imposter is saying, we just want the other generals to ALL agree on what the imposter is saying. If half think the imposter is saying attack and the other half think its retreat, there will be an issue. 

There is no solution for 3m+1 generals with > m traitors. So you need atleast 4 generals if you have one traitor.

For example, think of 3 generals with 1 traitor.

You will get one Attack message and one Retreat message, no matter which of the two is a traitor (assume one initiates the messages and the other sends you a follow up message), and wont know what to do.

With Four generals though, you will have three messages, one from each general. We dont care what the traitor sends (or does), because all the other generals will send the same thing, and in your message store of three messages, you will have two messages from two non traitors that will win majority, EVEN if the one to initiate the messages is the tritor and he sends different messages to all. 


Solution
TLDR: All generals send their decisions to everyone, and the majority decision is chosen. Assuming the number of traitors are lesser than or equal m, where hthere are 3m+1 generals, all will be okay!

