# Infrastructure Documentation

## TrueNAS Services (`10.0.220.145`)
- **Syncthing**: Port `8384` - File synchronization service
- **Immich**: Port `2283` - Photo/video management platform
- **Filebrowser**: Port `8080` - Web-based file manager
- **Portainer NAS**: Port `31015` - Standalone Portainer instance (not connected to master)
- **MongoDB**: Port `27017` - MongoDB database
- **MinIO**: Port `9001` - Object storage console
- **PostgreSQL Fast**: Port `5433` - NVME-based ZFS pool (`/mnt/zfs02-fast/postgres-fast`)
- **PostgreSQL Slow**: Port `5432` - HDD-based ZFS pool (`/mnt/zfs01/postgres`)
- **Neo4j Slow Test**: Port `7474` - HDD-based ZFS pool (`/mnt/zfs01/neo4j-slow`)
- **Neo4j Fast Test**: Port `7475` - NVME-based ZFS pool (`/mnt/zfs02-fast/neo4j-fast`)

### ZFS Dataset Structure
Datasets have been created on each pool (fast/slow) for database services. Child datasets can be created within these parent datasets for specific projects:
- **Fast Pool** (`zfs02-fast`): NVME-based storage for high-performance databases
  - `postgres-fast` - PostgreSQL fast instance
  - `neo4j-fast` - Neo4j fast test instance
- **Slow Pool** (`zfs01`): HDD-based storage for standard databases
  - `postgres` - PostgreSQL slow instance
  - `neo4j-slow` - Neo4j slow test instance

## NFS Shares for Proxmox VMs
NFS shares mounted on Jellyfin stack VM (`10.0.75.113`):
- `/mnt/Stripe1TB/downloads` - Downloads directory
- `/mnt/Stripe1TB/media` - Main media directory
- `/mnt/Stripe1TB/media/movies` - Movies directory
- `/mnt/Stripe1TB/media/shows` - TV shows directory

## Proxmox Cluster VMs

### Hardware Details
*   Aitherios:
    *   Aitherios: (Αιθέριος): Rooted in "aether," representing the upper sky and brightness. 
    *   Ryzen Threadripper, 2 RTX 3060 ti's, 128GB RAM
        *   Each GPU: 8 GB VRAM
        *   CPU 16 Cores, 32 Threads
    *   1TB NVME
    * PSU:
    * Networking: *(For WoL purposes, etc.)*
	    * Interface (10GB NIC): enp9s0f0 
			* MAC: 40:a8:f0:b3:f5:38
		* Virtual Bridge (Holds ip 10.0.0.225 and bridges it to NIC): vmbr0
			* MAC: a8:a1:59:9b:6a:3a
*   Sterianos:
    *   Στεριανός (Sterianos): This term conveys a sense of solidity, stability, or groundedness, providing a contrast to the more ethereal nature of "Αιθέριος."
    *   Intel Core i9 13990K, RTX 3090, 32GB RAM
        *   GPU: 24 GB VRAM
        *   24 Cores, 8 Performance Cores, 16 Efficiency Cores
        *   32 Threads
		* Interface: enp7s0 (10GB NIC1) *(For WoL purposes)*
		* MAC: 1a:4b:24:aa:9e:f4 *(For WoL purposes)*
    *   2TB NVME Drive
    * PSU:
*   NAS Machine: https://pcpartpicker.com/list/G3QwpK
    *   Lower-Grade Machine for Running TrueNAS scale + Other services
    *   10.0.220.145
    *   Intel XEON E5 
    *   GTX 1650 
    *   16GB RAM
    *   3 (or 4) 4TB HHDs + 1 12TB HHD
    *   PSU: 
* Phidios
	* **Phidios (Φειδιός)** – Inspired by "φειδώ" (pheidó), meaning "thrift" or "conservation," representing its power-conscious nature
	* AMD Ryzen 7 7900X 8-Cores
	* RTX 3080 12GB 
	* 96 GB 5600 speed RAM
	* 1TB NVME Drive
	* PSU: 
	* Interface: vmbr0 *(For WoL purposes)*
	* ipv4: 10.0.0.154
	* MAC: ffe083ca80 *(For WoL purposes)*

