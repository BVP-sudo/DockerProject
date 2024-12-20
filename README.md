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

Insure that all the Python Libaries needed are installed

    sudo apt install python3-pip

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
            - "53:53/tcp"
            - "53:53/udp"
            - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
            - "80:80/tcp"
            environment:
            TZ: "America/Chicago"  # Adjust for your timezone
            WEBPASSWORD: "securepassword"  # Replace with a strong password
            volumes:
            - ./config/pihole:/etc/pihole
            - ./config/dnsmasq.d:/etc/dnsmasq.d
            restart: unless-stopped

Be careful with the indentations. They Matter.

## Step 3: Starting the Pi-hole Container:
Disable Ubuntu's default caching DNS stub resolver so pihole can listen on port 53

    sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf

Modify netplan:

    sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'

after doing this you must restart

    systemctl restart systemd-resolved

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

Update the DNS settings on a test device to use Pi-hole’s IP address.

On the device, set the DNS server to Pi-hole server address.

Visit an ad-heavy website or use a test tool to confirm that ads are being blocked.

## Sources:
Docker Lab - Project 2 powerpoint on provided on Harvey

The offical pi-hole docker repository:

    https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker
