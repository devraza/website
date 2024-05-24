+++
title = "Home server security"
date = 2024-03-28
draft = false

[taxonomies]
categories = ["Cybersecurity & Privacy"]
tags = ["homelab", "hardening", "selfhosted"]

[extra]
lang = "en"
toc = true
comment = true
copy = true
math = false
mermaid = false
+++

# Introduction

Home server security is pretty often overlooked from what I can tell.
Any device accessible from the internet has *some* degree of
vulnerability in the current era of the internet. I aim for this
document to detail methods to amend the contemporary cybersecurity
challenges faced by most homelabbers.

# Justification in Depth

Of course, my statements about home servers needing some security
measures put in place aren't baseless. My own experience, as well as
that of a sizable number of people on the wonderful
[lemmy](https://join-lemmy.org) community at
`selfhosted@lemmy.world` shows that home servers are endlessly 'knocked"
on, and that login attempts to services like SSH *are* made. Here's a
snippet from my [fail2ban](https://fail2ban.org) filter to
verify this point:

```
Mar 29 14:38:13 icefall fail2ban.filter[1097]: INFO [...] Found 176.126.240.158 - 2024-03-29 14:38:13
Mar 29 14:40:11 icefall fail2ban.filter[1097]: INFO [...] Found 176.126.240.158 - 2024-03-29 14:40:11
Mar 29 14:40:29 icefall fail2ban.filter[1097]: INFO [...] Found 185.8.165.204 - 2024-03-29 14:40:29
Mar 29 14:40:40 icefall fail2ban.filter[1097]: INFO [...] Found 162.212.154.58 - 2024-03-29 14:40:40
```

Within the past *few minutes*, I've already got a few IP addresses from
all over the world taking a peak at my services. If I had my SSH port
set to the standard `22`, I could have expected a few rogue login
attempts to have been made, too.

And, speaking of not having my SSH port set to the standard `22`, I'll
now move on to what you should be done to secure a home server. One
thing that I think should be noted, however, is that security doesn't
need to be very strong, and you generally don't need to go too far out
of your way with security measures (though this definitely depends on
invdividual circumstance). Honestly speaking, you *probably* **don't**
have competent black hats looking to get in to your server - what you
probably **do** have, however, are a bunch of script kiddies and
perversive bots.

# The list

The fairly basic stuff you'd need to do in this case doesn't make much
room for detail. So, here it all is in the form of a simple list (I've
included the relevant NixOS configuration where I think it'd be
useful[^1]):

-   Move your SSH daemon to a non-default port, like `3291`.
    ```nix
    services.openssh = {
      ports = [ 3291 ]; # whatever you like
    };
    ```

-   Force public key authentication with SSH and disable root logins.
    ```nix
    services.openssh.settings = {
      PermitRootLogin = "no";
      PasswordAuthentication = false;
    };
    ```

-   Set up a pretty basic firewall - something like `ufw` would do the trick.
    ```nix
    networking = {
      nftables.enable = true; # use the newer nftables
      firewall = {
        enable = true;
        rejectPackets = true; # explicit deny
        interfaces.enp1s0 = { # obviously, replace `enp1s0` with your interface
          allowedTCPPorts = [ ... ]; # put in the ports you need here
        };
      };
    };
    ```

-   This probably doesn't need to be said, but **use strong passwords**!

-   Host a [fail2ban](https://fail2ban.org) instance to ban hosts making bruteforce attempts.

I think that's all there is for almost everyone, and is basically the
minimal amount of effort a home server administrator should do.
Personally, I would prefer to enforce a VPN connection in order to
access my *personal* services for that extra layer of security (because
why'd they need to be exposed to the internet?). This can be done faily
easily with tailscale, and for the slightly more paranoid -
[headscale](https://headscale.net/) is a
viable...alternative? Anyways, I've got a blog post that explores
headscale in a little more detail, which might be worth checking out.

Well, that's all I wanted to say. It's been a while since my last blog
post, and the inspiration for this one came seemingly randomly - I hope
someone finds this useful.

[^1]: Naturally, you shouldn't just copy and paste the snippets into your own config. Do your research first!
 
