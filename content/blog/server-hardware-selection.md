+++
title = "Selecting hardware for a server"
date = 2024-01-31
draft = false

[taxonomies]
categories = ["Self-hosting"]
tags = ["selfhosted", "home lab", "sysadmin", "hardware"]

[extra]
lang = "en"
toc = true
comment = true
copy = true
math = false
mermaid = false
+++

# Introduction
  I see a lot of people worryingly mistaken about what a server needs (specifically, a home server). Some think that a bland and incredibly ignorant '20% budget for CPU, 30% for GPU, and the rest for the rest' plan for selecting hardware is good enough (at least, before choosing specific items) - you can't exactly be *wrong* when choosing
  hardware, but this is very, *very* far from right.

# Requirements
  Server hardware needs to be low-power and resource-efficient - so as not to waste any money unnessarily, obviously. Your budget will strongly impact the specifications you can get your hands on, but I would think that even $200 is enough for a decent home server - depending on what you want to do with it, obviously.
  Keep in mind that you could always repurpose an old laptop or desktop lying around; it's cheap, and you get what may be a surprisingly decent machine.

## Power consumption
   The difference between low *peak* and low *idle* power should be noted in particular. Running costs can get very high if you don't work to moderate power consumption, and probably wouldn't be something you would ignore when it comes to home servers.

## Usage of the server
   Naturally, how a server will be used will affect pretty much everything about the hardware chosen for it. For example, if you're looking to stream games remotely, you'd go for a (perhaps powerful) dedicated GPU and would likely invest in some high-speed internet solution.
   As indicated by the above example of the GPU, you need to be *very* specific with what you choose - do you *need* a powerful GPU or do you not? After all, one of the last things you would want, ever, is money going to waste on something you don't need, or failing to buy something that meets your expectations.

# Conclusion
  I only gave two point of interest when selecting a server - looking at the numbers alone, this might seem like hardly anything to consider at all. My reasons for this are:

  - Power comumption is one factor that people often forego thinking about, and an extremely important one at that.
  - Building on top of the previous point, you (probably) aren't stupid. Knowing (albeit at a very basic level) what to look out for should be enough.
  - It would be extremely difficult for me, or anyone else for that matter, to provide a **truly** complete solution to everyone's needs for a home server.

  Well, that's it. I wish you luck in selecting your hardware.
