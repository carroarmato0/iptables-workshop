# iptables Workshop

This repo containes a Vagrant file for a Centos 7 VM with some services running, albeit with a set of very draconian iptables rules.

Only incoming SSH and Ping (together with the rules for replying) are allowed to make logging in, and testing connectivity working.


The goal is to write necessary iptables rules to allow access to the various services running inside the VM.
