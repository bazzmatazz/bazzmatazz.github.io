---
layout: post
title:  "Consul With DNS Forwarding Using Ansible"
date:   2020-06-10 05:12:13 +0500
categories: 
---

For a scalable, decoupled infrastructure, we prefer using microservices to serve different business requirements. However, for complex business requirements, this means there could be quite a few microservices, and each service would need to communicate with other services. Since manually sharing addresses is not scalable, we want to automate the service discovery process using appplications like **Consul**.

We run consul servers and clients in a single datacentre and these agents communicate with each other using gossip protocol. Services running in this datacentre can then use consul to discover other services through an HTTP API or DNS.

![Flashing](/images/consul-highlevel.jpg)

While HTTP API gives us more control, sevice discovery through DNS helps us avoid a tight coupling with consul.

**Setup**
I am using 2 virtual box machines (ubuntu 18) and my host machine to form a three node datacentre. One of the virtual box machines has the consul server, while other 2 machines host consul clients.

**Consul Server**

Consul is setup using its latest docker image:
`docker run --network=host -it consul agent -node server -server -datacenter dc1 -advertise 192.168.100.4 -client 0.0.0.0 -bootstrap-expect 1 -enable-script-checks -ui`

`-advertise` is used to specify the IP of my server (in this case 192.168.100.4). This is required when there are multiple interfaces live and we need to specify which one is part of the relevant network.

`-bootstrap-expect` is set as 1 because we only have 1 server (this is not recommended in a production environment. Should have 3-5). For now setting this would mean we don't have to go through the process of electing a leader.

**Consul Client**
The next step is running the clients which can join the cluster

`docker run --network=host -it consul agent -datacenter dc1 -client 0.0.0.0 -bind '{{ GetPrivateInterfaces | include "network" "192.168.100.0/24" | attr "address" }}' -join 192.168.100.4 -enable-script-checks -ui`

And there, we are up and running. Now we would like to register the services we want to be discovered

**Registering Services**

I'll start by registering my two flask apps:
flapp1 running on port 5000
flapp2 running on port 5001

flapp1.json
```
{
  "service": {
    "name": "flapp1",
    "tags": [
      "flask-app"
    ],
    "port": 5000
  }
}
```

flapp2.json
```
{
  "service": {
    "name": "flapp2",
    "tags": [
      "flask-app"
    ],
    "port": 5001
  }
}
```
`docker exec consul consul services register flapp1.json`
`docker exec consul consul services register flapp2.json`

If not specified otherwise, consul would assign these services addresses:
`flapp1.service.consul`
`flapp2.service.consul`

**Discovering The Services Over DNS**

Since by consul DNS by default handles request on port 8600, we can check the address resolution for flapp1 with:
`dig -p 8600 flapp1.service.consul`

The answer section shows the IP address of the running service, and we can see the port and host information by asking for SRV record
`dig -p 8600 flapp1.service.consul SRV`

**DNS Forwarding**
We come gotten pretty far but our services still can't communicate with each other. Since the addresses are by default resolved on port 53, we can only get this working if consul can return the same information on port 53.

A simple excercise would be to run consul on port 53 but a much better option would be to forward all consul related inquiries to port 8600.

We can do this by using **DNSMasq**.

1. _Install DNSMASQ_
`sudo apt-get install dnsmasq`

2. _Run DNSMASQ on 127.0.0.2_
- Update `/etc/dnsmasq.conf` to listen on 127.0.0.2 by adding the line:
`listen-address=127.0.0.2`
- Next we want all queries for addresses with consul in it to go to consul DNS
`server=/consul/127.0.0.1#8600`


3. _Make DNSMasq the primary DNS_
- Update `/etc/systemd/resolved.conf` by setting DNS equal to 127.0.0.2
```
[Resolve]
DNS=127.0.0.2
#FallbackDNS=8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
#Domains=
#LLMNR=yes
#DNSSEC=no
```

4. Restart dnsmasq and systemd-resolve
- `sudo systemctl restart dnsmasq`
- `sudo systemctl restart systemd-resolved`

**Get a cup of coffee**
`dig flapp1.service.consul` should now resolve correctly and you can access your service on:
`http://flapp1.service.consul:5000`

### Debugging
- Verify DNSMASQ is being used as your primary server:
`sudo systemd-resolve --status`

- Check DNSMasq logs
  - Uncomment `log-queries` in dnsmasq.conf
  - Use journalctl to check the logs:
  `journalctl -u dnsmasq.service -f`

- Listen on port 53
`sudo tcpdump -vvv -s 0 -l -n -i tun0 port 53`