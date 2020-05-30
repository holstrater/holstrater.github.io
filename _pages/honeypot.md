---
layout: archive
permalink: /honeypot/
author_profile: true
header:
  image: "/images/banner2.png"
---

# Using a honeypot to monitor unauthorized access

A honeypot is a computer (system) that's primarily meant to mirror common targets of cyberattacks. It can be designed to look and feel almost exactly like an actual production environment, luring in potential attackers. Attempts to gain unauthorized access to it are usually closely monitored and logged for further analysis. Post-access activity on the system can also be monitored and logged.

I've known about honeypots for a while and never really got around to setting one up until recently. I always thought the idea behind them was very cool and it seemed like there's a lot that can be learned from setting one up. I went with [Cowrie](https://github.com/cowrie/cowrie), an open source medium to high interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker.

Since the [documentation](https://cowrie.readthedocs.io/en/latest/index.html) on Cowrie is already outstanding, I won't bother with details on how I installed it. I did make some configuration changes, though:

* I changed the TCP port that the honeypot listens on to 2222
* I made sure that my firewall forwarded TCP traffic from 22 to 2222, so that attackers connecting to 22 would still be connected to the honeypot
* I edited `/home/cowrie/cowrie/etc/userdb.txt` to contain a list of custom credentials that would grant an attacker access to the honeypot environment. I denied certain extremely weak username-password combinations while allowing others, hopefully to decrease the chances of the attackers becoming suspicious.
* I changed the honeypots hostname to `home` instead of the default `svr04`
* I installed VirusTotal's CLI tool and signed up for an API key. This allows me to easily use VirusTotal for malware analysis whenever an attacker downloads something to the honeypot.

At this point I want to note that this honeypot, its configuration and even this website post are all works in progress. The idea behind this honeypot to me is create a continuous loop: keeping an eye on any activity and malware, reporting about it, tweaking configurations and improving monitoring, etc. This means that I'll be revisiting this post a lot and posting updates. That being said, feel free to take a look around. It's on 136.144.226.39 ;)
