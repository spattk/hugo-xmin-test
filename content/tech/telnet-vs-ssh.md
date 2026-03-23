---
title: "Telnet vs SSH"
date: "2024-12-15"
tags: ["networking", "ssh", "telnet", "devops", "backend"]
description: "What Telnet taught me about connectivity debugging that SSH never could — and why your manager might have been right all along."
---

When I was working at a small startup back in the day, there wouldn't always be enough subject matter experts (SMEs) to solve problems due to small dev teams. In most cases, the startup required the engineering manager to be highly technical.

Back then, we were manually deploying by building the Rust web server on the EC2 instance and restarting the application server for non-prod environments.

Once, there was an issue where we were not able to SSH into the machine, and my manager quickly joined the call and fired a Telnet command first to check the connectivity. I couldn't understand why he preferred Telnet over SSH — maybe it was just what he was familiar with, I thought.

---

Fast forward to today, I was checking a couple of things around DNS resolution and had to use Telnet to check connectivity — and it connected the dots for me.

## What makes Telnet useful?

- Telnet is a command-line utility, like other tools such as `curl` or `ssh`
- The best thing about Telnet is that you can **specify the port** to check whether the server responds on it — helping establish the first point of connectivity
- To check if a MySQL server is responsive: `telnet root@mysql-server-ip 3306`
- To check if SSH is reachable: `telnet root@server-ip 23`

## Why not just use curl?

You might be thinking — we can use `curl` too, right?

Not really. Unlike Telnet, curl doesn't have a direct way to make a raw TCP connection request. It works mostly for HTTP/HTTPS, meaning it strictly targets ports 80 or 443. If those ports aren't accessible, you can't clearly tell whether the connectivity is broken or the port is simply inaccessible.

There are ways to use curl with Telnet — `curl telnet://192.168.0.1:8082` — but that's more verbose and less intuitive for quick connectivity checks.

---

So when my manager fired Telnet before SSH, he wasn't just going with what was familiar. He was checking whether the machine was reachable at all before assuming it was an SSH-specific problem. A small but important distinction when you're debugging under pressure.
