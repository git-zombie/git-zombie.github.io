---
layout: post
title: Traefik
published: true
---

To quote one of the great code-wizards I know, ‘Traefik is magic.’
![traefik.logo.png](./traefik.logo.png)

Traefik is self-described as “a modern HTTP reverse proxy and load balancer made to deploy microservices with ease.”  I’d rather call it ‘a way better alternative to nginx that also solved my SSL certificate headache.’

I’ve been wanting to expose some of my internal web services for a while now, but using nginx for a reverse proxy was a nightmare, and my services ranged from self-signed certificates, no HTTPS support at all, or bring-your-own-certs. My friend Jon and I had messed with nginx and Let’s Encrypt, but having to completely re-implement everything for each service was more trouble than it was worth.

Enter Traefik.

I mostly followed their documentation here to implement Let’s Encrypt. If you aren’t familiar with it, Let’s Encrypt is a free, open Certificate Authority, certifying the cryptography keys needed to see that green lock next to the URL in your browser that you get the warm fuzzies from. You do check to see if the lock is there everytime you log into your bank, right?

Setting up Traefik was pretty easy.

Following the directions, it’s pretty much making a directory, creating three files, and then turning it on.

The few changes I made from the documentation involved turning on the Traefik dashboard and accommodating my unusual network topology.

Let’s look at the docker-compose yaml file.


    [user@host ~]$ cat traefik/docker-compose.yml
    version: '2'
    services:
    traefik:
    image: traefik
    restart: always
    ports:
    - 80:80
    - 443:443
    - 8080:8080
          networks:
            network1:
            network2:
              ipv4_address: 192.168.1.2
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - /home/user/traefik/traefik.toml:/traefik.toml
    - /home/user/traefik/acme.json:/acme.json
    container_name: traefik
          labels:
          - "traefik.frontend.rule=Host:monitor.gsp.cloud"
          - "traefik.port=8080"
    networks:
      iot:
        external: true
      usenet:
        external: true

These changes accomplished two things:

The monitoring dashboard, available at port 8080 has been exposed, but not actually made available. I can check it, but only inside the LAN. This can be helpful for confirming route availability.
I also set Traefik up to span across two different Docker network interfaces, one of which is a bridge and the other is a macvlan interface.

> [user@host ~]$ cat traefik/traefik.toml debug = false checkNewVersion
> = true logLevel = "ERROR" defaultEntryPoints = ["https","http"] InsecureSkipVerify = true [entryPoints] [entryPoints.http] address =
> ":80" [entryPoints.http.redirect] entryPoint = "https"
> [entryPoints.https] address = ":443" [entryPoints.https.tls] [retry]
> [docker] endpoint = "unix:///var/run/docker.sock"   domain =
> "gsp.cloud" watch = true exposedbydefault = false [acme]   email =
> "myemail@domain.com" storage = "acme.json" entryPoint = "https"
> OnHostRule = true [web]   address = ":8080" [web.auth.basic]   users =
> ["admin:asdfasdfasdfasdfasdf"]

Past the documentation, there’s only a couple things going on here.

I’m setting my email and domain name.
I’m configuring the dashboard/GUI for Traefik itself
I also turned on the InsecureSkipVerify flag to accomodate the Unifi SDN Controller that uses a self-signed cert. I’ll cover that in a later post.
At this point, it’s just a simple docker-compose up -d and we’re up and running!

At the completion of this, I have Traefik stable, running and talking to Let’s Encrypt, but it’s not proxying/load-balancing any applications yet. I can check the dashboard, but it won’t be very interesting yet. However, Traefik is now monitoring the docker socket for any containers indicating their desire to work with it. Traefik is now stable, and new microservices can be brought up without reconfiguring it or bringing it down again.

Until the next post…
