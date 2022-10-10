# A sample WireGuard setup
This directory contains the WireGuard configuration files for the 3 VMs
making up or sample scenario.

It's crucial to note **you MUST NOT use these under any circumstance**. The
private keys guaranteeing WireGuard's security haven't been deleted. If you
have them, others (beginning with me) do too. If you want to base a WireGuard
deployment on this one be sure to use the following to generate the necessary
keys:

    # Generate private keys
    $ wg genkey

    # Generate a public key from a private one. Be sure NOT to
        # include your password 'as-is' on the command if you're
        # using an insecure machine. Also, be sure to clear the
        # key from your shell's history file afterwards...
    $ echo 'public-key' | wg pubkey

    # Generate symmetric keys
    $ wg genpsk

As the names imply, each file will be copied into `/etc/wireguard/wg0.conf`. The
copying itself is handled by `vagrant` when provisioning the machines as defined
in `../Vagrantfile`.

    core.conf     -> vpn-core:/etc/wireguard/wg0.conf
    client-a.conf -> client-a:/etc/wireguard/wg0.conf
    client-b.conf -> client-b:/etc/wireguard/wg0.conf

Be sure to check [`wg-quick(8)`](https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8)
for information on the configuration syntax. You can also find a deeper discussion on
[`wg(8)`](https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8). The configuration files also
contain comments where things get a bit trickier...
