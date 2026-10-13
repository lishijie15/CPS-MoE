# CPS-MoE: Zero-Shot Detection, Identification, and Localization of Cyber–Physical Compound Events in Power Distribution Systems

Shijie Li<sup>1</sup>, Qianwen Xu<sup>1,</sup>*  (*Corresponding author)<br />

<sup>1</sup>[Department of Electric Power and Energy Systems](https://www.kth.se/epe/), @[KTH Royal Institute of Technology](https://www.kth.se/en), Stockholm, 11428, Sweden

-----

## Abstract

*With the large-scale integration of photovoltaics and electric vehicles, modern power distribution systems (PDSs) face a higher risk of power disturbances, and attackers may opportunistically launch cyberattacks during physical disturbances, causing physical and cyber events to jointly impact the PDS in concurrent forms that are difficult to anticipate.
Existing diagnosis methods rely on event patterns seen during training and thus struggle to diagnose unseen combinations. Moreover, the power and communication features of concurrent events overlap with or mask one another, further complicating event identification and node localization.
To address these challenges, we propose a novel model named cyber-physical synergistic mixture-of-experts (CPS-MoE), which learns only from the normal state and single events yet achieves zero-shot detection, identification, and node localization of unseen compound events.
Specifically, the proposed model fuses power and communication features according to the mapping among telemetry channels, nodes, and remote terminal units, and combines global experts with dedicated physical-event and cyber-event experts, thereby preserving the distinct features of each event while supplementing local evidence.
Meanwhile, the loss is computed using only normal samples and samples of the corresponding single event, thereby reducing the mutual exclusion constraints among events.
Extensive experiments on the IEEE 34-node and 123-node feeders, constructed with a cyber-physical coupling model, demonstrate that the proposed CPS-MoE achieves state-of-the-art performance in diagnosing unseen compound events, with particularly notable advantages in event identification and node localization.*

![image](pictures/CarbonGPT.png)

## Introduction of Our Model

* To 
* To 
* To 
* To 

## Introduction to the PTN

To evaluate the 

To address this, 

![image](pictures/RPTM.png)

## Simulation Setting

### Data Split

Each annual scenario (8,784 hourly time steps in 2016) is split by calendar month: 7 months for training, 2 months for validation, and 3 months for testing.

To evaluate zero-shot diagnosis, all methods are trained and validated only on normal operation and the five single events (PV, EV, FDIA, Worm, and DoS). The six compound events (PV+EV, PV+FDIA, EV+DoS, FDIA+Worm, Worm+DoS, and PV+FDIA+Worm) are used for testing only.

### Training Protocol

- CPS-MoE consists of three submodels for detection, identification, and localization, which are trained separately.
- Each submodel is trained with five random seeds, and all reported results are averaged over the five seeds.
- Under the same seed, each submodel shares the parameters of its global expert network across the IEEE 34-node and IEEE 123-node feeders, whereas the specialized experts and decision thresholds are determined separately for each feeder.
- Model outputs are calibrated by temperature scaling on the validation set, and the decision thresholds are selected by maximizing the validation F1-score (see Section IV-C of the paper).
- All three submodels are optimized with AdamW.

### Implementation

All neural networks, including the baselines, are implemented in PyTorch 2.0.1.

### Hardware Requirements

- A single GPU with 8 GB of memory is sufficient for CPS-MoE inference, and a GPU with at least 12 GB of memory is recommended for running all baselines at a batch size of 512. Detailed memory usage is reported in the efficiency analysis of the paper.
- Training requires more memory than inference, since intermediate activations are stored for backpropagation. On GPUs with less memory, reduce the batch size.
- Training requires more memory than inference, since intermediate activations are stored for backpropagation. On GPUs with less memory, reduce the batch size.
