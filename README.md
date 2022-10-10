# Security, access control and traffic shaping though L3 VPNs
This repository contains several goodies we leveraged to illustrate the concepts set forth in
the our talk at the XVII REDIMadrid conference.

We decided to take a deep look at how we can make use of WireGuard, a L3 VPN, as a solution
offering security and access control for large scale networks in an efficient and escalable
fashion.

Given how WireGuard integrates with the Linux kernel, we also decided to investigate how we
could apply QoS techniques on the traffic being "tunelled" through the VPN with the help
of [`tc(8)`](https://man7.org/linux/man-pages/man8/tc.8.html).

We finished by delving into eBPF and how it can also play nicely with WireGuard to offer a
completely programmable packet handling framework taking advantage of the latest networking
technology the Linux kernel has to offer.

Let's take a (brief) look at these topics!

## First things first: Vagrant
All the examples are built with a virtual setup in mind. The setup itself consists of 3
[VirtualBox](https://www.virtualbox.org) Virtual Machines (VMs) managed through
[Vagrant](https://www.vagrantup.com). The configuration is embedded into the Vagrantfile
you'll find at the repository's root, and making use of it is as simple as running:

    vagrant up

The Vagrantfile should take care of everything from starting the machines to installing
necessary dependencies. It'll even deploy the WireGuard configuration we've included in
the repository!

Bear in mind we have tested this deployment on a machine running macOS. According to our
experience everything should be okay on Linux too, but we can't say the same about
Windows...

If you find any issues let us know by, well, opening an issue :P

For help on how to install VirtualBox and Vagrant refer to
[this](https://www.virtualbox.org/wiki/Downloads) and [that](https://www.vagrantup.com/downloads)
site, respectively.

## Checking everything works
At this point you should be able to `ssh` into a machine. Do so and try to ping the
others with address both belonging and not belonging to the VPN. You can use the
following as a guide to avoid loosing track of what you've checked and what not:

    # Log into the client-a machine
    $ vagrant ssh client-a

    # Try to ping the host machine
    vagrant@client-a:~$ ping -c 1 10.0.123.1

    # Try to ping vpn-core through the VM subnet
    vagrant@client-a:~$ ping -c 1 10.0.123.1

    # Try to ping client-b through the VM subnet
    vagrant@client-a:~$ ping -c 1 10.0.123.3

    # Try to ping vpn-core through the VPN
    vagrant@client-a:~$ ping -c 1 192.168.4.1

    # Try to ping client-b through the VPN
    vagrant@client-a:~$ ping -c 1 192.168.4.3

If all the above return a line along the lines of the following, you're all set!
Bear in mind `<pinged-ip>` is to be substituted by whatever IPv4 address you
`ping`ed with each command...

    64 bytes from <pinged-ip>: icmp_seq=1 ttl=63 time=1004 ms

## Playtime!
Now it's time to try out and break stuff. You can experiment with different 
[`iptables(8)`](https://man7.org/linux/man-pages/man8/iptables.8.html) rules and see
whether the behaviour is the one you expected, for example.

You can also give our examples a try! Be sure to take a look at the PDF file in the
repository's root to get some ideas on what you can do. We are including the `tc(8)`
commands we invoked to modify the delay of packets traversing the VPN, for instance.

### Adding delays with tc
In an effort to showcase how one can easily integrate `tc(8)` wit WireGuard, we decided
to leverage the `netem` (i.e. [`tc-netem(8)`](https://man7.org/linux/man-pages/man8/tc-netem.8.html))
*qdisc* to add a steady $500\ ms$ delay to packets traversing the `wg0` interface on `vpn-core`.

We achieved that with:

    # We begin by displaying the configured qdisc on wg0.
        # The noqueue qdisc implies none has been set up.
    vagrant@vpn-core:~$ tc -g -d -s qdisc show dev wg0
    qdisc noqueue 0: root refcnt 2 
     Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0) 
     backlog 0b 0p requeues 0

    # We can then add a 500000 us == 500 ms delay to packets by
        # configuring netem as the root qdisc for wg0
    vagrant@vpn-core:~$ sudo tc qdisc add dev wg0 root netem delay 500000

    # We can see the changes did indeed take effect. At this point you
        # can check packets traversing the VPN do indeed suffer an
        # additional 500 ms delay.
    vagrant@vpn-core:~$ tc -g -d -s qdisc show dev wg0
    qdisc netem 8003: root refcnt 2 limit 1000 delay 500ms
     Sent 1260 bytes 15 pkt (dropped 0, overlimits 0 requeues 0) 
     backlog 0b 0p requeues 0

    # When we're done, we can undo the changes by deleting the
        # netem qdisc. This will configure the noqueue qdisc again.
    vagrant@vpn-core:~$ sudo tc qdisc del dev wg0 root

### Inspecting and filtering packets with eBPF
We have included a small [eBPF](https://ebpf.io) program in the `wg_ebpf` directory
on the repository's root. The directory contains a `README.md` file with more information
on the topic, but we just want to point out how we can easily integrate WireGuard with
the eBPF ecosystem, thus opening up the door to a myriad of opportunities and applications!

## Conclusion
That's pretty much it from us...

If you find anything is wrong or you have any suggestions or feedback, please feel free to
drop us an email at the address shown on our profile. You can also open an issue in the
repository if that's your jam.

We hope you found our work interesting or, at least, amusing :)
