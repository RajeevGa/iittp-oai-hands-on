# Required Software Installation

# 1. Git
```shell
sudo apt-get update
sudo apt-get install git
```

# 2. Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
```

:exclamation: Add your username to the docker group, otherwise you will have to run in sudo mode. 

```bash
sudo usermod -a -G docker $(whoami)
```

:exclamation: Now, close the terminal tab and re-open it.

# 2. Wireshark (optional)

:exclamation: Make sure to select `yes` when the Wireshark installation asks you whether non-superusers should be able to capture packets.
Otherwise, you will have to run in sudo mode.

```bash
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt update
sudo apt install -y net-tools wireshark tshark
```

# 3. Networking
```bash
sudo apt install -y iperf3
```

# 4. Tutorial Repository
```bash
git clone https://github.com/rakesh-mundlamuri/oai_ants_tutorial_2025.git
```

# 5. Openairinterface repository
Open an another terminal and clone the Openairinterface repository
```bash
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
```

