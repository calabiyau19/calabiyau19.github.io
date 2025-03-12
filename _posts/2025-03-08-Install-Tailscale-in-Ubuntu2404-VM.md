---
layout: post
title: "Add Tailscale to Ubuntu 24.04 VM"
date:  2025-03-08
---

### How to add Tailscale to a Ubuntu VM

Scenario: Ubuntu 24.04 server running in a VM on a proxmox host

#### Add Tailscale's GPG key

{% highlight ruby %}
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
{% endhighlight %}

#### Add the tailscale repository

{% highlight ruby %}
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
{% endhighlight %}

#### Install Tailscale

{% highlight ruby %}
sudo apt-get update && sudo apt-get install tailscale
{% endhighlight %}

#### Start Tailscale!

{% highlight ruby %}
sudo tailscale up
{% endhighlight %}

#### Login to Tailscale account and accept connection
##### Done