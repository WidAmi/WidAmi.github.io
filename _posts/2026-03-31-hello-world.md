---
layout: post
title: "Vibe Coded WiFi Monitor"
date: 2026-03-31
---

So this is something I had been kicking around in my head for a while but was able to bring to life
in an evening using Claude Code. I did some fine tuning later on, but the bulk of this was built
in just a few hours after dinner.

As I mentioned, I'd been thinking about this for a while so I had some idea of what I wanted it to
look like. Basically a few dashboards. One showing overall status of the WiFi, one for showing details
about any particular AP, and one for showing details of any single client.

As background, my WiFi network consists of four Netgear Orbi tri-band access points which have had
OpenWRT installed on them. The 2.4GHz radio serves clients, one 5GHz also serves clients, and one 5GHz
acts as the mesh backhaul. One AP (orbi1 by name) has a wired connection to the router. A couple of the APs
have wired clients as well (each Orbi has 4x wired ethernet ports).

To get the data out of the APs there is a collection script written in Python which is run by systemd every
two mintes. It ssh's to each AP and runs a bunch of commnands to get the stats. Most of the research on what
commands to run was done by Claude Code. It's pretty good at figuring out all that stuff. I'd just tell it
what numbers I considered important and it would figure out what commands needed to be run on the APs to
pull that information out. Doing this was pretty iterative and interactive.

Once that was done CC was able to get the initial version without much help from me. Install Docker. Make
a docker-compose. Get InfluxDB and Grafana running. Get systemd to run the collection script on a timer.
We did get stuck on that piece for a while because the collection script was running as root and couldn't
find the right SSH key to log into the Orbi's. But once that was done we had data going into Influx.

Again, I had Claude implement the dashboards in Grafana. That was back to a pretty iteractive mode. I'd
tell it what to add, or change, etc and it could just get it done. Since then I've had it make a couple
small changes to the dashboards but most of it was done that first night.

Is this a fancy project? Not really. Most of it is pretty trivial. The collection script has a little bit
of sophistication, as do the dashboard configs. It's not architected in a way that lets it be generally
useful. Some of the things like how many AP's you have and what type they are are kind of baked in more
than they should be. But hey, just vibe code up your own version!
