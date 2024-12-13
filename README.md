# Moria Server Setup Guide

Tested in conteneur with image:  ubuntu-20.04-standard_20.04-1

## Base System Setup
Update and prepare the base system:
```bash
apt update
apt upgrade -y
apt install software-properties-common lsb-release wget -y
```
**Explanation:**
- `apt update`: Updates the list of available packages and their versions.
- `apt upgrade -y`: Installs the latest versions of all installed packages.
- `software-properties-common lsb-release wget`: Installs utilities required for managing repositories and fetching files.

---

## Install WINE
Set up WINE for running Windows applications:
```bash
mkdir -pm755 /etc/apt/keyrings
wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/dists/$(lsb_release -cs)/winehq-$(lsb_release -cs).sources
dpkg --add-architecture i386
apt update
apt install --install-recommends winehq-staging -y
apt install cabextract winbind screen xvfb -y
```
**Explanation:**
- WINE setup includes adding its repository and key, enabling multi-architecture support (i386), and installing dependencies for running Windows applications.

---

## Install Winetricks
Utility for managing WINE configurations:
```bash
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
mv -v winetricks /usr/local/bin
```
**Explanation:**
- Downloads the latest version of Winetricks, makes it executable, and moves it to the system's PATH for easy access.

---

## Install SteamCMD
Set up SteamCMD to manage server files:
```bash
# For Ubuntu
apt-add-repository multiverse

# For Debian
apt-add-repository non-free

apt update
apt install steamcmd -y
steamcmd
# Inside the Steam terminal, type:
quit
```
**Explanation:**
- Adds the appropriate repositories for SteamCMD and installs it. Runs SteamCMD once to set up necessary files.

---

## Create a User for the Server
Set up a dedicated user for running the server:
```bash
useradd -m moria
su moria
cd
/usr/games/steamcmd +@sSteamCmdForcePlatformType windows +force_install_dir /home/moria/moriaserver +login <steam_username> +app_update 3349480 +quit
```
**Explanation:**
- Creates a new user `moria` and switches to it. Downloads the server files using SteamCMD.
- Replace `<steam_username>` with your Steam account credentials.

---

## Configure WINE and the Server
Perform initial setup and generate configuration files:
```bash
# In moria
xvfb-run winetricks -q vcrun2019
xvfb-run wine64 /home/moria/moriaserver/MoriaServer.exe
# Should crash it with ctrl + C after few seconds, and relaunch it
xvfb-run wine64 /home/moria/moriaserver/MoriaServer.exe
```
**Explanation:**
- Installs `vcrun2019` (Visual C++ Redistributable 2019) required for the server.
- Runs the server twice to complete initial setup.

---

## Generate and Modify Configuration Files
Run the server to generate `.ini` configuration files:
```bash
xvfb-run wine /home/moria/moriaserver/MoriaServer.exe
```
**Explanation:**
- `xvfb-run` allows running the server without a graphical display.
- After the first run, modify `.ini` configuration files as needed.

To find the invite code:
```bash
cat /home/moria/moriaserver/Moria/Saved/Logs/Moria.log
```
To reset the invite code:
```bash
rm /home/moria/moriaserver/Moria/Saved/Config/InviteSeed.cfg
xvfb-run wine /home/moria/moriaserver/MoriaServer.exe
```

---

## Create a Systemd Service for the Server
Automate server startup:
```bash
nano /etc/systemd/system/moriaserver.service
```
Paste the following content:
```ini
[Unit]
Description=Moria Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/xvfb-run /usr/bin/wine /home/moria/moriaserver/MoriaServer.exe
WorkingDirectory=/home/moria/moriaserver
User=moria
Group=moria
Restart=on-failure
Environment=DISPLAY=:0

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
systemctl enable moriaserver.service
systemctl start moriaserver.service
```
**Explanation:**
- Creates a systemd service to manage the server, ensuring it starts on boot and restarts if it crashes.

---
