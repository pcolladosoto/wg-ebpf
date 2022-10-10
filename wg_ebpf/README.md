# A sample eBPF program
This directory contains all the necessary goodies to compile and run an eBPF program
capable of keeping track of the number of datagrams per source IPv4 address 'seen' on
a given network interface.

In order to do so, it 'hooks' itself right into the
[eXpress Data Path](https://en.wikipedia.org/wiki/Express_Data_Path) (i.e. XPD) so that
the eBPF program is the 'first' thing an ethernet frame encounters after traversing the
machine's NIC.

The program itself is compiled by an [eBPF backend](https://llvm.org/docs/CodeGenerator.html)
provided by the [LLVM](https://llvm.org) project through the [`clang`](https://clang.llvm.org)
compiler. However, we don't really do this ourselves: we rely on
[cilium/ebpf](https://github.com/cilium/ebpf) for that task. The actual program carrying out the
compilation (which, in fact calls `clang`) is [`bpf2go`](https://github.com/cilium/ebpf/tree/master/cmd/bpf2go).
Instead of invoking it directly we can conveniently do so through [`go generate`](https://go.dev/blog/generate).
On top of that, we decided to write a `Makefile` (based on the one provided by [@fbac](https://github.com/fbac)
on [fbac/sklookup-go](https://github.com/fbac/sklookup-go)) which handles all of this transparently.

The above basically boils down to the fact that you need to run the following to build the code:

    make build

That's it! Please bear in mind this program is basically the same as the XDP example found on
[cilium/ebpf](https://github.com/cilium/ebpf/tree/master/examples). We just added a new function
skipping the Ethernet header for datagrams being forwarded within the WireGuard VPN. We also
added an IPv4 address we decided to ban by tweaking the program's return code. In the end, it
basically remains the same thing though.

## Dependencies
We can't really have our cake and eat it too... We need some dependencies for the above to work. These
are being installed by a provisioning script embedded on the Vagrantfile, but you can also get them
on your own. On a machine running Ubuntu these boil down to:

1. The Go language: we need it to compile the program itself... You can get it over [here](https://go.dev/doc/install).
2. The LLVM goodies: to compile the eBPF program. You can get those with `apt install llvm`.
3. The `clang` compiler: to compile the eBPF program too. You can also get that one with `apt install clang`.
4. `make`: to leverage the Makefile. You can get it with `apt install make`, but we recommend just installing `build-essential` instead.

With that you should be all set!

## Running the program
Once it's compiled you should be able to run the program with:

    # Bear in mind we need to 'tell' the program what interface to
        # attach to. The name can be any of the ones included on
        # `ip l`'s output.
    $ sudo ./bin/wg-ebpf <ifname>

Once it starts, you should being to see the map's contents every second on screen. Be sure to
send some `ping`s here and there to check how everything works!
