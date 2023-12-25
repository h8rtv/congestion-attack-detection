# congestion-attack-detection
Detection of simulated traffic congestion in VANETs using Temporal Graph Networks.

The implementation details and processing done can be seen in the notebook.

## Introduction

Ad-hoc networks are a decentralized wireless network model, composed of several devices, with the function of sending and receiving messages. One type of ad-hoc network is the vehicular ad-hoc network (VANET). VANETs have a dynamic topology, that is, the connection between vehicles can change over time.

VANETs mainly contain two types of communication:

* Road-to-vehicle communication
* **Vehicle-to-vehicle communication**

The applications of VANETs are varied, but are mainly based on the exchange of cooperative messages, for understanding and decision-making in traffic. There are two main elements within a VANET, the On-Board Unit (OBU) and the Road Side Unit (RSU). The OBU is responsible for sending vehicle information on the network, such as position and speed, while the RSU can send road information, such as weather and accidents that have occurred.

These networks, like any other, can be the target of attacks that prevent them from functioning correctly. One of the possible attacks is the congestion simulation attack, where an attacker simulates several other vehicles stopped or at low speed near another vehicle, in a way that forces it to stop as well.

To mitigate attacks, one possibility is to use some technique to detect when an attack occurs and then notify some entity so that a decision can be made regarding it. It can also be used later as a form of offline audit.

Therefore, the objective of this work is to find and apply a technique that uses graph neural networks to detect congestion attacks.

## Methodology

To determine the technique that will be applied to detect attacks, I searched the models implemented in the Torch Geometric library for any fit in the context of dynamic and temporal graphs. I decided to use TGN, as it was the best for the type of graph being treated. It will be better explained in the following sections.

From the dataset, the idea of this APS arises primarily from the VeReMi dataset, which contains the congestion attack and will be the main object of study. It will be more detailed later.

## Dataset

VeReMi is a dataset simulated with VEINS that contains failures and attacks in VANETs. It is presented in the form of "log" messages between vehicles, and contains two types of messages, GPS (receiving the vehicle's own position, speed, acceleration and current direction) and BSM (messages from other vehicles receiving the position, speed, acceleration and steering of nearby vehicles). Only a part of the dataset containing genuine behavior and congestion attack behavior is used.

## Dynamic graphs

A dynamic graph is one in which connections change over time. There are two main ways to represent dynamic graphs:

* Discrete dynamic graph: these are sequences of "snapshots" of static graphs.

* Continuous dynamic graph: more general representation, and are represented as lists of events with timestamps, which may include the addition or removal of edges, nodes or transformations of edge features.

The model chosen to be worked on is the Temporal Graph Network (TGN), presented in [1]. This model uses a continuous dynamic graph as input, therefore, it is necessary to transform the VeReMi data to apply the model.

### Data processing

The data is already in the form of event lists, so few changes are needed.

In this work, a case is tested where upon receiving the message, the user does not have the global ID of the sending vehicle, only a pseudonym used for occasional transmissions. This makes the work more complex to identify attackers, as each vehicle simulated by the attacker will be interpreted as a separate vehicle.

The graph has 1501368 interactions, between 3726 senders and 1688 recipients. In this problem, the graph is treated as a bipartite graph, since the senders cannot be re-identified.

To consume the data, an `InMemoryDataset` present in the torch_geometric library is used, and this dataset contains `TemporalData`, which is the representation used for continuous dynamic graphs in this library.

Here, I reduced the number of features after some tests, leaving only the position and speed features of both the sender and the receiver.

This class was adapted from [2].

## Results

It was possible to achieve 81% accuracy in the task, with a precision of 86% and a recall of 87%, resulting in an F1 measure of 86%. This result, together with the fact that the dataset contains 64% of interactions as attacks, shows that the results were positive and that the model demonstrates acceptable performance for the task.

| Accuracy | Precision | Recall   | F1       |
|----------|-----------|----------|----------|
| 0.816981 | 0.862779  | 0.871389 | 0.862079 |

## Conclusion

In the tests carried out, the TGN Model managed to achieve a promising result, even with the limitations of the sender.

A limitation of this work is the lack of comparisons with other methods to validate the quality of the results obtained with this model. Also, no optimizations were made to the hyperparameters in order to maximize performance.

Finally, it is concluded that the TGN technique is very interesting and can be applied in the context of identifying attacks in VANETs.

## References and resources

[1] Rossi, E., Chamberlain, B., Frasca, F., Eynard, D., Monti, F., & Bronstein, M. (2020). Temporal Graph Networks for Deep Learning on Dynamic Graphs. Available at: https://arxiv.org/abs/2006.10637

[2] Torch Geometric Team. JODIE Dataset Loader. Available at: https://pytorch-geometric.readthedocs.io/en/latest/_modules/torch_geometric/datasets/jodie.html#JODIEDatase

[3] Torch Geometric Team. Temporal Graph Network (TGN) example. Available at: https://github.com/pyg-team/pytorch_geometric/blob/master/examples/tgn.py

[4] Torch Geometric Team. Question about LastNeighborLoader. GitHub Issues. Available at: https://github.com/pyg-team/pytorch_geometric/discussions/2153

