+++
title = "DNS-level adblock on the go with blocky"
date = 2024-05-31
draft = false

[taxonomies]
categories = ["Self-hosting"]
tags = ["blocky", "adblock", "headscale", "tailscale"]

[extra]
lang = "en"
toc = true
comment = true
copy = true
math = false
mermaid = false
+++

# Introduction
An adblocker is something you commonly find installed in browsers, usually through an extension. However, what if you wanted an adblocking system that was a bit deeper? Something that doesn't require a browser extension, and gives you a consistent adblocking system? This is where DNS-level adblockers like [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) or [Pi-hole](https://pi-hole.net) come in.

# DNS?
DNS stands for **D**omain **N**ame **S**ystem. It's what points URLs like `https://duck.com` to an IP address (like `52.142.124.215`), making it much easier to find things on the internet.

DNS-level adblockers work by filtering out queries for URLs pointing to IP addresses serving ads. In this blog post, I'll use [blocky](https://0xerr0r.github.io/blocky) as an example of one such adblocker for demonstration purposes.

# Setting up blocky
## NixOS configuration
There's a configuration option for `blocky` provided by NixOS, so you can enable and configure it in your NixOS config:

```nix,linenos
services.blocky = {
  enable = true;
  settings = {
    prometheus.enable = true;
    blocking = {
      blackLists.ads = [
        "https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts"
        "https://sysctl.org/cameleon/hosts"
        "https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt"
        "https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt"
      ];
      clientGroupsBlock = {
        default = [ "ads" ];
      };
    };
    upstreams = {
      groups.default = [
        "9.9.9.9"
        "1.1.1.1"
      ];
    };
    ports = {
      dns = "0.0.0.0:53";
    };
  };
};
```

{% alert(header="Why isn't it running?") %}
You might need to reboot after running a `nixos-rebuild switch`, or move/kill any process running on port `53` for this to work.
{% end %}

{% note(header="Custom DNS mapping") %}
You can use blocky to map a domain of your choice to an IP of your choice - refer to the [documentation](https://0xerr0r.github.io/blocky/latest/configuration/) for more information.
{% end %}

Here, I've used two upstream nameservers for blocky to forward valid DNS requests to (since blocky doesn't do any DNS resolution itself - except for custom mapping, detailed later). One is [Cloudflare's DNS](https://one.one.one.one) (`1.1.1.1`) and the other is [Quad9](https://quad9.net) (`9.9.9.9`).

As indicated by lines 6 through 11, you need to add lists containing URLs you want to be filtered from your DNS requests.

## Making it work everywhere
The thing is, you'll need to set the IP address of the machine running blocky as a nameserver for all of your workstations - it just won't recieve any requests otherwise, so it won't be *doing* any adblocking if you don't do this.

Of course, if you've got a router worth keeping around, you should be able to set a network-wide DNS resolver, and you can point this to your blocky-running machine in your router's settings.

However, what if you wanted to have this work everywhere you go, perhaps on a portable laptop? Well, if you're using [Tailscale](https://tailscale.com) or [Headscale](https://headscale.net) you can just can edit the nameservers you use in your VPN's settings and set it to the *IP address* of the device running blocky. This way, any device on your VPN can utilise blocky and have a functioning DNS-level adblocker no matter where you are.

If you're looking to setup headscale, I've made [a blog post about it](../selfhost-tailscale).

# Finishing thoughts
With the existence of browser extensions doing the same thing adblockers like blocky and Pi-hole can, not everyone is going to need something this sophisticated. I think something like this is better suited to those looking for better coverage in their adblocking, or something that gives more control over DNS requests - for example, to easily setup custom DNS mappings or to restrict access to certain websites.
