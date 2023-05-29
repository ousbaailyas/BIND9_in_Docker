# Unleashing the Power of BIND9 in Docker: A Comprehensive Guide

## Introduction

DNS (Domain Name System) plays a crucial role in modern infrastructure by translating human-readable domain names into IP addresses that computers can understand. It acts as a backbone of the internet, enabling seamless communication and accessibility across networks. In this article, we will explore the powerful combination of BIND9, a widely-used DNS server software, and Docker, a popular containerization platform, to enhance DNS management.

Using BIND9 in conjunction with Docker offers numerous benefits for DNS administrators and system operators. Let's take a closer look at why this combination is advantageous:

1. **Isolation and Portability:** Docker allows us to encapsulate BIND9 and its dependencies into a container, providing an isolated environment that can be easily replicated across different systems. This ensures consistent behavior and simplifies deployment, regardless of the underlying infrastructure.
2. **Scalability and Flexibility:** With Docker, we can effortlessly scale BIND9 by spinning up multiple instances or containers to handle increased DNS traffic. This flexibility enables us to adapt to changing demands and ensures reliable DNS resolution for our applications.
3. **Version Control and Rollbacks:** Docker enables us to version our BIND9 container images, providing a mechanism for easy rollbacks in case of configuration issues or updates. This simplifies maintenance and reduces the risk of service disruptions during DNS management operations.
4. **Resource Optimization:** Docker's lightweight containerization approach ensures efficient resource utilization. By running BIND9 in a container, we can maximize server efficiency, reduce hardware costs, and simplify resource allocation for DNS services.

Throughout this article, we will delve into the process of Dockerizing BIND9, setting up a BIND9 Docker container, customizing the BIND9 configuration, and exploring best practices for secure DNS management. By harnessing the combined power of BIND9 and Docker, we can unlock new possibilities for managing DNS infrastructure efficiently and effectively.

## Prerequisites

Before we proceed, ensure that you have the following prerequisites in place:

