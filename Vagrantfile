$vpn_core_setup = <<~EOF
    apt update

    # Install needed dependencies:
        # wireguard: The WireGuard VPN
        # build-essential: Provides `make` for building the eBPF program.
        # clang: The LLVM-based compiler for the eBPF program.
        # llvm: Facilities leveraged by `clang` for eBPF program compilation.
    apt install -y wireguard build-essential clang llvm

    # Install Go. Refer to https://go.dev/doc/install for details.
    rm -rf /usr/local/go
    curl -LO https://go.dev/dl/go1.19.2.linux-amd64.tar.gz
    tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
    rm go1.19.2.linux-amd64.tar.gz
    echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile

    # Enable IPv4 forwarding and make it permanent
    sysctl -w net.ipv4.ip_forward=1
    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.d/99-enable-forwarding.conf

    # Create WireGuard's directory structure and copy the config over
    mkdir -p /etc/wireguard
    mv /tmp/wg0.conf /etc/wireguard/wg0.conf
    chmod 0600 /etc/wireguard/wg0.conf
EOF

$vpn_client_setup = <<~EOF
    # Install Wireguard
    apt update
    apt install -y wireguard

    # Create WireGuard's directory structure and copy the config over
    mkdir -p /etc/wireguard
    mv /tmp/wg0.conf /etc/wireguard/wg0.conf
    chmod 0600 /etc/wireguard/wg0.conf
EOF

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/jammy64"

    config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", 1024]
    end

    config.vm.define "vpn-core" do |vpn_core|
        vpn_core.vm.hostname = 'vpn-core'
        vpn_core.vm.network :private_network, ip: "10.0.123.2"

        # Take a look at https://www.vagrantup.com/docs/provisioning/file
        # Note this provisioner CANNOT run in privilged mode...
        vpn_core.vm.provision "file", source: "wg_conf/core.conf", destination: "/tmp/wg0.conf"

        # Take a look at https://www.vagrantup.com/docs/provisioning/shell
        vpn_core.vm.provision "shell", inline: $vpn_core_setup

        vpn_core.vm.provision "shell", inline: "wg-quick up wg0"
    end

    config.vm.define "client-a" do |client_a|
        client_a.vm.hostname = 'client-a'
        client_a.vm.network :private_network, ip: "10.0.123.3"

        client_a.vm.provision "file", source: "wg_conf/client-a.conf", destination: "/tmp/wg0.conf"

        client_a.vm.provision "shell", inline: $vpn_client_setup
        client_a.vm.provision "shell", inline: "wg-quick up wg0"
    end

    config.vm.define "client-b" do |client_b|
        client_b.vm.hostname = 'client-b'
        client_b.vm.network :private_network, ip: "10.0.123.4"

        client_b.vm.provision "file", source: "wg_conf/client-b.conf", destination: "/tmp/wg0.conf"

        client_b.vm.provision "shell", inline: $vpn_client_setup
        client_b.vm.provision "shell", inline: "wg-quick up wg0"
    end
end
