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

### 2.3 - Samba NAS

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

Restart samba and update permissions.
```bash
sudo systemctl restart smbd
sudo systemctl enable smbd

sudo chown -R sambauser:sambauser /mnt/storage
sudo chmod -R 775 /mnt/storage
```

### 2.4 - ... VPN

### 2.4 - Docker

Install the [Docker](https://www.docker.com/) engine. Instructions based on [dockerdocs](https://docs.docker.com/engine/install/ubuntu/) (install using the `apt` repository).
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Refresh Apt package index
sudo apt update

# Install
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Once installed, follow the [post-installation step](https://docs.docker.com/engine/install/linux-postinstall/).
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

### 2.5 - Steam

Install [Steam](https://store.steampowered.com/). Instruction based on [this article](https://linuxcapable.com/how-to-install-steam-on-ubuntu-linux/) (install via official steam repository).
```bash
# Add Steam's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /usr/share/keyrings
curl -fsSL https://repo.steampowered.com/steam/archive/stable/steam.gpg | sudo gpg --dearmor -o /usr/share/keyrings/steam.gpg
sudo chmod a+r /usr/share/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee  /etc/apt/sources.list.d/steam.sources <<EOF
Types: deb
URIs: https://repo.steampowered.com/steam/
Suites: stable
Components: steam
Architectures: amd64 i386
Signed-By: /usr/share/keyrings/steam.gpg
EOF

# Refresh Apt package index
sudo apt update

# Install
sudo apt install steam-launcher

# Fix repository duplication (by deleting source list files and locking them as read-only)
sudo rm /etc/apt/sources.list.d/steam-beta.list
sudo rm /etc/apt/sources.list.d/steam-stable.list
sudo touch /etc/apt/sources.list.d/steam-beta.list /etc/apt/sources.list.d/steam-stable.list
sudo chmod 444 /etc/apt/sources.list.d/steam-beta.list /etc/apt/sources.list.d/steam-stable.list

# Refresh Apt package index
sudo apt update
```

Once installed, run the Steam application and accept all console prompts to install requiired dependencies.

### 2.6 - Minecraft

Install [Minecraft](https://www.minecraft.net/en-us) Launcher for Debian + Debian Based from the [official site](https://www.minecraft.net/en-us/download). Once installed run the `.deb` file and install via the Software Installer application.

<!-- TODO: mods, shaders, etc... -->

To setup a minecraft server, see [dockercraft](https://github.com/Kameroni33/dockercraft).

### 2.? - Other Software

The following applications can be installed using [snap](https://snapcraft.io/) pacakages.
```snap
sudo snap install <...>
```

* [Spotify](https://open.spotify.com/)
* [Discord](https://discord.com/)

## 3 - Customization

### 3.1 - Controller Support


## 4 - Updates & Maintainence