### High Availability Configuration
All VMs are configured for HA across the entire cluster except dedicated Kubernetes VMs.

### VM Inventory
- **misc-host** (`10.0.2.247`): Misc Docker services host
  - Services: Admin-Mongo (`1234`), pgAdmin4 (`5050`), GhostBoard (`8081`), Filebrowser (`8080`), SFTP (`8081`), sftpgo (`8888`)
- **portainer-master** (`10.0.9.28`): Portainer master instance (`9000`)
- **jellyfin-host** (`10.0.75.113`): Jellyfin media stack VM
- **mc-host** (`10.0.0.155`): Minecraft servers host
- **nginx-proxy-manager-host** (`10.0.0.136`): Main reverse proxy (`81`)
- **wireguard-host** (`10.0.173.26`): Wireguard VPN server
- **admin-panel** (`10.0.153.222`): SSH gateway for K3s VMs, Ansible management
  - Services: Semaphore (`3000`)

## Portainer Architecture
- **Master**: `10.0.9.28:9000` - Central Portainer instance
- **Agents**: All Docker VMs connect as agents to master except NAS Portainer
  - mc-host (`10.0.0.155:9001`)
  - nginx-proxy-manager-host (`10.0.0.136:9001`)
  - wireguard-host (`10.0.173.26:9001`)
  - misc-host (`10.0.2.247:9001`)
  - jellyfin-host (`10.0.75.113:9001`)

## Database Services
- **MongoDB**: `10.0.220.145:27017` (Admin-Mongo UI: `10.0.2.247:1234`)
- **PostgreSQL Fast**: `10.0.220.145:5433` - NVME-based ZFS pool (`/mnt/zfs02-fast/postgres-fast`)
- **PostgreSQL Slow**: `10.0.220.145:5432` - HDD-based ZFS pool (`/mnt/zfs01/postgres`)
- **PostgreSQL UI**: pgAdmin4 at `10.0.2.247:5050`
- **Neo4j Slow Test**: `10.0.220.145:7474/browser` - HDD-based ZFS pool (`/mnt/zfs01/neo4j-slow`)
- **Neo4j Fast Test**: `10.0.220.145:7475/browser` - NVME-based ZFS pool (`/mnt/zfs02-fast/neo4j-fast`)
- **MinIO**: Console at `10.0.220.145:9001`

## Additional Services
- **Nginx Proxy Manager**: `10.0.0.136:81` - Main reverse proxy
- **Rancher**: `10.0.1.50:443` - Future K3s cluster management
- **Longhorn**: `10.0.1.51` - Future distributed storage for K3s
- **MeTube**: YouTube downloader (`10.0.75.113:8501`)

## Jellyfin Media Stack (`10.0.75.113`)
Complete media automation stack with ExpressVPN integration:

### Services
- **ExpressVPN**: Container with VPN tunnel (ports `6881`, `8080`)
- **qBittorrent**: Torrent client using VPN network (port `8080`)
- **Radarr**: Movie management (port `7878`)
- **Sonarr**: TV show management (port `8989`) 
- **Prowlarr**: Indexer management (port `9696`)
- **Jellyfin**: Media server (ports `8096`, `8920`)
- **Jellyseerr**: Request management (port `5055`)

### Volume Mounts
All services mount NFS shares from TrueNAS:
- Downloads: `/mnt/Stripe1TB/downloads`
- Media: `/mnt/Stripe1TB/media`
- Movies: `/mnt/Stripe1TB/media/movies`
- Shows: `/mnt/Stripe1TB/media/shows`

### Network Architecture
- qBittorrent uses ExpressVPN's network for secure torrenting
- Other services run on host network for local access
- All media stored on TrueNAS NFS shares

## Minecraft Host (`10.0.0.155`)
Ubuntu 22.04 docker vm template

### Port-Forwarded Ports
- `25565-25570`

