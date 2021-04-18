---
layout: post
title: Byzantine Fault Tolerance
---


When a node in a distributed system crashes, it stops completely. But there are other ways a node could be 'faulty'. Maybe a virus caused it to function differently or hackers programmed it to work differently. Any kind of issue with your node that causes it to function differently than expected is called a Byzantine fault.

We can check for some changes in performances using SLIs or Service Level Indicators. These indicators could be some measure of performance etc. These SLIs have SLOs or service level objectives that state the target of your SLIs. These are then included in thr SLA or service level agreements between you and your customers.


The Two Generals problem

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
