# Pihole


Pi-hole role. This will install [Pi-hole®: A black hole for Internet
advertisements – A black hole for Internet
advertisements](https://pi-hole.net/) for you.

It also helps you administering Pi-Hole using a simple CSV file that you can
edit with your favourite editor.

## Requirements

A host running Ubuntu/Debian/Photon OS or e.g. a raspberry pi running Raspberry
Pi OS (Lite).

If you run into problems, a look into the [Pi-hole
documentation](https://docs.pi-hole.net/docker/dhcp/) may help.

### Host CSV File

Pi-hole configuration is done via a csv file. This gives you a nice overview
and makes it easy to manage your environment.

The role includes a full example CSV file in the `files` folder for you to
review.

The following lines do show `dhcp-option` settings that will be written into
the dnsmasq configuration file. The lines tag one IP address as the vm based
DNS server and the raspberry pi based DNS server running on an actual raspberry
pi.

#### Set DHCP Options

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,type,comment
,,,,,"tag:vm-dns,option:dns-server,192.168.30.253","<<tag>>","Sets value of dhcp-option configuration option, here which DNS server to use"
,,,,,"tag:pi-dns,option:dns-server,192.168.30.254","<<tag>>","Sets value of dhcp-option configuration option, here which DNS server to use"
```

**Note:** If you are setting up multiple DNS servers here, you probably do want
to set the `riv_pihole_dns_default_tag` option when configuring your setup to set a default DNS server
for clients without a specific configuration.

#### Setup IP/Host mapping

The configuration below configures the host with the name `slash` to receive an
IP address of `192.168.128.2` if that address is still available. If a DHCP
lease was already handed out to another machine, a new IP address will be
assigned. It also identifies the machine as a physical host.

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,type,comment
slash,example.net,192.168.128.2,true,,,"<<physical>>","ESXi Host""
```

#### Setup static IP address

If you want to make sure that the IP address is assigned to a specific hostname
only, you can set the `static` field to `true` to achieve that. Setting this
option will instruct dnsmasq to ignore DHCP requests from any host named
`vmhole` as shown in the example below. The type is also identified as a virtual machine.

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,type,comment
vmhole,example.net,192.168.128.253,true,,,"<<virtual>>","Pi-hole Virtual machine DNS server"
```
 
#### Setup Mac address/IP address mapping

To assign a specific IP address and hostname to a special device using the mac
address of the device, use the following entry in the csv configuration
database.

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,type,comment
blib,example.net,192.168.128.18,,00:0c:29:43:37:dc,,"<<virtual>>","Pi-hole Virtual machine DNS server"
```

#### Set different DNS server

Setting up a special DNS server for some devices can be achieved by supplying
the appropriate DHCP option when defining the mapping. The below example shows
how to set the `vm-dns` DNS server for the photon host.

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,type,comment
photon,example.net,192.168.128.19,,00:0c:29:51:80:1f,"vm-dns","<<virtual>>","Proxy server VM based on a docker image, using vm-dns server"
```

## Role Variables

The role uses the following variables:

| Variable                             | Default                                                                                        | Description                                                                                                                                                                                                                          |
| ------------------------------------ | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| riv_pihole_admin_password_generated  | {{ lookup('password', '/dev/null length=15 chars=ascii_letters') }}                            | This variable sets the password for the web user interface. If no password is set, it will be auto-generated. The password value is shown in plain text in the last step if the variable `riv_pihole_show_summary` is set to `true`. |
| riv_pihole_dns_default_tag           |                                                                                                | This variable sets a DHCP tag that lets you specify a default DNS server for hosts not listed in the configuration csv file.                                                                                                         |
| riv_pihole_show_summary              | false                                                                                          | If set `true` the last step will show the (generated) password in plain text along with some useful information like the IP address and name of the host pihole was installed on.                                                    |
| riv_pihole_docker_network            | host                                                                                           | The network to which docker connects. If you want to use DHCP, you need to connect to the host network.                                                                                                                              |
| riv_pihole_docker_purge_networks     | yes                                                                                            | Remove the created network when the docker container is shut down.                                                                                                                                                                   |
| riv_pihole_dhcp_active               | false                                                                                          | Enable or disable the built-in DHCP server. If you want to use Pi-hole as DHCP server, you need to turn this on.                                                                                                                     |
| riv_pihole_dhcp_leasetime            | 24h                                                                                            | The default lease time to set when handing out a client IP address                                                                                                                                                                   |
| riv_pihole_dhcp_start                | 192.168.1.2                                                                                    | The starting range of the built-in DHCP server.                                                                                                                                                                                      |
| riv_pihole_dhcp_end                  | 192.168.1.253                                                                                  | The last address of the built-in DHCP server.                                                                                                                                                                                        |
| riv_pihole_dhcp_router               | 192.168.1.1                                                                                    | The router which should be advertised to clients getting an IP address.                                                                                                                                                              |
| riv_pihole_domain                    | example.net                                                                                    | The domain of your local network.                                                                                                                                                                                                    |
| riv_pihole_interface                 | eth0                                                                                           | The interface processes inside Pi-hole will bind to                                                                                                                                                                                  |
| riv_pihole_dhcp_ipv6                 | false                                                                                          | Enable IPv6 support on DHCP.                                                                                                                                                                                                         |
| riv_pihole_dhcp_rapid_commit         | false                                                                                          | Control switch for the rapid commit option.                                                                                                                                                                                          |
| riv_pihole_dnsmasq_listening         | all                                                                                            | The interface dnsmasq should listen on.                                                                                                                                                                                              |
| riv_pihole_query_logging             | true                                                                                           | Log DNS queries.                                                                                                                                                                                                                     |
| riv_pihole_install_web_server        | true                                                                                           | Install the built in Web-Server                                                                                                                                                                                                      |
| riv_pihole_install_web_interface     | true                                                                                           | Install the Web-Interface.                                                                                                                                                                                                           |
| riv_pihole_lighttpd_enabled          |                                                                                                | Enable lighttpd                                                                                                                                                                                                                      |
| riv_pihole_ipv4_address              | {{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] \| default(ansible_host) }} | The IPv4 address you want Pi-Hole to listen on.                                                                                                                                                                                      |
| riv_pihole_ipv6_address              |                                                                                                | The IPv6 address you want Pi-Hole to listen on.                                                                                                                                                                                      |
| riv_pihole_dns_bogus_priv            |                                                                                                | Control bogus private reverse lookups                                                                                                                                                                                                |
| riv_pihole_dns_fqdn_required         |                                                                                                | Controls if the unqualified name is put into DNS as well                                                                                                                                                                             |
| riv_pihole_dnssec                    |                                                                                                | Control DNSSEC                                                                                                                                                                                                                       |
| riv_pihole_rev_server_enabled        |                                                                                                | Enable DNS conditional forwarding for device name resolution.                                                                                                                                                                        |
| riv_pihole_rev_server_cidr           |                                                                                                | If conditional forwarding is enabled, set the reverse DNS zone (e.g. 192.168.0.0/24)                                                                                                                                                 |
| riv_pihole_rev_server_domain         |                                                                                                | If conditional forwarding is enabled, set the domain of the local network router (e.g example.net).                                                                                                                                  |
| riv_pihole_rev_server_target         |                                                                                                | If conditional forwarding is enabled, set the IP of the local network router.                                                                                                                                                        |
| riv_pihole_docker_image              | pihole/pihole:latest                                                                           | Which docker image to pull. You can e.g. specify a version.                                                                                                                                                                          |
| riv_pihole_configuration_base_folder | /opt/pihole                                                                                    | Where do you want your configuration to live on the target host.                                                                                                                                                                     |
| riv_pihole_etc_pihole_folder         | {{ riv_pihole_configuration_base_folder }}/pihole                                              | Control position of pihole folder.                                                                                                                                                                                                   |
| riv_pihole_etc_dnsmasq_folder        | {{ riv_pihole_configuration_base_folder }}/dnsmasq.d                                           | Control position of dnsmasq.d folder.                                                                                                                                                                                                |
| riv_pihole_sys_dns_server1           | 127.0.0.1                                                                                      | System DNS servers for Pi-hole. The first one HAS TO BE 127.0.0.1                                                                                                                                                                    |
| riv_pihole_sys_dns_server2           | 8.8.8.8                                                                                        | System DNS server for Pi-hole                                                                                                                                                                                                        |
| riv_pihole_dns_server1               | 1.1.1.1                                                                                        | DNS Server used inside the docker container                                                                                                                                                                                          |
| riv_pihole_dns_server2               | 8.8.8.8                                                                                        | DNS Server used inside the docker container                                                                                                                                                                                          |
| riv_pihole_serverip                  | {{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] \| default(ansible_host) }} | The IP address of the Pi-hole host                                                                                                                                                                                                   |
| riv_pihole_timezone                  | Europe/Berlin                                                                                  | Your timezone                                                                                                                                                                                                                        |
| riv_pihole_open_firewall_ports       | true                                                                                           | Set to false if you don't want the role to open required firewall ports.                                                                                                                                                             |
| riv_pihole_dns_db_configuration_file | {{ role_path }}/files/mappings.csv                                                             | The location of the DNS configuration file. The default file pointed to should be copied and adopted appropriately.                                                                                                                  |

## Example Playbook

The example below shows a playbook you can use to configure pi-hole on a Photon
OS based virtual machine which is reflected in the inventory file. 

### Folder structure

The folder structure is as follows (the roles folder will be created automatically later):

```
|-- dns-db.csv
|-- inventory
|-- pihole-vm.yml
`-- roles
    `-- authsec.pihole
        |-- defaults
        |   `-- main.yml
        |-- files
        |   `-- mappings.csv
        |-- LICENSE
        |-- meta
        |   `-- main.yml
        |-- README.md
        |-- tasks
        |   |-- checkssh-photonos.yml
        |   |-- checkssh.yml
        |   |-- firewall-iptables.yml
        |   |-- install-apt.yml
        |   |-- install-photonos.yml
        |   `-- main.yml
        `-- templates
            |-- 02-pihole-dhcp.conf.j2
            |-- 10-pihole-custom-static.conf.j2
            `-- setupVars.conf.j2
```

### Import Role

You can import the role into your project using:

``` bash
#> ansible-galaxy install -p roles authsec.pihole
```

### Create database 

This is a copy of the example database in `roles/authsec.pihole/files`.

**dns-db.csv:**

``` csv
hostname,domainname,ip_address,static,mac_address,dhcp_option,comment
,,,,,"tag:vm-dns,option:dns-server,192.168.128.253","Sets value of dhcp-option configuration option"
,,,,,"tag:pi-dns,option:dns-server,192.168.128.254","Sets value of dhcp-option configuration option"
slash,example.net,192.168.128.2,,,,"ESXi Host"
mohh,example.net,192.168.128.5,,,,"The Brain (vCenter Server)"
blib,example.net,192.168.128.18,,00:0c:29:43:37:dc,"pi-dns","VM using Raspberry Pi based DNS (and DHCP) server"
photon,example.net,192.168.128.19,,00:0c:29:51:80:1f,"vm-dns","VM using VM based DNS server"
vmhole,example.net,192.168.128.253,true,,,"Pi-hole Virtual machine DNS server"
pihole,example.net,192.168.128.254,,,,"Raspberry Pi backed Pi-Hole DNS and DHCP server for this network, docker based and ansible managed"
```

**inventory:**

```
[dns_vms]
vmhole.example.net ansible_host=192.168.128.253 ansible_user=pihole ansible_become_method='su' ansible_become_password='kevin.is.dead' ansible_python_interpreter=/usr/bin/python3
```

**pihole-vm.yml:**

``` yaml
---
# Configure a machine to run pi-hole inside a docker container
- hosts: dns_vms
  gather_facts: yes
  become: yes
  tasks: 
    - include_role: 
        name: authsec.pihole
      vars:
        # true|false
        riv_pihole_admin_password: "secure.me"
        riv_pihole_dhcp_active: "false"
        riv_pihole_dhcp_start: "192.168.128.8"
        riv_pihole_dhcp_end: "192.168.128.252"
        riv_pihole_dhcp_router: "192.168.128.1"
        riv_pihole_domain: "example.net"
        riv_pihole_show_summary: true
        riv_pihole_dns_db_configuration_file: "dns-db.csv"
```

### Run ansible

Once everything is configured, you can run `ansible` to setup pi-hole on the
system configured in the `inventory`.

``` bash
#> ansible-playbook -i inventory pihole-vm.yml
```

License
-------

MIT
