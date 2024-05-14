---
title: Serveur Fedora
type: WIKI
categories:
  - system
description: Mise en place d'un serveur avec GPU sous Fedora.
author: Flavien PERIER <perier@flavien.io>
date: 2021-11-23 18:00
---

## Objectifs

- Le premier objectif va être d'installer [Docker](https://www.docker.com/) avec [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) afin de pouvoir utiliser la puissance de calcul de la carte graphique dans un conteneur [Jupyter](https://jupyter.org/) conteneurisé avec l'image [flavienperier/jupyter](https://hub.docker.com/r/flavienperier/jupyter).

- Le second objectif est de bénéficier d'une machine virtuelle [KVM](https://www.linux-kvm.org/page/Main_Page) avec single GPU Passthrough, pour faire tourner un Windows sur lequel pourront s'exécuter des jeux.

## Configuration matérielle

Le serveur est doté d'un [AMD Ryzen 9 5900X](https://www.amd.com/fr/products/cpu/amd-ryzen-9-5900x), d'une [Nvidia RTX-3080Ti](https://www.nvidia.com/fr-fr/geforce/graphics-cards/30-series/rtx-3080-3080ti/) ainsi que de 32Go de RAM.

## OS

Pour ce type de serveur, les distributions basées sur [RedHat](https://www.redhat.com/) sont relativement avantageuses. En effet, cette famille de distribution Linux étant destinée aux entreprises, elles bénéficient de nombreux avantages en termes de virtualisation et de conteneurisation.

Cependant, il peut-être dommage de ne pas bénéficier des dernières mises à jour du kernel quand on souhaite avoir une infrastructure performante. On oublie donc [RHEL](https://www.redhat.com/fr/technologies/linux-platforms/enterprise-linux) (de toute façon, pas de nécessité de support), [RockyLinux](https://rockylinux.org/) ou [CentOS](https://www.centos.org/).

La distribution qui semble donc le plus appropriée pour cette installation est [Fedora Server](https://fedoraproject.org/fr/server/).

## Installation

Utilisateur principal: admin

Toutes les commandes sont à exécuter en tant que root.

Pour une mise à jour de l'os vers une nouvelle version de Fedora :

```bash
dnf system-upgrade download --releasever=39
dnf system-upgrade reboot
```

## Nommage du serveur

Pour donner un nom à une machine, il suffit d'exécuter la commande :

```bash
echo "flavien-server" > /etc/hostname
```

### Base

Installation des drivers Nvidia et configuration minimale du serveur :

```bash
curl -s https://raw.githubusercontent.com/flavien-perier/linux-shell-configuration/master/linux-shell-configuration.sh | bash -

dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora39/x86_64/cuda-fedora39.repo
dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
dnf config-manager --add-repo https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo

dnf install kernel-devel kernel-headers
dnf module install nvidia-driver
dnf remove plymouth*
```

### Connection SSH

Pour accéder au serveur à distance il est important de commencer par créer une clé SSH afin de s'y connecter.

Depuis une machine client (autre que le serveur), il faut utiliser les commandes suivantes afin de générer les clés :

```bash
SERVER_IP=192.168.X.X

ssh-keygen -f ~/.ssh/flavien-server -t rsa -b 4096
ssh-copy-id -i ~/.ssh/flavien-server admin@$SERVER_IP

cat << EOL >> ~/.ssh/config
Host flavien-server
    HostName $SERVER_IP
    User admin
    Port 22
    IdentityFile ~/.ssh/flavien-server
    IdentitiesOnly yes
EOL
```

Par la suite, il faut modifier la configuration du serveur ssh avec la commande :

```sh
cat << EOL > /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/libexec/openssh/sftp-server
EOL
systemctl reload sshd
```

### Connection VPN

```bash
SERVER_IP=`ip route get 1.1.1.1 | awk 'NR==1 {print $(NF-2)}'`

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

cd /tmp
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.6/EasyRSA-3.1.6.tgz
tar xvf EasyRSA-*.tgz
rm EasyRSA-*.tgz
mv EasyRSA-* /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
echo 'set_var EASYRSA                 "$PWD"
set_var EASYRSA_PKI             "$EASYRSA/pki"
set_var EASYRSA_DN              "cn_only"
set_var EASYRSA_REQ_COUNTRY     "France"
set_var EASYRSA_REQ_PROVINCE    "France"
set_var EASYRSA_REQ_CITY        "Limoges"
set_var EASYRSA_REQ_ORG         "Flavien"
set_var EASYRSA_REQ_EMAIL       "perier@flavien.io"
set_var EASYRSA_REQ_OU          "Flavien"
set_var EASYRSA_KEY_SIZE        4096
set_var EASYRSA_ALGO            rsa
set_var EASYRSA_CA_EXPIRE       7500
set_var EASYRSA_CERT_EXPIRE     365
set_var EASYRSA_NS_SUPPORT      "no"
set_var EASYRSA_NS_COMMENT      "Flavien CA"
set_var EASYRSA_EXT_DIR         "$EASYRSA/x509-types"
set_var EASYRSA_SSL_CONF        "$EASYRSA/openssl-easyrsa.cnf"
set_var EASYRSA_DIGEST          "sha256"' | tee /etc/openvpn/easy-rsa/vars

# Server certificates
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req flavien-server nopass
./easyrsa sign-req server flavien-server
./easyrsa gen-dh
cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/server/
cp /etc/openvpn/easy-rsa/pki/dh.pem /etc/openvpn/server/
cp /etc/openvpn/easy-rsa/pki/private/flavien-server.key /etc/openvpn/server/
cp /etc/openvpn/easy-rsa/pki/issued/flavien-server.crt /etc/openvpn/server/
openvpn --genkey secret /etc/openvpn/server/ta.key

# Client certificates
./easyrsa gen-req client nopass
./easyrsa sign-req client client
cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/client/
cp /etc/openvpn/easy-rsa/pki/issued/*.crt /etc/openvpn/client/
cp /etc/openvpn/easy-rsa/pki/private/*.key /etc/openvpn/client/
rm /etc/openvpn/client/flavien-server.*

# Verify CA validity
openssl verify -CAfile /etc/openvpn/server/ca.crt /etc/openvpn/server/flavien-server.crt
openssl verify -CAfile /etc/openvpn/client/ca.crt /etc/openvpn/client/client.crt

# Add firewall rules
firewall-cmd --permanent --add-service=openvpn
firewall-cmd --permanent --zone=trusted --add-service=openvpn
firewall-cmd --permanent --zone=trusted --add-interface=tun0
firewall-cmd --add-masquerade
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o $SERVER_IP -j MASQUERADE
firewall-cmd --reload

# OpenVPN configuratiuon
echo 'port 1194
server 10.8.0.0 255.255.255.0
proto udp4
dev tap

ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/flavien-server.crt
key /etc/openvpn/server/flavien-server.key
dh /etc/openvpn/server/dh.pem

cipher AES-256-CBC

route-nopull
route 192.168.1.254

push "redirect-gateway def1"
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
push "dhcp-option DNS 1.1.1.1"
push "dhcp-option DNS 1.0.0.1"
push "dhcp-option DNS 151.80.222.79"

daemon
max-clients 5
client-to-client
user nobody
group nobody
keepalive 20 120
log-append /var/log/openvpn.log
verb 3' > /etc/openvpn/server/server.conf

# SeLinux
restorecon -Rv /etc/openvpn

# Start OpenVPN
systemctl enable openvpn-server@server
systemctl start openvpn-server@server
```

Par la suite il est possible de générer le fichier `ovpn` qui sera transmis au client afin qu'il puisse se connecter :

```bash
cat << EOL > ~/vm-jeux.ovpn
client
remote $SEREVER_IP 1194
proto udp4
dev tap

cipher AES-256-CBC

resolv-retry infinite
remote-cert-tls server
nobind
verb 3

route-nopull

<ca>
$(cat /etc/openvpn/server/ca.crt)
</ca>

<cert>
$(cat /etc/openvpn/client/client.crt)
</cert>

<key>
$(cat /etc/openvpn/client/client.key)
</key>
EOL
```

Il suffit alors de récupérer le fichier `~/clien.ovpn` sur ça machine à l'aide de SCP et de l'exécuter avec :

```bash
sudo sudo openvpn --config client.ovpn
```

### Wake on Lan

La configuration [Wake on Lan](https://fr.wikipedia.org/wiki/Wake-on-LAN) permet à un ordinateur d'être démarré par le réseau. Une fois en place, le seul prérequis pour pouvoir allumer un ordinateur dont le WoL est actif et de posséder son adresse mac et d'être sur le même réseau que lui.

Une partie de la carte réseau de la machine écoutera donc constamment le réseau même quand l'ordinateur est éteint. Si elle reçoit un paque contenant les octets FF FF FF FF FF FF suivi de 16 répétitions de l'adresse mac, le reste de la machine va démarrer (magic packet).

Il faut préalable activer le démarrage par le réseau dans le bios de la machine (configuration différente d'un constructeur à l'autre).

Puis au niveau de Linux, mettre les configurations suivantes :

```bash
ethtool -s enp4s0 wol g
echo 'ETHTOOL_OPTS="wol g"' > /etc/sysconfig/network-scripts/ifcfg-enp4s0
```

### Docker

Installation de Docker et de nvidia-docker :

```bash
dnf install docker-ce docker-ce-cli containerd.io nvidia-docker2 nvidia-container-toolkit

systemctl enable docker
systemctl start docker

usermod -aG docker admin

cat << EOL > /etc/nvidia-container-runtime/config.toml
disable-require = false
#swarm-resource = "DOCKER_RESOURCE_GPU"
#accept-nvidia-visible-devices-envvar-when-unprivileged = true
#accept-nvidia-visible-devices-as-volume-mounts = false

[nvidia-container-cli]
#root = "/run/nvidia/driver"
#path = "/usr/bin/nvidia-container-cli"
environment = []
debug = "/var/log/nvidia-container-toolkit.log"
#ldcache = "/etc/ld.so.cache"
load-kmods = true
no-cgroups = true
#user = "root:video"
ldconfig = "@/sbin/ldconfig"

[nvidia-container-runtime]
#debug = "/var/log/nvidia-container-runtime.log"
EOL
```

Par la suite pour lancer le conteneur il suffit de créer le fichier [docker-compose](https://docs.docker.com/compose/) avec les paramètres suivants :

```yaml
version: "3.9"
services:
  jupyter:
    image: flavienperier/jupyter
    container_name: jupyter
    restart: always
    volumes:
      - ./data/jupyter:/opt/notebooks
    ports:
      - 8080:8080
    environment:
      JUPYTER_PASSWORD: password
    devices:
      - /dev/nvidia0:/dev/nvidia0
      - /dev/nvidiactl:/dev/nvidiactl
      - /dev/nvidia-modeset:/dev/nvidia-modeset
      - /dev/nvidia-uvm:/dev/nvidia-uvm
      - /dev/nvidia-uvm-tools:/dev/nvidia-uvm-tools
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

Et d'exécuter le fichier en question avec notre utilisateur par défaut grâce à la commande `docker-compose up -d`.

### KVM & GPU Passtrough

Pour installer la base de KVM :

```bash
dnf install bridge-utils libvirt virt-install qemu-kvm libvirt-devel virt-top libguestfs-tools

systemctl enable libvirtd
systemctl start libvirtd

usermod -a -G libvirt admin
usermod -a -G kvm admin
setfacl -m g:qemu:rx /home/admin
```

Pour que le GPU Passtrough puisse fonctionner :

```bash
echo 'GRUB_TIMEOUT=3
GRUB_DISTRIBUTOR=Fedora
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on amd_iommu=pt iommu=1 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1 initcall_blacklist=simpledrm_platform_driver_init"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true' | tee /etc/default/grub

grub2-mkconfig -o /boot/grub2/grub.cfg
mkdir -p /etc/libvirt/hooks/qemu.d/vm-jeux/prepare/begin
mkdir -p /etc/libvirt/hooks/qemu.d/vm-jeux/release/end

wget https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu -O /etc/libvirt/hooks/qemu
chmod 750 /etc/libvirt/hooks/qemu

wget https://raw.githubusercontent.com/eretl/fedora-single-gpu-passtrough/main/start.sh -O /tmp/start-qemu.sh

cat /tmp/start-qemu.sh | sed 's/youruser/admin/g' > /etc/libvirt/hooks/qemu.d/vm-jeux/prepare/begin/start.sh

wget https://raw.githubusercontent.com/eretl/fedora-single-gpu-passtrough/main/revert.sh -O /etc/libvirt/hooks/qemu.d/vm-jeux/release/end/revert.sh

find /etc/libvirt/hooks/ -name "*.sh" -exec chmod 750 {} \;

echo VIRSH_GPU_VIDEO=pci_0000_`lspci | grep -i nvidia | grep -i vga | cut -f1 -d ' ' | tr ':' '_' | tr '.' '_'` > /etc/libvirt/hooks/kvm.conf
echo VIRSH_GPU_AUDIO=pci_0000_`lspci | grep -i nvidia | grep -i audio | cut -f1 -d ' ' | tr ':' '_' | tr '.' '_'` >> /etc/libvirt/hooks/kvm.conf
```

#### XML de configuration de la vm Windows 11

La configuration de cette machine virtuelle est prévue pour le jeu. Dans le XML ci-dessous un certain nombre d'optimisations ont pour but d'améliorer la performance du CPU vis-à-vis de la VM, mais également de dissimuler au mieux le fait qu'il s'agisse d'une machine virtuelle. En effet les anti-sheats tel qu'[Easy Anti-Cheat](https://www.easy.ac/) peuvent essayer de bloquer les jeux dans des machines virtuelles. Ces optimisations devraient rendre la détection beaucoup plus compliquée.

Avec [Virt manager](https://virt-manager.org/) il est possible de configurer l'hyperviseur à distance à travers un tunnel SSH.

Voici le XML de configuration utilisé pour l'interface réseau :

```xml
<network>
  <name>network</name>
  <uuid>******</uuid>
  <forward dev="enp4s0" mode="nat">
    <nat>
      <port start="1024" end="65535"/>
    </nat>
    <interface dev="enp4s0"/>
  </forward>
  <bridge name="virbr1" stp="on" delay="0"/>
  <mac address="52:54:00:2a:e1:56"/>
  <domain name="network"/>
  <ip address="192.168.2.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.2.1" end="192.168.2.254"/>
    </dhcp>
  </ip>
</network>
```


Voici le XML de configuration utilisé pour la machine Windows 11 :

```xml
<domain type="kvm">
  <name>vm-jeux</name>
  <uuid>******</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://microsoft.com/win/11"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="KiB">20971520</memory>
  <currentMemory unit="KiB">20971520</currentMemory>
  <memoryBacking>
    <source type="memfd"/>
    <access mode="shared"/>
  </memoryBacking>
  <vcpu placement="static">16</vcpu>
  <iothreads>2</iothreads>
  <cputune>
    <vcpupin vcpu="0" cpuset="12"/>
    <vcpupin vcpu="1" cpuset="13"/>
    <vcpupin vcpu="2" cpuset="14"/>
    <vcpupin vcpu="3" cpuset="15"/>
    <vcpupin vcpu="4" cpuset="16"/>
    <vcpupin vcpu="5" cpuset="17"/>
    <vcpupin vcpu="6" cpuset="18"/>
    <vcpupin vcpu="7" cpuset="19"/>
    <emulatorpin cpuset="0-1"/>
    <iothreadpin iothread="1" cpuset="0-1"/>
    <iothreadpin iothread="2" cpuset="2-3"/>
  </cputune>
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-8.1">hvm</type>
    <firmware>
      <feature enabled="yes" name="enrolled-keys"/>
      <feature enabled="yes" name="secure-boot"/>
    </firmware>
    <loader readonly="yes" secure="yes" type="pflash" format="qcow2">/usr/share/edk2/ovmf/OVMF_CODE_4M.secboot.qcow2</loader>
    <nvram template="/usr/share/edk2/ovmf/OVMF_VARS_4M.secboot.qcow2" format="qcow2">/var/lib/libvirt/qemu/nvram/vm-jeux_VARS.qcow2</nvram>
    <boot dev="hd"/>
    <smbios mode="host"/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <runtime state="on"/>
      <synic state="on"/>
      <stimer state="on">
        <direct state="on"/>
      </stimer>
      <reset state="on"/>
      <vendor_id state="on" value="Asus"/>
      <frequencies state="on"/>
      <reenlightenment state="on"/>
      <tlbflush state="on"/>
      <ipi state="on"/>
      <evmcs state="off"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <vmport state="off"/>
    <smm state="on"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <topology sockets="1" dies="1" cores="8" threads="2"/>
    <cache mode="passthrough"/>
    <feature policy="require" name="topoext"/>
  </cpu>
  <clock offset="localtime">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type="block" device="disk">
      <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
      <source dev="/dev/nvme1n1"/>
      <target dev="sda" bus="sata"/>
      <address type="drive" controller="0" bus="0" target="0" unit="0"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x02" slot="0x00" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pcie-root"/>
    <controller type="pci" index="1" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="1" port="0x10"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="2" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="2" port="0x11"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x1"/>
    </controller>
    <controller type="pci" index="3" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="3" port="0x12"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x2"/>
    </controller>
    <controller type="pci" index="4" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="4" port="0x13"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x3"/>
    </controller>
    <controller type="pci" index="5" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="5" port="0x14"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x4"/>
    </controller>
    <controller type="pci" index="6" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="6" port="0x15"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x5"/>
    </controller>
    <controller type="pci" index="7" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="7" port="0x16"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x6"/>
    </controller>
    <controller type="pci" index="8" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="8" port="0x17"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x7"/>
    </controller>
    <controller type="pci" index="9" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="9" port="0x18"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="10" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="10" port="0x19"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x1"/>
    </controller>
    <controller type="pci" index="11" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="11" port="0x1a"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x2"/>
    </controller>
    <controller type="pci" index="12" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="12" port="0x1b"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x3"/>
    </controller>
    <controller type="pci" index="13" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="13" port="0x1c"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x4"/>
    </controller>
    <controller type="pci" index="14" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="14" port="0x1d"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x5"/>
    </controller>
    <controller type="sata" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1f" function="0x2"/>
    </controller>
    <interface type="network">
      <mac address="52:54:00:e5:75:75"/>
      <source network="network"/>
      <model type="e1000e"/>
      <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
    </interface>
    <input type="tablet" bus="usb">
      <address type="usb" bus="0" port="1"/>
    </input>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <tpm model="tpm-crb">
      <backend type="emulator" version="2.0"/>
    </tpm>
    <graphics type="spice" autoport="yes">
      <listen type="address"/>
    </graphics>
    <audio id="1" type="spice"/>
    <video>
      <model type="vga" vram="16384" heads="1" primary="yes">
        <resolution x="1920" y="1080"/>
      </model>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
    </video>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x0a" slot="0x00" function="0x0"/>
      </source>
      <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
    </hostdev>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x0a" slot="0x00" function="0x1"/>
      </source>
      <address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>
    </hostdev>
    <watchdog model="itco" action="reset"/>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </memballoon>
  </devices>
</domain>
```

### [Cockpit](https://cockpit-project.org/)

Cockpit est une interface web préinstallée sur Fedora serveur permettant de manager notre serveur à distance. Certains modules sont déjà préinstallés (comme celui permettant de gérer [SElinux](https://selinuxproject.org/)), mais d'autres peuvent être installés afin de gérer Docker, KVM et quelques autres aspects du système directement depuis l'interface. Pour ce faire :

```bash
dnf install cockpit-machines

cd /tmp
wget https://github.com/mrevjd/cockpit-docker/releases/download/v2.0.3/cockpit-docker.tar.gz
tar xvf cockpit-docker.tar.gz -C /usr/share/cockpit

systemctl restart cockpit
```

## Sources

- [Arch Wiki - QEMU](https://wiki.archlinux.org/title/QEMU)
- [How to Install or Upgrade Nvidia Drivers on Rocky Linux 8](https://www.linuxcapable.com/how-to-install-or-upgrade-nvidia-drivers-on-rocky-linux-8/)
- [nvidia-docker Installation Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
- [Comprehensive guide to performance optimizations for gaming on virtual machines with KVM/QEMU and PCI passthrough](https://mathiashueber.com/performance-tweaks-gaming-on-virtual-machines/)
- [Howto do QEMU full virtualization with bridged networking](https://ahelpme.com/linux/howto-do-qemu-full-virtualization-with-bridged-networking/)
- [Creating a Windows 10 kvm VM on the AMD Ryzen 9 3900X using Qemu 4.0 and VGA Passthrough](https://www.heiko-sieger.info/creating-a-windows-10-vm-on-the-amd-ryzen-9-3900x-using-qemu-4-0-and-vga-passthrough/)
- [Fedora single gpu passtrough](https://github.com/eretl/fedora-single-gpu-passtrough)
- [Ubuntu 18.04 VFIO GPU passthrough with a single GPU (onboard automatically disables itself)](https://gist.github.com/TomFaulkner/389e8e2e9525e11afe2e775355954cdf)
- [Passthrough Helper for Manjaro](https://github.com/pavolelsig/passthrough_helper_manjaro)
- [Single GPU Passthrough (VFIO) for Nvidia + Ryzen CPU [Arch-based]](https://www.reddit.com/r/VFIO/comments/ir58fi/single_gpu_passthrough_vfio_for_nvidia_ryzen_cpu/)
- [Installing Tensorflow on Fedora 34](https://rickycorte.medium.com/installing-tensorflow-on-fedora-34-6d2f97651e60)
- [cgroup issue with Nadia container runtime on Debian testing](https://github.com/NVIDIA/nvidia-docker/issues/1447)
- [Docker plugin for Cockpit](https://github.com/mrevjd/cockpit-docker)
- [Bridged Host-VM Network](https://briantward.github.io/bridge-host-vm/)
- [Enabling the RPM Fusion repositories](https://docs.fedoraproject.org/en-US/quick-docs/setup_rpmfusion/)
- [Launch OpenVPN Access Server On Red Hat](https://openvpn.net/vpn-software-packages/redhat/)
- [How To Install OpenVPN on CentOS/RHEL 8](https://tecadmin.net/install-openvpn-centos-8/)
- [Arch Wiki - Easy-RSA](https://wiki.archlinux.org/title/Easy-RSA)
- [Fedora Wiki - OpenVPN](https://fedoraproject.org/wiki/OpenVPN)
