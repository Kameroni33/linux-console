# Ubuntu Console

## 1 - Installation

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

### 2.3 - Samba NAS

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
Architecture: amd64
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


### 2.6 - VPN

## 3 - Customization

### 3.1 

## 4 - Updates & Maintainence

