+++
title = "Host your own private search engine with SearXNG"
date = 2023-12-31
draft = false

[taxonomies]
tags = ["searxng", "privacy", "selfhosted", "nixos"]

[extra]
lang = "en"
toc = true
comment = true
copy = true
math = false
mermaid = false
+++

# Introduction
[SearXNG](https://docs.searxng.org/), put in its own words, is a 'free internet metasearch engine'.
Note that it describes itself as a *metasearch* engine specifically - unlike your traditional search engine like Google or Bing, SearXNG does things a little bit differently:
It aggregrates the results produced by search services like those aforementioned, and feeds them back to you.

Because of this key detail and a great deal of effort by those who've helped shape it, SearXNG protects your privacy, and does so very well:
- Private data from requests going to the search services it aggregrates results from is removed
- It does **not** forward *anything* to any third parties through search services
- Private data is *also* removed from requests going to the results pages

Furthermore, SearXNG can be configured to use [Tor](https://torproject.org).

However, the aspect of privacy isn't the only great selling feature of the engine; from my use of the engine so far, it's also great at...searching (is that a surprise?). The fact that it's a metasearch engine plays a key role in this,
as it provides SearXNG the ability to pull content more efficiently and gives *you* the ability to further tailor your experience.

# Setting up SearXNG
## Installing the service
As you may have expected if you've used NixOS for a while, searxng is packaged *and* has a service on NixOS. This makes setting it up just that much easier.

To get started, place somewhere in your *system* config the following:
```nix
{
    # ...
    services.searx = {
    enable = true;
      settings = {
        server = {
          port = 8888;
          bind_address = "127.0.0.1";
          secret_key = "@SEARX_SECRET_KEY@";
          base_url = "https://search.devraza.duckdns.org/"; # replace with wherever you want to host yours
        };
      };
    };
    # ...
}
```

The snippet above starts the `searx` systemd service for listening on port `8888`, and assumes a `base_url` of `https://search.devraza.duckdns.org`.

Now that we've got the actual `searx` instance running, we can now set up a reverse proxy allowing the service to be accessed remotely (whether this is within your local network or across the internet is up to you).

## Setting up a reverse proxy
### What is a reverse proxy?
Before I get started with the technical details of setting this up, I'd like to briefly clarify what a reverse proxy exactly is (to my understanding).

Let's get the wikipedia definition of reverse proxy out of the way first:

> [...] a reverse proxy is an application that sits in front of back-end applications and forwards client requests to those applications. [...]

However, you might be confused as to what this actually means; I'll give an example of the usage of reverse proxies to better explain this:

- Suppose you've got a few services running on a server (for demonstration purposes, let's name these `x`, `y` and `z`), each running on their own unique port.
- Assuming you had a domain, and wanted to access all of these services from their own unique sub-domains (e.g. `x.yourdomain.com`, `y.yourdomain.com` and `z.yourdomain.com`), you would have to use a reverse proxy.
- This reverse proxy would take in requests from clients going to sub-domains, and forward these requests to the appropriate port on your machine for the service being requested.

The concept should be clear now, if it wasn't already.

### Using NGINX to set up the reverse proxy
NGINX is a popular web server that supports the creation of virtual hosts and the usage of reverse proxies. To accomodate our `searx` instance, we append the following to our NixOS server configuration:

```nix,hl_lines=12,linenos
{
  # ...
  services.nginx = {
    enable = true;
    # any extra configuration here
    virtualHosts = {
      "search" = { # this can be anything, being an arbitrary identifier
        forceSSL = true;
        serverName = "search.yourdomain.com"; # replace this with whatever you're serving from
        # SearX proxy
        locations."/" = {
          proxyPass = "http://${toString config.services.searx.settings.server.bind_address}:${toString config.services.searx.settings.server.port}";
          proxyWebsockets = true;
          recommendedProxySettings = true;
        };
      };
    };
  };
  # ...
}
```

{% note(header="Note") %}
The expression highlighted above is used to dynamically adjust the location NGINX will forward requests to, depending on your `searx` config
{% end %}

After saving your changes and rebuilding your server's system configuration (as usual), you should have a working *private* instance of SearXNG that you can access using the `serverName` you've given it.

Set your browser to use this as your search engine using the relevant documentation (with Firefox this is as easy as right-clicking on the URL after opening up the page and clicking a button). Enjoy!
