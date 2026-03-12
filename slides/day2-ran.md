% 5G Data Plane & RAN Hands-on
% OAI 5G NR Workshop — IIT Tirupati
% March 15, 2026 | Day 2


## Quick Recap — Day 1

- Deployed OAI 5G Core (docker) + gNB + nrUE in rfsim
- Got an IP address to UE, ping, ran iperf
- Understood registration and authentication procedures
- Analyzed messages in Wireshark
- **Things we skipped:** 
  - How did the UE actually get its IP address? 
  - How does your ping packet travel from UE to the core?
  - How do UE and gNB (cell tower) communicate over the air?

## Day 2 — Agenda

| Session | Topic |
|---------|-------|
| **Morning I** | PDU Session & Data Plane|
| **Morning I** | RAN Protocol Stack|
| **Morning II** | RAN Hands-on — experiments, scope visualization|
| **Afternoon** | RAN Hands-on, and advanced topics |

## 5G Protocol Stack

<div style="display: flex; align-items: start; gap: 2em; margin-top: 0;">

<div style="flex: 1; margin: 0; padding: 0;">

- **Control Plane**
  - **Non-Access Stratum (NAS)**: Functional layer to exchange control plane messages between UE and CN
  - **Signaling** - Who is the user? Is she/he/it a valid one? where should the user data should go? How to handle Roaming?
  - Establishment and management of communication sessions (NAS-SM)
  - Mobility management (NAS-MM)
  - Example NAS messages:  UE attach and registration, authentication etc.
  - **AMF** handles control signaling with the UE and RAN
- **User  Plane**
  - Access Stratum (AS)
  - **SMF** programs the **UPF**
  - **UPF** just forwards packets — it's told what to do by the SMF
  - Actual data packets Ex: YouTube video
</div>

<div style="flex: 1;">
<div style="display: flex; flex-direction: column; align-items: center;">
  <img src="images/proto-nas.jpeg" style="max-width: 80%; height: auto;">
  <p class="caption"><small>Control Plane</small></p>
<img src="images/proto-as.jpeg" style="max-width: 85%; height: auto;">
  <p class="caption"><small>User Plane</small></p>
</div>

<small>
  **NR-Uu:** Radio interface (UE ↔ gNB)
  **N2:** gNB ↔ AMF — NGAP over SCTP
  **N11:** AMF ↔ SMF — HTTP/2 (Service-Based Interface)
</small>
</div>

</div>



<!-- ============================================================ -->
<!-- SESSION 1: PDU SESSION & DATA PLANE                          -->
<!-- ============================================================ -->

## PDU Session


**Packet Data Unit (PDU) Session** — a data pipe from UE to the internet.

