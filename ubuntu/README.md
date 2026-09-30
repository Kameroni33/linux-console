# Ubuntu Console

## 1 - Installation

Download the latest version of [Ubunutu LTS](https://ubuntu.com/download/desktop).

<!-- TODO: add instructions for flashing and installation -->

## 2 - Setup

### 2.1 - Git Repository

Install [git](https://git-scm.com/).
```bash
sudo apt install -y git
```

Setup a SSH key.
```bash
sudo ssh-keygen
```

Clone the repository.
```
cd ~ # move to the home directory
git clone https://github.com/Kameroni33/linux-console.git
```

### 2.2 - RAID Configuration

First, identify all connected drives using `lsblk`. Then wipe each drive to be used for the RAID setup.
```bash
sudo wipefs -a /dev/sdx
```

Create a RAID-1 (Mirror) array (ex. `/dev/md0`).
```bash
sudo apt update
sudo apt install mdadm

sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdx ...

# check progress
watch cat /proc/mdstat
```

Create a new filesystem on the RAID array and create a mount point.
```bash
sudo mkfs.ext4 /dev/md0

sudo mkdir -p /mnt/storage
sudo mount /dev/md0 /mnt/storage
```

To persist the configuration, run `sudo blkid /dev/md0` and copy the UUID. Next edit `/etc/fstab` and add:
```/etc/fstab
UUID=...  /mnt/storage  ext4  defaults,noatime  0  2
```

Test the configuration with `sudo mount -a`.

Lastly, make the RAID auto-assemble.
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

Softlink the RAID storage to the user home directory and bookmark.
```bash
ln -s /mnt/storage ~/Storage
```

### 2.3 - Docker

Install the [Docker](https://www.docker.com/) engine. Instructions based on [dockerdocs](https://docs.docker.com/engine/install/ubuntu/) (install using the `apt` repository).
```bash
# Install prerequisites
sudo apt update
sudo apt install ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to apt sources
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Refresh apt package index
sudo apt update

# Install
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Once installed, follow the [post-installation step](https://docs.docker.com/engine/install/linux-postinstall/).
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

### 2.4 - Samba NAS

Install [Samba](https://www.samba.org/).
```bash
sudo apt update
sudo apt install samba

# Make a Samba user
sudo adduser sambauser
sudo smbpasswd -a sambauser
```

Edit `/etc/samba/smb.conf` and add the following:
```/etc/samba/smb.conf
[Storage]
   path = /mnt/storage
   browseable = yes
   read only = no
   writable = yes
   guest ok = no
   valid users = sambauser
   force create mode = 0660
   force directory mode = 0770
```

Update permissions and restart samba.
```bash
sudo groupadd storage
sudo usermod -aG storage sambauser
sudo usermod -aG storage $USER
sudo chown -R sambauser:storage /mnt/storage
sudo chmod 2770 /mnt/storage

sudo systemctl restart smbd
sudo systemctl enable smbd
```

### 2.5 - WireGuard VPN

#### Server Setup

```bash
sudo apt update
sudo apt install wireguard

# Generate keys
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

```/etc/wireguard/wg0.conf
[Interface]
Address = 10.66.66.1/24
ListenPort = 51820
PrivateKey = <server_private_key>

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.66.66.2/32
```

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# NAT setup for network forwarding
sudo iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT

sudo apt install iptables-persistent
sudo netfilter-persistent save
```

#### Client Setup

```bash
sudo apt update
sudo apt install wireguard
```

```/etc/wireguard/wg0.conf
[Interface]
PrivateKey = <client_private_key>
Address = 10.66.66.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server_public_key>
Endpoint = <server_ip_address>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# TODO: better to setup via network manager...
```

Useful commands:
```bash
sudo wg # check status
curl ipinfo.io/ip # check public IP address
ip route # check for default network connection
nmcli connection modify wg0 <...> # modify the connction (ex. `ipv4.route-metric 100`)
```

### 2.6 - Remote Desktop Protocol

<!-- #### Server Setup

```bash
sudo apt update
sudo apt install xrdp
```

#### Client Setup

```bash
sudo apt update
sudo apt install xfreerdp
``` -->

### 2.7 - Plex Media Server

Install [Plex](https://www.plex.tv/). Instruction based on [this article](https://linuxcapable.com/install-plex-media-server-on-ubuntu-linux/).
```bash
# Install pre-requisites
sudo apt update
sudo apt install ca-certificates curl gnupg apt-transport-https software-properties-common lsb-release

# Add Plex's official GPG key
curl -fsSL https://downloads.plex.tv/plex-keys/PlexSign.key | sudo gpg --dearmor -o /usr/share/keyrings/plex.gpg
sudo chmod a+r /usr/share/keyrings/plex.gpg

# Add the repository to apt sources
sudo tee /etc/apt/sources.list.d/plexmediaserver.sources <<'EOF'
Types: deb
URIs: https://downloads.plex.tv/repo/deb
Suites: public
Components: main
Signed-By: /usr/share/keyrings/plex.gpg
EOF

# Refresh apt package index
sudo apt update

# Install
sudo apt install plexmediaserver
```

#### qBittorrent Client

Install [qBittorrent](https://www.qbittorrent.org/). Instruction based on [this article](https://linuxcapable.com/how-to-install-qbittorrent-on-ubuntu-linux/).

```bash
# Install pre-requisites
sudo apt update
sudo apt install software-properties-common

# Import the PPA
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable

# Install
sudo apt install qbittorrent
```

### 2.8 - Steam

Install [Steam](https://store.steampowered.com/). Instruction based on [this article](https://linuxcapable.com/how-to-install-steam-on-ubuntu-linux/) (install via official steam repository).
```bash
# Install pre-requisites
sudo apt update
sudo apt install ca-certificates curl

# Add Steam's official GPG key
sudo install -m 0755 -d /usr/share/keyrings
curl -fsSL https://repo.steampowered.com/steam/archive/stable/steam.gpg | sudo gpg --dearmor -o /usr/share/keyrings/steam.gpg
sudo chmod a+r /usr/share/keyrings/steam.gpg

# Add the repository to apt sources
sudo tee  /etc/apt/sources.list.d/steam.sources <<EOF
Types: deb
URIs: https://repo.steampowered.com/steam/
Suites: stable
Components: steam
Architectures: amd64 i386
Signed-By: /usr/share/keyrings/steam.gpg
EOF

# Refresh apt package index
sudo apt update

# Install
sudo apt install steam-launcher

# Fix repository duplication (by deleting source list files and locking them as read-only)
sudo rm /etc/apt/sources.list.d/steam-beta.list
sudo rm /etc/apt/sources.list.d/steam-stable.list
sudo touch /etc/apt/sources.list.d/steam-beta.list /etc/apt/sources.list.d/steam-stable.list
sudo chmod 444 /etc/apt/sources.list.d/steam-beta.list /etc/apt/sources.list.d/steam-stable.list

# Refresh apt package index
sudo apt update
```

Once installed, run the Steam application and accept all console prompts to install requiired dependencies.

### 2.9 - Minecraft

Install [Minecraft](https://www.minecraft.net/en-us) Launcher for Debian + Debian Based from the [official site](https://www.minecraft.net/en-us/download). Once installed run the `.deb` file and install via the Software Installer application.

<!-- TODO: mods, shaders, etc... -->

To setup a minecraft server, see [dockercraft](https://github.com/Kameroni33/dockercraft).

### 2.10 Emulators

...

### 2.? - Other Software

The following applications can be installed using [snap](https://snapcraft.io/) pacakages.
```snap
sudo snap install <...>
```

* [Spotify](https://open.spotify.com/)
* [Discord](https://discord.com/)
* [Remmina (RDP Client)](https://remmina.org/)

## 3 - Customization

### 3.1 - Controller Support


## 4 - Updates & Maintainence

