# OAI_CU_DU_Split

### Hardware Requirement:
- Minimum two servers are required
- Current test is done on:
    - Server one --> CU --> local_IP --> 192.168.1.233
    - Server six --> DU --> local_IP --> 192.168.1.56

### Steps to follow for the deployment:

1. Clone this repo on both the servers(CU and DU)
```bash
git clone -b develop_v1.0 https://github.com/NgKore47/openairinterface5g.git
```

<hr>


2. Install UHD driver on `DU_Server`

```bash
sudo apt install -y libboost-all-dev libusb-1.0-0-dev doxygen python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools cmake build-essential
git clone https://github.com/NgKore47/uhd.git
cd uhd/host
mkdir build
cd build
cmake ../
make -j $(number of cores you want to use)
sudo make install
sudo ldconfig
sudo uhd_images_downloader
```

<hr>

3. Build OAI gNB

```bash
cd ~/openairinterface5g/
cd cmake_targets/
./build_oai -I

cd ..
source oaienv
cd cmake_targets/

# For CU_Server
./build_oai --gNB -c


# For DU_Server
./build_oai -w USRP --ninja --gNB -c

```

<hr>

4. Make changes in the conf file of cu and du

##### In CU_Server

Make sure following parameters in `~/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/cu.conf` file are updated:

```txt
TAC                             =   1
mcc                             =   001
mnc                             =   01
local_s_address                 =   CU_Server_IP
remote_s_address                =   DU_Server_IP
amf_ip_address                  =   192.168.70.132
GNB_IPV4_ADDRESS_FOR_NG_AMF     =   192.168.70.129
GNB_IPV4_ADDRESS_FOR_NGU        =   192.168.70.129

// Make sure to remove sd from conf file if present
```
> **NOTE:** Check [here](./patch_files/cu.patch) for `cu.patch` file


##### In DU_Server

Make sure following parameters in `~/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/du.conf` file are updated:

```txt
TAC                             =   1
mcc                             =   001
mnc                             =   01
local_n_address                 =   DU_Server_IP
remote_n_address                =   CU_Server_IP


// Make sure to remove sd from conf file if present
```
> **NOTE:** Check [here](./patch_files/du.patch) for `du.patch` file

<hr>

5. Build OAI-Core:

- Follow this docs for Ngkore: https://github.com/NgKore47/Ngkore-core
- Follow this docs for OAI-Core: https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md?ref_type=heads

##### OAI CN5G
- OAI CN5G pre-requisites
```bash
sudo apt install -y git net-tools putty

# https://docs.docker.com/engine/install/ubuntu/
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your username to the docker group, otherwise you will have to run in sudo mode.
sudo usermod -a -G docker $(whoami)
reboot
```

- OAI CN5G configuration files
```bash
wget -O ~/oai-cn5g.zip https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g
unzip ~/oai-cn5g.zip
mv ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
```
- Pull OAI CN5G docker images
```bash
docker pull mysql:8.0
docker pull oaisoftwarealliance/oai-amf:develop
docker pull oaisoftwarealliance/oai-nrf:develop
docker pull oaisoftwarealliance/oai-smf:develop
docker pull oaisoftwarealliance/oai-udr:develop
docker pull oaisoftwarealliance/oai-udm:develop
docker pull oaisoftwarealliance/oai-ausf:develop
docker pull oaisoftwarealliance/oai-spgwu-tiny:develop
docker pull oaisoftwarealliance/trf-gen-cn5g:jammy
docker pull oaisoftwarealliance/ims:latest
```

- Start OAI CN5G
```bash
cd ~/oai-cn5g
docker compose up -d
```


- Stop OAI CN5G
```bash
cd ~/oai-cn5g
docker compose down
```


<hr>

6. Run CU and DU seperately

##### Run CU
```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_gnb.conf --sa -E --continuous-tx
```

##### Run DU

> **NOTE:** Make sure to connect SDR to DU_Server
```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/du_gnb.conf --sa -E --continuous-tx
```

<hr>

### Logs
You can also check out the logs of cu and du [here](./logs/)