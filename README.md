# pfSense-and-Pi-Hole-Deployment
Repurposed a computer to run pfSense, and a Raspberry Pi to run Pi-Hole

## Pi-Hole Installation and Setup

### Components Used:
-	Pi 4 with 4 GB of RAM
-	Micro SD Card with 32 GB
-	PC with Micro SD Card reading capability – temporary use
-	USB Keyboard – temporary use
-	HDMI capable display w/micro connection – temporary use

### Software needed:

-	https://www.balena.io/etcher/
    -	Used to flash the OS to the SD card
-	https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit
    -	Go with the light version as we do not need a GUI interface.
----
## Initial Setup of Raspberry Pi OS

Flash the SD card using Balena with Raspberry Pi OS Lite, insert the card when done into the Pi and connect a monitor and keyboard.

Run initial setup and then reboot the device and go into the configuration


----
## Enable SSH

Use the command: 

- sudo raspi-config

![image](https://user-images.githubusercontent.com/102389429/180330475-de0799e3-2001-436e-987e-51de77b8572b.png)

![image](https://user-images.githubusercontent.com/102389429/180330718-cf5776fb-2e4d-4b21-bae3-dcd6d040fcd7.png)

![image](https://user-images.githubusercontent.com/102389429/180330722-901b0304-9947-48ed-81c7-8663bbe9142b.png)

Now plug the Pi 4 into the network and connect via SSH
 
![image](https://user-images.githubusercontent.com/102389429/180330824-bbd7f396-ad14-4a32-a037-b0ec478ecc8c.png)
 
First update to ensure latest version with command: 
- sudo apt update && sudo apt full-upgrade -y

Next change the time zone to the correct one with command:
- Dpkg-reconfigure tzdata

![image](https://user-images.githubusercontent.com/102389429/180332696-3c622eb2-42df-4931-a015-fc11b220a60b.png)

----
## Installation of Pi-Hole

Note! Make sure to have set a static IP address for the Pi in your router.

Start with the command to get root:
- sudo -i 
or 
- sudo -s

 ![image](https://user-images.githubusercontent.com/102389429/180332867-20dab92e-470b-439f-b798-8d283e1061b5.png)
 
Now to start the installation of Pi-Hole use the command:
- sudo curl -sSL https://install.pi-hole.net | bash

![image](https://user-images.githubusercontent.com/102389429/180332902-67adea57-6fb6-4699-ba86-cae833be8a60.png)

![image](https://user-images.githubusercontent.com/102389429/180332917-7ca91170-7986-4bd4-a37e-2b39809e0f6c.png)

 ![image](https://user-images.githubusercontent.com/102389429/180332933-d06d6dac-39af-401c-8e05-fa717d0a1a1a.png)

 ![image](https://user-images.githubusercontent.com/102389429/180332938-d6865e0e-3766-49fd-b82b-d013b0103200.png)

 ![image](https://user-images.githubusercontent.com/102389429/180332949-1b0ca40f-fbff-4467-99b8-38d6da92e973.png)

 ![image](https://user-images.githubusercontent.com/102389429/180332956-50588e85-5225-4d96-8c72-82b44758c9ec.png)
    
 ![image](https://user-images.githubusercontent.com/102389429/180332965-184d0db3-a553-4c04-913b-8581108a4b22.png)

 ![image](https://user-images.githubusercontent.com/102389429/180332980-f8ef9775-e83a-4c8d-bafe-516b3b1d0d23.png)
 
 ![image](https://user-images.githubusercontent.com/102389429/180332985-b1aecb4d-f841-4d64-b006-bdf4bdf3e75d.png)

![image](https://user-images.githubusercontent.com/102389429/180332988-f4ee0f9f-c7c7-4162-88e2-3ff13a1a4f42.png)

Last step is to change the default password for SSH make sure that you are out of root first then use the command:
- passwd

 ![image](https://user-images.githubusercontent.com/102389429/180333135-c248b8a4-4e68-4be1-8a57-78e6ce3e8746.png)

Now follow the link and log into the Pi-Hole dashboard.
 
 ![image](https://user-images.githubusercontent.com/102389429/180333145-4a2bb04e-1ddd-4c33-97db-e345d90c512d.png)

----
## Updating the IP address
To change the IP address of Pi-Hole after setup run the command:
- sudo nano /etc/dhcpcd.conf
Then scroll down and adjust these lines to the correct IP address.

 ![image](https://user-images.githubusercontent.com/102389429/180333178-9f530f29-c519-44d4-9ec4-f0141d6d0091.png)

----
## Installation of Unbound for Pi-Hole
Start with the command:
- sudo apt install unbound

Once the initial installation is complete run the command:
-	sudo wget https://www.internic.net/domain/named.root -qO- | sudo tee /var/lib/unbound/root.hints

That should create the file root hints file for the domains.

Next run the command:
- sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf

Then copy and past the entire text below into it:

```
 server:

    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. For most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

Next run the command to start unbound and the recursive server:
- sudo service unbound restart

Then test it with these commands:
- dig pi-hole.net @127.0.0.1 -p 5335
- dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5335
- dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335

First command should return an IP address, second should not and say failure, and third should return and IP address as shown below.
 
 ![image](https://user-images.githubusercontent.com/102389429/180333953-65dbb604-aa4d-4b18-8562-ee20e44e481e.png)
 
 ![image](https://user-images.githubusercontent.com/102389429/180333960-e9ba4425-ff3a-4234-969d-a95f78e846dd.png)

![image](https://user-images.githubusercontent.com/102389429/180333964-f7122bdf-a5a3-48b2-99ce-39a45c9692b4.png)
 
Last step is to configure Pi-Hole itself using the web interface.

Log in and go to “Settings” then “DNS” and enter as shown below:

![image](https://user-images.githubusercontent.com/102389429/180334009-00cd56c3-9274-4e2d-9984-3f7d4129c70d.png)
 
Then set Pi-Holes IP as your DNS server on your router and your router IP as the upstream DNS server on Pi-Hole.  

Then do one final test by visiting a website to see if it loads and if ads are removed.

----
## Installing NetData
Start off with the command:
- sudo wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh --disable-telemetry

This gives automatic updates and only stable releases, while not sending statistics back.

Load up the dashboard and have fun exploring by browsing to:
- http://”Pi-Hole IP”:19999/

For Temperature monitoring run the command:
- sudo nano /etc/netdata/edit-config charts.d.conf

Then uncomment “sensors=force” towards the bottom.

Then run:
- sudo systemctl restart netdata

Enjoy!
