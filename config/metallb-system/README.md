# MetalLB configuration

The specific configuration depends on the protocol(s) you want to use to announce service IPs. Jump to:

- [Layer 2 configuration](#layer-2-configuration)
- [BGP configuration](#bgp-configuration)

## Layer 2 configuration

Layer 2 mode is the simplest to configure: in many cases, you don't
need any protocol-specific configuration, only IP addresses.

Layer 2 mode does not require the IPs to be bound to the network interfaces
of your worker nodes. It works by responding to ARP requests on your local
network directly, to give the machine's MAC address to clients.

For example, the following configuration gives MetalLB control over
IPs from `172.16.5.203` to `172.16.5.204`, and configures Layer 2
mode:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.16.5.203-172.16.5.204
```

## BGP configuration

For a basic configuration featuring one BGP router and one IP address
range, you need 4 pieces of information:

- The router IP address that MetalLB should connect to,
- The router's AS number,
- The AS number MetalLB should use,
- An IP address range expressed as a CIDR prefix.

As an example, if you want to give MetalLB the range 172.16.5.0/24
and AS number 64512, and connect it to a router at 172.16.5.254 with AS
number 64513, your configuration will look like:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    peers:
    - peer-address: 172.16.5.254
      peer-asn: 64513
      my-asn: 64512
    address-pools:
    - name: default
      protocol: bgp
      addresses:
      - 172.16.5.0/24
```

You will find below the [Mikrotik](https://mikrotik.com/) configuration to enable BGP support:

```
/routing bgp instance
set default as=64513 client-to-client-reflection=yes disabled=no ignore-as-path-len=no name=default redistribute-connected=yes redistribute-ospf=yes redistribute-other-bgp=no redistribute-rip=no redistribute-static=yes router-id=172.16.5.254
/routing bgp peer
add name=worker01 disabled=no instance=default remote-address=172.16.5.213 remote-as=64512
add name=worker02 disabled=no instance=default remote-address=172.16.5.214 remote-as=64512
```



