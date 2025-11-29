---
title: "Turning My Old PC Into A Homelab"
date: 2025-11-29T14:00:00-08:00
draft: false
tags: ["homelab", "docker", "ubuntu", "infrastructure", "self-hosting"]
---

I've had this weird itch to boot my old PC for a while now. This old dusty tower sitting under the desk that used to be important to me. I wanted to jump into the past for a second. So I plugged in a monitor, hit the power button, and the damn thing greeted me with GRUB errors and random leftover partitions that didn't mean anything anymore. It wouldn't boot Windows. It wouldn't boot Linux. So I just stared at it and said screw it. I'll wipe everything and give it a new life. This machine can be a homelab. A place for me to self host things, learn things, and use it as the heavy lifter so my main devices stay free to actually develop.

<!--more-->

## Starting Fresh

I downloaded Ubuntu Server, flashed the ISO to a USB, went into BIOS, disabled Secure Boot, made sure UEFI was on, set the boot order to USB, and the installer came up. It showed me the Patriot SSD full of Windows trash, a big 1 TB hard drive, and whatever stray partitions were left. I wasn't going to salvage anything. I wiped the SSD and let Ubuntu take over completely. During installation I enabled OpenSSH because I wanted this thing headless from day one. I also installed some extra tools I knew I'd eventually want, like Wormhole for quick file transfers, Google Cloud SDK, AWS CLI, Prometheus, and a couple other utilities useful in a personal cloud. Once it rebooted, it dropped me into a fresh terminal. That's when the PC officially became a server.

![Homelab System Info](/images/homelab-neofetch.png)

## SSH Setup and Remote Access

On my Mac, I generated SSH keys using `ssh-keygen`. That creates a private key on my system and a public key I can safely put on the server. I added an entry to `~/.ssh/config` so I don't have to type long commands. Something like:

```
Host monolith
HostName 10.0.0.34
User bitterfq
```

Now I can just type `ssh monolith`. Then I ran `ssh-copy-id monolith` so my public key gets added to the server's `authorized_keys` file. After that, the box didn't need a monitor anymore. It didn't need a keyboard. It lived entirely in my terminal.

## Docker and Remote Context

Once I was inside Ubuntu, I updated packages and installed Docker properly. I added the Docker GPG key, added the official Docker repo, installed `docker-ce`, `docker-compose`, the whole thing. Then I added myself to the docker group with `usermod` so I don't need sudo for every command. Logged out and back in. Docker was ready.

The whole point of all this was simple. I wanted my laptop to handle development, and I wanted the server to handle compute. So I created a Docker context on my Mac, something like:

```bash
docker context create monolith --docker "host=ssh://bitterfq@10.0.0.34"
docker context use monolith
```

And just like that, `docker ps` on my Mac shows containers running on my server. `docker run postgres:16` on my Mac actually launches a Postgres container on the server. My laptop does basically nothing. The server does all the actual work. It's the cleanest feeling ever because you realize your development machine doesn't need to run heavy services anymore. You offloaded the entire workload into a box in the corner of your room.

## Portainer for Container Management

Once Docker was working remotely, I installed Portainer because visual dashboards make your life easier. On the server I created a volume with `docker volume create portainer_data` and then spun up Portainer CE:

```bash
docker run -d \
  -p 9000:9000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

I opened my browser, went to my server's IP on port 9000, set up the admin user, and suddenly everything was visible. Every container. Every volume. Every log. CPU usage. Memory usage. Networks. It felt like looking into the guts of my own little cloud.

![Portainer Dashboard](/images/homelab-portainer-dashboard.png)

## Uptime Monitoring with Uptime Kuma

Later on, I wanted uptime monitoring for all my services. That's where Uptime Kuma comes in. It's like a self-hosted Pingdom but way nicer. I installed it by running:

```bash
docker run -d \
  --restart=always \
  -p 3001:3001 \
  -v uptime-kuma:/app/data \
  --name uptime-kuma \
  louislam/uptime-kuma:1
```

Then I opened it in a browser and set it to monitor Portainer, my API server, OpenMetadata, Airflow, and whatever else I run. Now I have real visibility. If something crashes, freezes, or becomes unreachable, I know instantly.

![Uptime Kuma Monitoring Portainer](/images/homelab-uptime-kuma-portainer.png)

The monitoring integrates directly with Discord, so I get notifications when services go down or come back up:

![Discord Alerts](/images/homelab-discord-alerts.png)

I also made sure to have Wormhole installed on the server so I can shoot files into it instantly. And AWS CLI and Google SDK are sitting there for cloud interactions if I ever need them. All of this together makes the machine feel like a tiny datacenter running next to me.

## What's Next

So now I have a headless Ubuntu Server box running Docker with remote context configured from my Mac. Portainer gives me visual container management. Uptime Kuma monitors everything and alerts via Discord. SSH access means I never need a monitor. The whole setup acts as a compute offloader so my laptop stays clean for development work.

It sits there, humming quietly, running everything I throw at it. And it's kind of sick how much value you get out of an old machine once you repurpose it properly.

Next up I'm planning to run k3s for lightweight Kubernetes orchestration, spin up a Plex media server, and set up MinIO for object storage to support data engineering workloads. I've also got a couple Raspberry Pis that I'll add to the cluster for distributed compute. The foundation is there. Now it's just about stacking services on top of it.

If you ever want to learn infrastructure, or backend work, or just want a place to run services you actually use, this is how you do it. Turn an old PC into a server. Let your personal machine focus on development. Let the server carry the weight. And slowly you build your own ecosystem at home, completely under your control.