- **Linux Server**: Prepare a Linux server where you intend to run BIND9 in Docker. This server should have Docker and Docker Compose already installed.
- **Docker and Docker Compose**: If you haven't installed Docker and Docker Compose on your server yet, I recommend checking out my previous article for detailed installation instructions:
  - [Mastering Docker: A Comprehensive Guide to Containerization](https://www.linkedin.com/pulse/mastering-docker-comprehensive-guide-containerization-ilyas-ou-sbaa?trackingId=%2BItTN8x5zCNo19tVsIppUQ%3D%3D&lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base_recent_activity_content_view%3BmqQWRnbGTEScP4nN3zLsHA%3D%3D)

Please note that although it's possible to install BIND9 on a Linux server without Docker, this tutorial focuses on Dockerizing BIND9 for easy management and scalability.

## Dockerizing BIND9

Let's dive into the process of Dockerizing BIND9. We'll cover the necessary steps to create a Docker Compose configuration and configure BIND9 inside the Docker container.

### Step 1: For ubuntu

Begin by editing the systemd-resolved configuration file located at `/etc/systemd/resolved.conf`. Uncomment the line `DNSStubListener` and set it to `no`. This change ensures that BIND9 can properly listen on port 53.

```conf
[Resolve]
...
DNSStubListener=no
...
```

After saving the changes, restart the systemd-resolved service by executing `sudo systemctl restart systemd-resolved`.

### Step 2: Create the Docker Compose File

Next, create a `docker-compose.yml` file in your project directory. Ensure that you replace the `container_name` value with your desired container name. Here's an example of the `docker-compose.yml` file:

```yaml
version: '3'

services:
  bind9:
    image: ubuntu/bind9
    container_name: bind9
    environment:
      - BIND9_USER=root
      - TZ=Africa/Casablanca
    ports:
      - "53:53"
      - "53:53/udp"
    volumes:
      - ./config:/etc/bind
      - ./cache:/var/cache/bind
      - ./records:/var/lib/bind
    restart: unless-stopped
```

Ensure that you save the `docker-compose.yml` file after making the necessary changes.

### Step 3: Create the Main Config File

In your project directory, create a `./config/` folder and copy the example `named.conf` file into it. Replace the values in the `named.conf` file with your desired configuration. Here's an example of the `named.conf` file:

```conf
acl internal {
  172.18.0.0/24;
};

options {
  forwarders {
    1.1.1.3;
    1.0.0.2;
  };
  allow-query { internal; };
};

zone "grandzero.io" IN {
  type master;
  file "/etc/bind/grandzero.io.zone";
};
```

Ensure that you save the `named.conf` file with your modifications.

## Cloudflar DNS quick review

If you're looking for enhanced protection against malware and adult content, Cloudflare offers DNS options specifically designed for these purposes. By changing your router's DNS settings, you can enable additional layers of security for your network.

### Malware Blocking Only

Change your router DNS to:

- 1.1.1.2
- 1.0.0.2

### Malware and Adult Content Blocking Together

Change your router DNS to:

- 1.1.1.3
- 1.0.0.3

### Step 4: Prepare the Zone File

In the `./config/` folder of your project directory, create a file. Copy the example `grandzero.io.zone` file and replace the values with your desired configuration. Here's an example of the demo-grandzero-de.zone file:

```conf
$TTL 2d

$ORIGIN grandzero.io.

@             IN     SOA    grandzero.io.  ns.grandzero.io. (
                            2022052603     ; serial
                            12h            ; refresh
                            15m            ; retry
                            3w             ; expire
                            2h )           ; minimum TTL
;
@   IN  NS  ns.grandzero.io.
ns  IN  A   192.168.8.146
grandzero.io.   IN  A  192.168.8.146
```

Save the `grandzero.io.zone` file with your desired configuration.

### Step 5: Add Your DNS Records

You can add additional DNS records based on your requirements and the examples provided. Refer to the [IANA's DNS Resource Records TYPEs](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-4) for more information on defining DNS records.

### Step 6: Start the Container

To start the BIND9 container, navigate to the project directory in your terminal and execute the following command:

```sh
docker-compose up -d
```

The container will start, and BIND9 will be up and running within the Docker environment.

## Testing Bind9

To test your BIND9 DNS server, you can use the `nslookup` command on your local machine. Replace `name-to-resolve.tld` with the domain or hostname you want to resolve and `your-dns-server-ip` with the IP address of your DNS server.

```sh
nslookup grandzero.io 192.168.8.146
```

Output

```
➜ nslookup grandzero.io 192.168.8.146
Server:		192.168.8.146
Address:	192.168.8.146#53

Name:	grandzero.io
Address: 192.168.8.146
```

## DNS Lookup Chain

Understanding the DNS lookup chain is crucial for troubleshooting and optimizing your DNS infrastructure. 

The DNS lookup process involves several steps, which can be summarized as follows:

1. Local DNS Cache: When you attempt to resolve a domain name, your operating system checks its local DNS cache first. If the domain name and corresponding IP address are already cached, the lookup process ends here.
2. Recursive Resolver: If the domain name is not found in the local DNS cache, the next step is to query a recursive resolver. This resolver is typically provided by your ISP (Internet Service Provider) or a third-party DNS resolver such as Google DNS or Cloudflare DNS. The recursive resolver handles the DNS resolution process on your behalf.
3. Root Nameservers: If the recursive resolver doesn't have the requested domain name in its cache, it starts the resolution process by querying the root nameservers. The root nameservers are the top-level DNS servers that maintain information about the root zone of the DNS hierarchy. There are 13 sets of root nameservers distributed worldwide.
4. Top-Level Domain (TLD) Nameservers: The root nameservers respond to the recursive resolver with a referral to the appropriate TLD nameservers based on the requested domain extension (.com, .org, .net, etc.). The TLD nameservers are responsible for managing the DNS records for their respective TLDs.
5. Authoritative Nameservers: The recursive resolver then queries the authoritative nameservers for the specific domain being resolved. The authoritative nameservers are responsible for storing the DNS records for the corresponding domain.
6. DNS Records: The authoritative nameservers respond to the recursive resolver with the requested DNS records, such as A records (mapping domain names to IP addresses), MX records (specifying mail server information), or other types of records.
7. Response to Client: Finally, the recursive resolver receives the DNS records from the authoritative nameservers and returns the response to the client. The client's operating system can then use the obtained IP address to establish a connection with the desired server.

This process typically occurs within milliseconds, and subsequent requests may benefit from caching at various levels to reduce lookup time.

## References

- [Bind9 Configuration and Zone Files](https://bind9.readthedocs.io/en/v9_18_10/chapter3.html)
- [IANA's DNS Resource Records TYPEs](https://www.iana.org