### Servers
- **server1**: Creative mess-around server for everyone
- **server2**: Me, vivi, nat survival world
- **server3**: Everyone amplified survival world

### Datapacks Installed
- **AFK Display** - Greys out a player's name in the player list if they haven't moved for 5 minutes, indicating they're AFK
- **Anti Enderman Grief** - Prevents endermen from picking up blocks, helping to preserve your builds
- **Anti Ghast Grief** - Stops ghast explosions from destroying blocks, protecting your structures in the Nether
- **Bat Membranes** - Allows bats to drop phantom membranes upon death, providing an alternative source for this item
- **Cauldron Concrete** - Enables players to convert concrete powder into concrete by right-clicking it into a cauldron filled with water
- **Cauldron Mud** - Lets players create mud by using a water-filled cauldron and dirt blocks
- **Classic Fishing Loot** - Restores the old fishing loot tables, allowing players to catch valuable items while fishing
- **Confetti Creepers** - When creepers explode, they emit a harmless confetti effect instead of causing damage
- **Custom Nether Portals** - Allows the creation of nether portals in any shape and size, offering more flexibility in portal design
- **Double Shulker Shells** - Ensures that shulkers drop two shells upon death, making it easier to craft shulker boxes
- **Dragon Drops** - Modifies the Ender Dragon's drops to include an elytra and a dragon egg upon defeat
- **Durability Ping** - Notifies players with a ping sound when their tool or armor durability is low
- **Ender Chest Always Drops** - Ensures that ender chests drop themselves when broken, even without a Silk Touch pickaxe
- **Fast Leaf Decay** - Speeds up the decay of leaves after tree logs are removed, reducing waiting time
- **Glass Always Drops** - Makes glass blocks drop themselves when broken, eliminating the need for Silk Touch
- **Graves** - Upon death, a grave is created containing all your items, which can be retrieved by right-clicking
- **Husks Drop Sand** - Adds a chance for husks to drop sand upon death, providing an alternative sand source
- **Mini Blocks** - Introduces miniature versions of blocks that can be used for decoration
- **More Effective Tools** - Enhances tool effectiveness, allowing tools to break blocks faster or more efficiently
- **More Mob Heads** - Increases the variety of mob heads that can drop when mobs are killed by players
- **Multiplayer Sleep** - Allows one player to sleep to skip the night on multiplayer servers, rather than requiring all players to sleep
- **Name Colors** - Enables players to change the color of their in-game names using commands
- **Painting Picker** - Allows players to cycle through different paintings by right-clicking on them with an empty hand
- **Silence Mobs** - Lets players silence mobs by naming them "Silence Me" with a name tag, preventing them from making noise
- **Silk Touch Budding Amethyst** - Permits the collection of budding amethyst blocks using a Silk Touch pickaxe
- **Unlock All Recipes** - Automatically unlocks all crafting recipes for players upon joining the game
- **Villager Death Messages** - Displays a message in chat when a villager dies, indicating the cause of death
- **Villager Workstation Highlights** - Highlights the workstation a villager is linked to, making it easier to identify their profession
- **Wandering Trades** - Adds mini blocks to the Wandering Trader's trade offers, providing more decorative options

immich api key: `3p3NV5vwU6li4j1y4ppYWAP4QY0yVfcLK4MTKfY8g6c`

# Proxmox Details:

### **Docker VM Template**
* HA template for spinning up any standalone VMs within the Proxmox Cluster
### **Installed Services & Utilities**
- **Core System Tools:**
    - `htop` – Interactive process viewer for system monitoring
    - `nvtop` – GPU usage monitoring for NVIDIA cards
    - `qemu-guest-agent` – Enhances virtualization performance & communication
- **Docker Support:** Pre-installed and configured (with compose)
- **Auto IP assignment via DHCP**
- **Logging Optimization (not tested):**
    - Docker logs capped at **10MB per container** (max **3 log files**) via `daemon.json`
    - System logs capped at **100MB** in `/etc/systemd/journald.conf`
