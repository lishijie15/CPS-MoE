# CPS-MoE: Zero-Shot Detection, Identification, and Localization of Cyber–Physical Compound Events in Power Distribution Systems

Shijie Li<sup>1</sup>, Qianwen Xu<sup>1,\*</sup> (\*Corresponding author)

<sup>1</sup>[Department of Electric Power and Energy Systems](https://www.kth.se/epe/), [KTH Royal Institute of Technology](https://www.kth.se/en), Stockholm, 11428, Sweden

---

## Abstract

*With the increasing penetration of photovoltaics and electric vehicles, modern power distribution systems (PDSs) face a higher risk of physical events, during which attackers may opportunistically trigger cyber events, causing the two to jointly impact the PDS in compound forms that are difficult to anticipate.
Existing diagnosis methods rely on event patterns seen during training and thus struggle to diagnose unseen combinations. Moreover, the power and communication features of the constituent events become intermixed, further complicating event identification and node localization.
To address these challenges, we propose a novel model named cyber-physical synergistic mixture-of-experts (CPS-MoE), which learns only from normal operation and single events yet achieves zero-shot detection, identification, and node localization of unseen compound events.
Specifically, the proposed model fuses power and communication features according to the correspondence between nodes and telemetry channels under each remote terminal unit, and combines global experts with physical and cyber event specialists, thereby preserving the distinct features of each event while supplementing local evidence.
In addition, the loss of each event is computed using only normal samples and samples of that event, thereby reducing the mutual exclusion constraints among events.
Extensive experiments on scenarios constructed by the proposed cyber-physical coupled model in the IEEE 34 node and IEEE 123 node test feeders demonstrate that the proposed CPS-MoE achieves state-of-the-art performance in diagnosing unseen compound events, with particularly notable advantages in event identification and node localization.*

![image](pictures/CPS-MoE.png)

## Introduction of Our Model

- To achieve zero-shot diagnosis of cyber-physical compound events, we propose CPS-MoE, which consists of detection, identification, and localization submodels with independent parameters and determines the occurrence of each event type individually, so that it requires no predefined combination classes and can be trained using only data from normal operation and single events.
- To distinguish events and determine their scopes of influence, we fuse the spatiotemporal features of power and communication measurements according to the correspondence between the nodes and telemetry channels under each remote terminal unit (RTU), and employ the global experts of each task to extract holistic anomaly characteristics while preserving an independent representation for each event type.
- To prevent local features from being overwhelmed in the holistic representation, we design lightweight physical and cyber event specialists, which separately extract information such as power variations, telemetry anomalies, and communication degradation to supplement and correct the diagnostic results of the global experts.
- To simulate cyber-physical compound events in practical operation as closely as possible, we propose CPCM, which constructs scenarios of five single events and six of their combinations in the IEEE 34 node and IEEE 123 node test feeders, where the proposed CPS-MoE consistently achieves the best performance.

## Introduction to CPCM

To evaluate the zero-shot diagnosis of cyber-physical compound events, test scenarios must reflect how physical events alter the operating states of the PDS and how cyber events distort the telemetry received by the dispatch center, while retaining the labels and affected nodes of each constituent event.

To address this, we propose CPCM, a cyber-physical coupled model that integrates PV systems and EV charging stations (EVCSs) into the IEEE 34 node and IEEE 123 node test feeders and links the PDS nodes, RTUs, and the dispatch center through a communication system. The active and reactive power measurements of loads, PV systems, and EVCSs are organized into telemetry channels and assigned to their nodes and RTUs. From the reception records of these channels, five communication features are computed for each RTU, namely the update ratio, timeout ratio, mean data age, maximum data age, and sequence advancement ratio. The physical ground truth obtained by an unbalanced three-phase power flow is stored separately from the received telemetry, so that the effects of physical and cyber events can be distinguished. On this basis, CPCM models the following events.

- PV: sudden drops in PV output at the target nodes, e.g., due to cloud shading.
- EV: demand surges at the target EVCSs, e.g., when arriving EVs charge simultaneously, with the charging power capped at the EVCS power limit.
- FDIA: offsets added to the targeted power telemetry.
- Worm: telemetry delays introduced as the worm propagates through the RTU communication network.
- DoS: interrupted updates from the targeted RTUs, whose last received telemetry values are retained.
- Compound events: six combinations of the above events, whose constituent events may overlap in active periods and target scopes (see Data Split).

![image](pictures/CPCM.png)

## Simulation Setting

### Data Split

Each annual scenario (8,784 hourly time steps in 2016) is split by calendar month: 7 months for training, 2 months for validation, and 3 months for testing.

To evaluate zero-shot diagnosis, all methods are trained and validated only on normal operation and the five single events (PV, EV, FDIA, Worm, and DoS). The six compound events (PV+EV, PV+FDIA, EV+DoS, FDIA+Worm, Worm+DoS, and PV+FDIA+Worm) are used for testing only.

### Training Protocol

- CPS-MoE consists of three submodels for detection, identification, and localization, which are trained separately.
- Each submodel is trained with five random seeds, and all reported results are averaged over the five seeds.
- Under the same seed, each submodel shares the parameters of its global experts across the IEEE 34 node and IEEE 123 node test feeders, whereas the event specialists and decision thresholds are determined separately for each feeder.
- The detection and identification probabilities are calibrated on the validation set by temperature scaling or Platt scaling, whichever yields the lower Brier score, and the localization outputs are not calibrated. The decision thresholds are then selected by maximizing the validation F1-score (see Section IV-C of the paper).
- All three submodels are optimized with AdamW.

### Additional Settings

- In the global experts, the weights $\beta_j$ of the cross-domain representations in the PV and EV experts and of the communication representation in the FDIA expert are fixed constants, with $\beta_{\mathrm{PV}}=0.25$, $\beta_{\mathrm{EV}}=0.10$, and $\beta_{\mathrm{FDIA}}=$ `[value]` (see Section IV-B1 of the paper).
- The two checks applied to newly added EV predictions (see Section IV-B4 of the paper) are logistic regression (LR) models on standardized features. The first check uses six features, namely the proximity to the EVCS power limit, capacity utilization, peer deviation, historical deviation at the same time of day, unreliability of the historical reference, and RTU data staleness. The second check uses nine features, namely the EV specialist score, the calibrated EV score of the global experts, an auxiliary FDIA score, and six statistics of charging variations. The auxiliary FDIA score is produced by a separate LR model with 20 input features.

### Implementation

All neural networks, including the baselines, are implemented in PyTorch 2.0.1.

### Hardware Requirements

- A single GPU with 8 GB of memory is sufficient for CPS-MoE inference, and a GPU with at least 12 GB of memory is recommended for running all baselines at a batch size of 512. Detailed memory usage is reported in the efficiency analysis of the paper.
- Training requires more memory than inference, since intermediate activations are stored for backpropagation. On GPUs with less memory, reduce the batch size.
