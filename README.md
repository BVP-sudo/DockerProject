# Setting Up Pi-hole with Docker and Docker Compose on an Ubuntu-based System

Course: CYB 3353 - System Administration

Student: Benjamin Pikul

Instructor: Justin Miller

Institution: University of Tulsa

## Introduction:
Pi-hole is a network-wide ad blocker that can enhance your browsing experience by blocking intrusive ads and trackers. This guide walks you through setting up Pi-hole using Docker and Docker Compose on an Ubuntu-based system. By the end of this tutorial, you'll have a working Pi-hole instance running in a Docker container and accessible through a web interface.

## Prerequisites
A basic understanding of the Linux Command Line

System Requirements:

An Ubuntu-based Linux system (Ubuntu, Debian, or Mint).

    This proccess should replicatable on different distros of Linux if you alter some
    commands to match the differnt distro (I.e. using pacman instead of apt if you 
    wanted to use Arch Linux). To create this application Windows or MacOs requires 
    notably differnt steps and this documentation should not be used as a guide for 
    creating the application on those Operating Systems

Inusre the system is up to date (Update the System):

    sudo apt update && sudo apt upgrade -y

Install Docker:

    sudo apt install docker.io -y

Install Docker Compose

    sudo apt install docker-compose -y

Start and Enable Docker Service:

    sudo systemctl start docker
    
    sudo systemctl enable docker

Verify Installations:

Docker version:

    docker --version

Docker Compose version:

    docker-compose --version

## Step 1: Setting Up the Project Directory:
Create a directory to house the Pi-hole project:

    mkdir ~/pihole-setup && cd ~/pihole-setup

Create subdirectories for configuration files:

    mkdir -p config/pihole config/dnsmasq.d

## Step 2: Writing the docker-compose.yml File:

Open a new file named docker-compose.yml in your project directory:

nano docker-compose.yml

Add the following content:

        version: '3'
        services:
        pihole:
            container_name: pihole
            image: pihole/pihole:latest
            ports:
            - "80:80"         # Web interface
            - "53:53/tcp"     # DNS
            - "53:53/udp"
            environment:
            TZ: "America/Chicago"  # Adjust for your timezone
            WEBPASSWORD: "securepassword"  # Replace with a strong password
            volumes:
            - ./config/pihole:/etc/pihole
            - ./config/dnsmasq.d:/etc/dnsmasq.d
            restart: unless-stopped
            networks:
            pihole_net:
                ipv4_address: 192.168.1.2
    
        networks:
        pihole_net:
            driver: bridge
            ipam:
            config:
                - subnet: 192.168.1.0/24

## Step 3: Starting the Pi-hole Container:
Run Docker Compose to create and start the Pi-hole container:

    sudo docker-compose up -d

Verify that the container is running:

    sudo docker ps

Check the container logs for errors or initial setup information:

    sudo docker logs pihole

## Step 4: Accessing the Pi-hole Web Interface
Open a browser and navigate to the Pi-hole dashboard:

    http://<your-host-IP>/admin
    
Replace <your-host-IP> with your server's IP address.

Log in using the password set in the WEBPASSWORD environment variable.

## Step 5: Testing Pi-hole

Update the DNS settings on a test device to use Pi-hole’s IP address (192.168.1.2).

On the device, set the DNS server to 192.168.1.2.

Visit an ad-heavy website or use a test tool to confirm that ads are being blocked.

## Sources:
Docker Lab - Project 2 powerpoint on provided on Harvey

The offical pi-hole docker repository:

    https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker
