This is a fork of Cumulus Network repo (http://github.com/cumulusNetworks/cumulus-cl-interfaces-puppet).

Actual puppet forge page: https://forge.puppet.com/c2devel/cumulus_interfaces

Feel free to send patches to http://github.com/c2devel/cumulus-cl-interfaces-puppet/

# cumulus_interface

#### Table of Contents

- [cumulus_interface](#cumulus_interface)
      - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Module Description](#module-description)
    - [cumulus_interface](#cumulus_interface-1)
    - [cumulus_bond](#cumulus_bond)
    - [cumulus_bridge](#cumulus_bridge)
  - [Setup](#setup)
    - [What cumulus_interface affects](#what-cumulus_interface-affects)
  - [Usage](#usage)
  - [Reference](#reference)
    - [Types](#types)
      - [`cumulus_interface`](#cumulus_interface-2)
        - [Parameters](#parameters)
      - [`cumulus_bond`](#cumulus_bond-1)
        - [Parameters](#parameters-1)
      - [`cumulus_bridge`](#cumulus_bridge-1)
        - [Parameters:](#parameters-2)
      - [`cumulus_vxlan`](#cumulus_vxlan)
        - [Parameters:](#parameters-3)
  - [Limitations](#limitations)
  - [Development](#development)
  - [Cumulus Linux](#cumulus-linux)

## Overview

This module provides three resource types that can configure
most types of interfaces available on Cumulus Linux.

## Module Description

The module consists of three resources types:

### cumulus_interface

This resource type configures a network interface using [ifupdown2](http://docs.cumulusnetworks.com/display/CL25/Network+Interface+Management+Using+ifupdown2). The configuration for the interface is written to a file in the interface configuration file directory. This resource type does not configure VXLAN, bond, or bridge interfaces.

For bridge configuration use the `cumulus_bridge` module.

For bond configuration use the `cumulus_bond` module.

### cumulus_bond

This resource type configures a network bond using [ifupdown2](http://docs.cumulusnetworks.com/display/CL25/Network+Interface+Management+Using+ifupdown2). The configuration for the interface is written to a file in the interface configuration file directory.

### cumulus_bridge

This resource type configures a bridge using [ifupdown2](http://docs.cumulusnetworks.com/display/CL25/Network+Interface+Management+Using+ifupdown2). The configuration for the interface is written to a file in the interface configuration file directory.


## Setup

### What cumulus_interface affects

* This module affects the configuration files located in the interfaces folder and managed by [ifupdown2](http://docs.cumulusnetworks.com/display/CL25/Network+Interface+Management+Using+ifupdown2).
By default this is `/etc/network/interfaces.d`.

* You need to add `source /etc/network/interfaces.d/*` to `/etc/network/interfaces` to make use of the new files.

* To activate the changes, run `/sbin/ifreload -a`.

> **NOTE**: Reloading the interface configuration is not disruptive if there is no change in the configuration.

## Usage

**cumulus_interface Examples:**

Loopback interface and the management interface `eth0` using DHCP:

```puppet
cumulus_interface { 'lo':
  addr_method => 'loopback',
}

cumulus_interface { 'eth0':
  addr_method => 'dhcp',
}

cumulus_interface { 'eth0':
  ipv4    => ['192.168.0.10/24'],
  gateway => '192.168.0.1',
}
```

*swp33* as a 1GbE port with a single IPv4 address:

```puppet
cumulus_interface { 'swp33':
  ipv4  => ['10.30.1.1/24'],
  speed => 1000,
}
```

*peerlink.4094*, a bond sub-interface, as the CLAG peer interface:

```puppet
cumulus_interface { 'peerlink.4094':
  ipv4          => ['10.100.1.0/31'],
  clagd_enable  => true,
  clagd_peer_ip => '10.100.1.1/31',
  clagd_sys_mac => '44:38:39:ff:20:94',
}
```

**cumulus_bond Examples:**

Bond named *peerlink* with the interfaces *swp1* and *swp2* as bond members:

```puppet
cumulus_bond { 'peerlink':
  slaves => ['swp1-2'],
}
```

Bond named *bond0* with the interfaces  *swp3* and  *swp4* as  bond members, the minimum link count is set to 2 and the [CLAG](http://docs.cumulusnetworks.com/display/CL25/Multi-Chassis+Link+Aggregation+-+CLAG+-+MLAG) ID is set:

```puppet
cumulus_bond { 'bond0':
  slaves    => ['swp3-4'],
  min_links => 2,
  clag_id   => 1,
}
```

**cumulus_bridge Examples:**

[Default ("traditional") bridge driver](http://docs.cumulusnetworks.com/display/CL25/Ethernet+Bridging+-+VLANs):

```puppet
cumulus_bridge { 'br10':
  ports            => ['swp11-12.1', 'swp32.1'],
  ipv4             => ['10.1.1.1/24', '10.20.1.1/24'],
  ipv6             => ['2001:db8:abcd::/48'],
  alias_name       => 'classic bridge',
  mtu              => 9000,
  mstpctl_treeprio => 4096,
}
```

[VLAN-aware bridge](http://docs.cumulusnetworks.com/display/CL25/VLAN-aware+Bridge+Mode+for+Large-scale+Layer+2+Environments):

```puppet
cumulus_bridge { 'bridge':
  vlan_aware       => true,
  ports            => ['peerlink', 'downlink', 'swp10'],
  vids             => ['1-4094'],
  pvid             => 1,
  stp              => true,
  mstpctl_treeprio => 4096,
}
```

## Reference

### Types

#### `cumulus_interface`

##### Parameters

* `name` - Identifier for the interface.
* `ipv4` - Array of IPv4 addresses to be applied to the interface.
* `ipv6` - Array of IPv6 addresses to be applied to the interface.
* `gateway` - String of default gateway to be added with the interface.
* `alias_name` - Interface alias.
* `addr_method` - Address assignment method, `dhcp` or `loopback`. Default is empty (no address method is set).
* `speed` - The interface link speed.
* `mtu` - The interface Maximum Transmission Unit (MTU).
* `virtual_ip` - VRR virtual IP address.
* `virtual_mac` - VRR virtual MAC address.
* `access` - For bridging, a type of port that is non-trunking. For dot1x an IP source address or network that will be serviced (an integer from 1 to 4094).
* `allow_untagged` - A bridge port interface may allow untagged packets. Valid value: `false`.
* `vids` - Array of VLANs to be configured for a VLAN-aware trunk interface.
* `pvid` - Native VLAN for a VLAN-aware trunk interface.
* `location` - Location of the configuration snippets directory. Default is `/etc/network/interfaces.d/`.
* `mstpctl_portnetwork` - Enables bridge assurance on a VLAN-aware trunk.
* `mstpctl_bpduguard` - Enables BPDU guard on a VLAN-aware trunk.
* `mstpctl_portadminedge` - Enables admin edgeport
* `vlan_raw_device` - Vlan-aware bridge name, which is a master device for this interface.
* `vlan_id` - Interface's vlan id in vlan-aware bridge

The following CLAG-related attributes are also available. If CLAG is enabled, you must specify ``clagd_enable``,``clagd_priority``, ``clagd_peer_id`` and ``clagd_sys_mac``:

* `clagd_enable` - Enable the `clagd` daemon.
* `clagd_priority` - Set the CLAG priority for this switch.
* `clagd_peer_id` - Address of the CLAG peer switch.
* `clagd_sys_mac` - CLAG system MAC address. The MAC address must be identical on both CLAG peers.
* `clagd_args` - Any additional arguments to be passed to the `clagd` deamon.
* `clagd_backup_ip` -  backup IP address to communicate with the peer switch, in case the peer link is down but the peer switch is up.
* `clagd_vxlan_anycast_ip` - VXLAN anycast IP for CLAG.

#### `cumulus_bond`

##### Parameters

* `name` - Identifier for the bond interface.
* `slaves` - Bond members.
* `min_links` - Minimum number of slave links for the bond to be considered up. Default is 1.
* `mode` - Bond mode. Default is 802.3ad.
* `miimon` - MII link monitoring interval. Default is 100.
* `xmit_hash_policy` - TX hashing policy. Default is layer3+4.
* `lacp_rate` - LACP bond rate. Default is 1 (fast LACP timeout).
* `ipv4` - Array of IPv4 addresses to be applied to the interface.
* `ipv6` - Array of IPv6 addresses to be applied to the interface.
* `gateway` - String of default gateway to be added with the interface.
* `alias_name` - Interface alias.
* `addr_method` - Address assignment method. May be `dhcp` or empty. Default is empty (no address method is set).
* `mtu` - The interface Maximum Transmission Unit (MTU).
* `virtual_ip` - VRR virtual IP address.
* `virtual_mac` - VRR virtual MAC address.
* `access` - For bridging, a type of port that is non-trunking. For dot1x an IP source address or network that will be serviced (an integer from 1 to 4094).
* `allow_untagged` - A bridge port interface may allow untagged packets. Valid value: `false`.
* `vids` - Array of VLANs to be configured for a VLAN-aware trunk interface.
* `pvid` - Native VLAN for a VLAN-aware trunk interface.
* `location` - Location of the configuration snippets directory. Default is `/etc/network/interfaces.d/`.
* `mstpctl_portnetwork` - Enable bridge assurance on a VLAN-aware trunk.
* `mstpctl_bpduguard` - Enable BPDU guard on a VLAN-aware trunk.
* `mstpctl_portadminedge` - Enables admin edgeport
* `clag_id` - Define which bond is in the CLAG. The ID must be the same on both CLAG peers.
* `lacp_bypass_allow` - Enable LACP bypass, valid options are 0 or 1.
* `lacp_bypass_period` - Period for enable lacp_bypass.
* `lacp_bypass_priority` - Array of ports and priority
* `lacp_bypass_all_active` - Activate all interfaces for bypass: 0 or 1.
* `use_carrier` - Specifies whether or not miimon should use MII or Ethtool ioctls. Default is 1.
* `vlan_raw_device` - Vlan-aware bridge name, which is a master device for this interface.
* `vlan_id` - Interface's vlan id in vlan-aware bridge

#### `cumulus_bridge`

##### Parameters:

* `name` - Identifier for the bridge interface.
* `ipv4` - Array of IPv4 addresses to be applied to the interface.
* `ipv6` - Array of IPv6 addresses to be applied to the interface.
* `gateway` - String of default gateway to be added with the interface.
* `alias_name` - Interface alias.
* `addr_method` - Address assignment method. May be `dhcp` or empty. Default is empty (no address method is set).
* `mtu` - The interface Maximum Transmission Unit (MTU).
* `stp` - Enable spanning tree. Default is true.
* `mstpctl_treeprio` - Bridge tree root priority. Must be a multiple of 4096.
* `vlan_aware` - Use the VLAN-aware bridge driver. Default is false.
* `virtual_ip` - VRR virtual IP address.
* `virtual_mac` - VRR virtual MAC address.
* `vids` - Array of VLANs to be configured for a VLAN-aware trunk interface.
* `pvid` - Native VLAN for a VLAN-aware trunk interface.
* `location` - Location of the configuration snippets directory. Default is `/etc/network/interfaces.d/`.
* `mcsnoop` -  Enables IGMP/MLD Snooping on the bridge. Default is 0.

#### `cumulus_vxlan`

##### Parameters:

* `name` - Identifier for the VXLAN interface.
* `alias_name` - Interface alias.
* `mtu` - The interface Maximum Transmission Unit (MTU).
* `mstpctl_bpduguard` - Enable BPDU guard on a VLAN-aware trunk.
* `location` - Location of the configuration snippets directory. Default is `/etc/network/interfaces.d/`.
* `access` - For bridging, a type of port that is non-trunking. For dot1x an IP source address or network that will be serviced (an integer from 1 to 4094).
* `arp_nd_suppress` - ARP ND suppression. Valid values: `'on'`, `'off'`.
* `learning` - The bridge port learning flag. Valid value: `'off'`.
* `mstpctl_portbpdufilter` - BPDU filter on a port. Valid values: `true` or `false`.
* `mstpctl_bpduguard` - Bridge Protocol Data Unit guard. Valid values: `true` or `false`.
* `vxlan_id` - VXLAN Identifier (An integer from 1 to 16777214)
* `vxlan_local_tunnelip` - VXLAN local tunnel ip


## Limitations

This module only works on Cumulus Linux.

The `puppet resource` command for `cumulus_interface`, `cumulus_bond` and
`cumulus_bridge` is currently not supported. It may be added in a future release.

## Development

1. Fork it.
2. Create your feature branch (`git checkout -b my-new-feature`).
3. Commit your changes (`git commit -am 'Add some feature'`).
4. Push to the branch (`git push origin my-new-feature`).
5. Create new Pull Request.

## Cumulus Linux

![Cumulus Networks Icon](http://cumulusnetworks.com/static/cumulus/img/logo_2014.png)

Cumulus Linux is a software distribution that runs on top of industry-standard
networking hardware. It enables the latest Linux applications and automation
tools on networking gear while delivering new levels of innovation and
ﬂexibility to the data center.

For further details, please see [http://cumulusnetworks.com](http://www.cumulusnetworks.com).