<div class="key-concept">
<strong>PDU Session =</strong> An end-to-end data path with:
<ul>
  <li>An <strong>IP address</strong> for the UE (that's the 10.0.0.x you saw!)</li>
  <li><strong>QoS rules</strong> defining how traffic is handled</li>
  <li>A <strong>DNN</strong> (Data Network Name) — configured in your ue.conf</li>
</ul>
</div>


## PDU Session Setup — What Happened After Registration

```{.mermaid}
sequenceDiagram
    participant UE
    participant gNB
    participant AMF
    participant SMF
    participant UPF

    UE->>AMF: PDU Session Establishment Request (NAS-SM)
    Note over AMF: AMF selects SMF

    AMF->>SMF: Nsmf_PDUSession_CreateSMContext
    SMF->>UPF: PFCP Session Establishment (PDR, FAR, QER)
    UPF-->>SMF: PFCP Session Establishment Response

    SMF-->>AMF: N1N2 Message Transfer
    AMF->>gNB: NGAP PDU Session Resource Setup Request
    Note over gNB: gNB allocates radio resources (DRB)
    gNB->>UE: RRC Reconfiguration (PDU Session Accept)
    UE-->>gNB: RRC Reconfiguration Complete

    gNB-->>AMF: NGAP PDU Session Resource Setup Response
    AMF->>SMF: Update SM Context
    SMF->>UPF: PFCP Session Modification

    Note over UE,UPF: Data path ready!
```

---

## GTP-U — Your Packet Inside a Tunnel

When you ping from UE, the packet travels like this:

<div class="diagram">
  <img src="images/end-to-end-ip.jpeg"
       alt="End-to-end 5G architecture: UE — RAN — Core — Internet">
</div>

</small>

- UE IP packet is 
   - Tagged with QoS Flow Identifier (QFI)
   - Tunneled with GTP-U (GPRS Tunneling Protocol – User Plane)
   - Transparent to RAN
</small>

---


## Find PDU Session & GTP-U in Wireshark {.action}

 - Restart your network as in day 1
 -  Start capture using Wireshark
 - Launch gNB, UE
 - Do a few pings, then stop

**Find these messages:**

| # | Protocol | Message | What to Look For |
|---|----------|---------|------------|
| 1 | PFCP | **Session Establishment Request** | PDR and FAR rules from SMF to UPF |
| 2 | PFCP | **Session Establishment Response** | UPF confirms |
| 3 | NGAP | **PDU Session Resource Setup** | QoS info, tunnel info |
| 4 | GTP-U | **G-PDU** | Your ping packet inside the tunnel! |

---

## Observe GTP-U Packets {.observe}

Filter for GTP:

```
gtp
```

Click on a GTP-U packet and expand:

1. **Outer IP header:** gNB IP ↔ UPF IP
2. **UDP header:** Port 2152
3. **GTP-U header:** TEID value
4. **Inner IP header:** UE IP ↔ ping target

**This is the tunnel!** Your ping is wrapped inside GTP-U between gNB and UPF.


## User Plane RAN Protocol Stack
<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/ip-qos-flow.jpeg"
       style="max-width: 95%; height: auto;">
</div>
<div style="flex: 1;">
<div style="display: flex; flex-direction: column; align-items: center;">
<img src="images/proto-as.jpeg" style="max-width: 95%; height: auto;">
  <p class="caption"><small>User Plane</small></p>
</div>

 - Quality of Service (QoS)
   - Guaranteed bit rate (GBR) and non-GBR
</div>
</div>


## User Plane RAN Protocol Stack
<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/ip-qos-flow.jpeg"
       style="max-width: 95%; height: auto;">
</div>
<div style="flex: 1;">
  <img src="images/ran-proto-entire.jpeg"
       style="max-width: 95%; height: auto;">
</div>
</div>

## Analogy: Ordering on an e-commerce platform

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/analogy.png" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">

- Think of an e-commerce platform as your 5G operator
- Once registered, you can order from various vendors
- Lets look at a hypothetical delivery system
  - **Alice and Bob** each order items with different urgency:
    - 🥬 **Fresh** — same-day, perishable
    - 💻 **Prime** — next-day, guaranteed
    - 🧹 **Normal** — whenever, best effort
- How an order turned into a delivery
  - Need to handle priorities
  - Dispatch hub should tackle how many delivery resources are available
  - Can an order fit in one delivery?
  - Do we need a small or bif vechcle for this delivery?
</div>
</div>


## Service Data Adaptation Protocol (SDAP)

<div style="display: flex; align-items: start; gap: 2em; margin-top: 0;">

<div style="flex: 1; margin: 0; padding: 0;">
  <img src="images/ip-qos-flow.jpeg"
       alt="End-to-end 5G architecture: UE — RAN — Core — Internet"
       style="max-width: 60%; height: auto;">

- **Maps QoS flows to Data Radio Bearers** [3GPP TS 37.324]
- Each IP flow gets a Qos Flow Identification (QFI) — defines treatment
- SDAP groups similar QFI flows into a **DRB**
- Marks QFI in UL and DL packets
- Multiple QoS flows can share one DRB if they need similar treatment.
- One SDAP entity per PDU session
</div>

<div style="flex: 1;">
  <img src="images/sdap.jpeg"
       style="max-width: 80%; height: auto;">
</div>

</div>

## Analogy

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/analogy.png" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">

- Quality of service -- Item with deliver tier (Fresh 🥬, Priority customer)   
- SDAP DRB grouping --  Group sharing same tier (🍎, 🥭, 🥛 => Fresh )
- 🧹 Normal items — Whenever resources are available (best effort)
</div>
</div>


## Packet Data Convergence Protocol (PDCP)

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/pdcp.jpeg" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">

- **Security and reliability** [3GPP TS 38.323]
- Ciphering — encrypts data (keys from Day 1 authentication!)
- Integrity protection — detects tampering
- Sequence numbering — every PDCP packet is numbered
- Duplicate discarding
- In-order delivery during handover
- Header compression (ROHC)
- One PDCP entity per DRB

</div>
</div>


## Radio link contro (RLC) 

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/rlc.jpeg" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">

- **Segmentation and reliable delivery** [3GPP TS 38.322]
- Segmentation — split large RLC packets to small ones that can be transported by lower layers 
- Reassembly at receiver
- Sequence numbering
- **ARQ** — retransmit lost segments (AM mode)
- One RLC entity per DRB
- RLC channel ↔ Logical channel
</small>

| Mode | Behavior | Use Case |
|------|----------|----------|
| **AM** (Acknowledged) | Numbered, confirmed, retransmitted | Web, file download |
| **UM** (Unacknowledged) | Numbered, no retransmission | Voice, video |
| **TM** (Transparent) | Pass-through | Broadcast, paging |
</small>

</div>
</div>


## Medium Access Control (MAC) 

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/mac.jpeg" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">

- Medium Access Control [3GPP TS 38.321]
- Multiplexing — logical channels → Transport Block
- Multiplexing of transport channels and forming the Transport Block (TB)
- Scheduling — which UE, how many resoursess, at which rate?
- Error correction at TB level through HARQ
- Maintaining UE synchronization
- Adaptive modulation (mcs), power control

</div>
</div>

## Analogy

<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">
  <img src="images/analogy.png" style="max-width: 85%; height: auto;">
</div>
<div style="flex: 1;">
- Construct a delivery parcel per user
  - **Parcel 1:** 🥬 Fresh + 💻 Prime → 🚛 truck
  - **Parcel 2:** 🧹 Normal → 🏍️  bike (later)
- Assign a vehicle and route
  - Good road → 🚛 truck (big parcel)
  - Congested → 🏍️  bike (small parcel)
- One order may need multiple parcels across multiple delivery slots
- Dispatch algorithm based on
  - Number of orders and Order sizes
  - Traffic conditions and road reports
  - Delivery person availability
</div>
</div>


## PHY — Physical Layer


<div style="display: flex; align-items: start; gap: 2em;">
<div style="flex: 1;">

| Channel | Direction | What it carries |
|---------|-----------|----------------|
| **PDSCH** | Downlink | User data + system info |
| **PUSCH** | Uplink | User data + control info |
| **PDCCH** | Downlink | Scheduling decisions (DCI) |
| **PUCCH** | Uplink | CQI, HARQ ACK/NACK |
| **PRACH** | Uplink | Random access preamble |
| **PBCH** | Downlink | Master Information Block |

</div>

<div style="flex: 1;">
 Channels are mapped on

- **Resource Grid:** frequency (RBs) × time (slots)
- **OFDM** — data across many subcarriers
- **Modulation** — QPSK, 16QAM, 64QAM, 256QAM
- **Channel coding** — LDPC (data), Polar (control)
- **MIMO** — multiple antennas
- **Channel estimation** — constant measurement

</div>
</div>



## OAI Code Pointers

| Layer | OAI Source Path |
|-------|----------------|
| **SDAP** | `openair2/SDAP/nr_sdap/` |
| **PDCP** | `openair2/LAYER2/nr_pdcp/` |
| **RLC** | `openair2/LAYER2/nr_rlc/` |
| **MAC** | `openair2/LAYER2/NR_MAC_gNB/gNB_scheduler.c` |
| **PHY RX & TX** | `openair1/SCHED_NR/phy_procedures_nr_gNB.c` |


<!-- ============================================================ -->
<!-- SESSION 3: RAN HANDS-ON                                      -->
<!-- ============================================================ -->

## RAN Hands-on {.section-divider}

---

## OAI Scope — See Your Signal Live

| Tool | How to build | How to use |
|------|-------------|------------|
| **nrscope** | `./build_oai --build-lib nrscope --ninja` | Run with `-d` flag |

❗Note: If you get the following error while compiling scope

```bash
CMake Error at openair1/PHY/TOOLS/CMakeLists.txt:38 (message):
  required library forms not found for building scopes
```
Install the required package
```bash
sudo apt-get install libforms-dev
```

What you can see:

- Constellation diagrams (PDSCH, PUSCH, PBCH)
- Channel frequency response
- Time domain signal energy
- LLR graphs

---

## Launch with Scope {.action}

```bash
cd ~/iittp-oai-hands-on/openairinterface5g/cmake_targets/ran_build/build
```

gNB with scope
```bash
sudo ./nr-softmodem -O ~/iittp-oai-hands-on/ran/conf/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.[0].serveraddr server -d
```

UE with scope
```bash
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O ~/iittp-oai-hands-on/ran/conf/nrue.conf -d
```

❗ Remote VM? Use **X11 forwarding**: `ssh -X user@host`

---

## gNB Scope {.observe}

:::::::::::::: {.columns}
::: {.column width="50%"}
![](images/gNB_scope.png)
:::
::: {.column width="50%"}

- **PUSCH constellation** — tight clusters = good signal
- **Channel frequency response** — across subcarriers
- **Time domain** — energy over time

**Try:** Run `iperf3` uplink while watching!

:::
::::::::::::::

---

## UE Scope {.observe}

:::::::::::::: {.columns}
::: {.column width="50%"}
![](images/ue_scope.png)
:::
::: {.column width="50%"}

- **PDSCH constellation** — QPSK=4pts, 16QAM=16, 64QAM=64
- **PBCH constellation** — always QPSK
- **Channel estimates** — frequency domain

**Try:** Run `iperf3` downlink — watch constellation get denser!

:::
::::::::::::::

---

## Channel Model with RFsim {.action}

Folder:
```bash
cd ~/iittp-oai-hands-on/openairinterface5g/cmake_targets/ran_build/build
```

Add to gNB and UE config:

```
@include "channelmod_rfsimu.conf"
```

Launch UE with channel model:

gNB:
```bash
sudo ./nr-softmodem -O ~/iittp-oai-hands-on/ran/conf/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.[0].serveraddr server --rfsimulator.[0].options chanmod --rfsimulator.[0].modelname AWGN -d
```

nrUE:
```bash
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O ~/iittp-oai-hands-on/ran/conf/nrue.conf -d --rfsimulator.options chanmod --rfsimulator.modelname AWGN
```

**Watch:** Constellations spread out with noise and fading.

---


## Experiment 2: TDD Pattern {.action}

**5ms pattern (default):**

```
dl_UL_TransmissionPeriodicity  = 6;   # 5ms
nrofDownlinkSlots              = 7;
nrofDownlinkSymbols            = 6;
nrofUplinkSlots                = 2;
nrofUplinkSymbols              = 4;
```

**2.5ms pattern:**

```
dl_UL_TransmissionPeriodicity  = 5;   # 2.5ms
nrofDownlinkSlots              = 2;
nrofDownlinkSymbols            = 6;
nrofUplinkSlots                = 2;
nrofUplinkSymbols              = 4;
```

**Exercise:** Run iperf DL+UL with both. Which gives better UL throughput? Why?

---

## Experiment 3: Bandwidth {.action}

Folder:
```bash
cd ~/iittp-oai-hands-on/openairinterface5g/cmake_targets/ran_build/build
```

**20 MHz (51 PRBs):**

gNB:
```bash
sudo ./nr-softmodem -O ~/iittp-oai-hands-on/ran/conf/gnb.sa.band78.fr1.51PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.[0].serveraddr server
```
nrUE:
```bash
sudo ./nr-uesoftmodem -r 51 --numerology 1 --band 78 -C 3609300000 --ssb 228 --rfsim -O ~/iittp-oai-hands-on/ran/conf/nrue.conf -d 
```

**100 MHz (273 PRBs):**

gNB:
```bash
sudo ./nr-softmodem -O ~/iittp-oai-hands-on/ran/conf/gnb.sa.band78.fr1.273PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.[0].serveraddr server
```
nrUE:
```bash
sudo ./nr-uesoftmodem -r 273 --numerology 1 --band 78 -C 3649260000 --ssb 516 --rfsim -O ~/iittp-oai-hands-on/ran/conf/nrue.conf -d 
```

**Exercise:** Compare throughput at 20MHz vs 40MHz vs 100MHz.

---

## Experiment 4: Multiple UEs {.action}

Folder:
```bash
cd ~/iittp-oai-hands-on/openairinterface5g/cmake_targets/ran_build/build
```

Make `multi-ue.sh` as executable
```bash
chmod +x ~/iittp-oai-hands-on/ran/multi-ue.sh
```

gNB
```bash
sudo ./nr-softmodem -O ~/iittp-oai-hands-on/ran/conf/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 --rfsim --rfsimulator.[0].serveraddr server
```

UE1
```bash
sudo ~/iittp-oai-hands-on/ran/multi-ue.sh -c1 -e
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O /home/oai/iittp-oai-hands-on/ran/conf/ue1.conf --rfsimulator.serveraddr 10.201.1.100
```

UE2
```bash
sudo ~/iittp-oai-hands-on/ran/multi-ue.sh -c2 -e
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O /home/oai/iittp-oai-hands-on/ran/conf/ue2.conf --rfsimulator.serveraddr 10.202.1.100
```

---

## Multiple UEs — Testing {.action}

```bash
# UE1
sudo ip netns exec ue1 bash
ping -I oaitun_ue1 192.168.70.135

# UE2
sudo ip netns exec ue2 bash
ping -I oaitun_ue1 192.168.70.135
```

**Exercise:** Run iperf from both UEs simultaneously. Watch the scheduler sharing resources.

```bash
# Cleanup
sudo ~/iittp-oai-hands-on/ran/multi-ue.sh -d1 -d2
```

<!-- ============================================================ -->
<!-- WRAP-UP                                                      -->
<!-- ============================================================ -->

## Workshop Summary {.section-divider}

---

## What We Covered

**Day 1 — Core Network & Control Plane**

- 5G Core architecture: SBA, Network Functions
- Deployed OAI CN + gNB + UE in rfsim
- NAS registration and 5G-AKA authentication
- Wireshark analysis

**Day 2 — Data Plane & RAN**

- PDU session establishment
- GTP-U tunneling
- RAN protocol stack: SDAP → PDCP → RLC → MAC → PHY
- OAI scope visualization
- Experiments: TDD, bandwidth, multi-UE

---

## Want to explore more?

**Resources:**

- OAI RAN: `https://gitlab.eurecom.fr/oai/openairinterface5g`
- OAI CN5G: `https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g`
- 3GPP Specs: TS 23.501, TS 38.300, TS 38.321

---

## Thank You!

**Instructors:** Rajeev Gangula, Rakesh Mundlamuri, Venkatareddy Akumalla, Chandra R. Murthy, Subash Mondal

**Workshop materials:** `https://github.com/RajeevGa/iittp-oai-hands-on`

**Feedback is welcome!**
