# Project 3 - Wireguard Docker Container

# Step 1: Create a DigitalOcean Droplet
If you do not have a DigitalOcean account make one.

Log in to your DigitalOcean dashboard.

Click on Create Droplet.

Choose an operating system:

Use Ubuntu 22.04 (recommended).

Select a plan:

The basic plan with 1 GB RAM and 25 GB SSD is sufficient for most VPN setups. (This should cost about $6)

Choose a datacenter region close to your primary location for better latency.

Set a root password for authentication.

Click Create Droplet.

# Step 2: Update and Prepare the Server
SSH into your Droplet:
    
    ssh root@<your-droplet-ip>

Update the package list and upgrade installed packages:

    apt update && apt upgrade -y

Install Dependances:

    sudo apt install docker
    sudo apt install docker-compose
    sudo apt install wireguard

# Step 3: Install Wireguard

Make Wireguard Project folder:

    mkdir -p /opt/wireguard
    cd /opt/wireguard

Create a docker-compose.yml file:

    nano docker-compose.yml

Add the following configuration to docker-compose.yml:

    GNU nano 8.1                                                        
    version: '3.8'
    services:
    wireguard:
        container_name: wireguard
        image: linuxserver/wireguard
        environment:
        - PUID=1000
        - PGID=1000
        - TZ=American/New_York
        - SERVERURL=45.55.41.235 # Replace with the dropplet ip
        - SERVERPORT=51820
        - PEERS=pc1,pc2,phone1
        - PEERDNS=auto
        - INTERNAL_SUBNET=10.0.0.0
        ports:
        - 51820:51820/udp # this is adjustable
        volumes:
        - type: bind
            source: ./config/
            target: /config/
        - type: bind
            source: /lib/modules
            target: /lib/modules
        restart: always
        cap_add:
        - NET_ADMIN
        - SYS_MODULE
        sysctls:
        - net.ipv4.conf.all.src_valid_mark=1

Remember to set teh ServerUrl to the dropplet's ip.

# Step 4: Run Wireguard and Print logs

Run wireguard:

    docker-compose up -d 

Check logs to get the QR code:

    docker logs wireguard

# Step 5: Test VPN

### Mobile Device 

Open the Wireguard app and scan the QR code from the logs. 

Before connecting: 

Visit IPLeak.net and screenshot your local IP. 

After connecting: 

Turn on the Wireguard VPN and revisit IPLeak.net. 

Screenshot the VPN IP to confirm it is active.

### Laptop

Find the configuration file:

    /opt/wireguard/config

Copy the .conf file to your laptop

In our case the .conf file contained:

    [Interface]
    PrivateKey = iLwC8xbCzwVd5j9s7Et/72d6keAAVTlkmxcY/wX6Ako=
    ListenPort = 518
    .20
    Address = 10.0.0.2/32
    DNS = 10.0.0.1

    [Peer]
    PublicKey = P5GnsQQZk4X0KilGkKNg5ND/XZjV0KP7QDNuShSCcG4=
    PresharedKey = 5X6AWptfcPEHqhgi3nVlEb6vx833rLQic/ofI4TMy5s=
    AllowedIPs = 0.0.0.0/0, ::/0
    Endpoint = 45.55.41.235:51820

Import the file into the Wireguard app or CLI.



Follow the same steps as mobile to confirm functionality using IPLeak.net.
