Daemon to monitor when VM or CT is migrated from or to proxmox VE node.
Daemon uses inotifywait to monitor changes to conf files in /etc/pve/qemu-server
and /etc/pve/lxc directories, which have config files for VM and CT in current
node. When VM/CT is migrated out of node, all scripts in down.d directory
will be executed, when is migrated into node, all scripts in up.d directory
will be executed. Down.d and up.d scripts get config file as argument.

Pve-migration-monitor is a shell script, and it requires inotifywait, curl and jq
(provided by inotify-tools, curl and jq debian packages). It also requires grep,
awk and sed, which are usually installed in every linux system.

Install script will copy scripts to /etc/pve-migration-monitor and config file
to /etc/default/pve-migration-monitor. It will add pve-migration-monitor service
to systemd and enable it.

Failover IP
===========

Pve-migration-monitor comes with scripts to change failover IP on VM/CT when
is migrated, failoverip-down and failoverip-up (linked to down.d/10failoverip
and up.d/10failoverip). Failoverip-up script use another script to change
failover IP with API, a script for Online.net API is provided.

Install script can be run with -p <provider> argument to enable these scripts,
and it will ask for online.net api token.

Also, scripts to get public IP on VM/CT are included, they write VM/CT IP to config
files in /etc/pve/failoverip by default, so failoverip-down can get failoverip for
VM, because qemu/lxc config file has been already removed from node.

To get failover config files, 2 scrips are provided to get them in 2 different ways:

- vm-ip-from-arp: It gets VM/CT mac addresses from config files, then looks for
  these ip addresses in ARP table. Failover ip address won't be in ARP table if
  node doesn't have traffic to these IP addresses first, so script gets failover ip
  addresses must be read with API, and ping to them. It can be enabled with -p arg.
  For example:
  
    for ip in $(OnlineFailoverIP ips); do
        arp -d $ip # remove cached entry in case IP has been removed
        ping -q -c 1 $ip > /dev/null
    done
    
- vm-public-ip: It gets VM IP addresses from qemu guest agent, so it guest agent has
  to be installed in VM. For CT, it gets IP addresses from ip addr command.

These scripts can be run with -v to print IPs per VM/CT without writting to /etc/pve/failoverip.
Only one script is needed, it can be run manually when failover ip is added or removed from VM,
or scheduled. Systemd timers are provided to run script hourly, it can be changed to any other
frequency.

Acknowledgements
================

To [@biapy](https://github.com/biapy) for [OnlineFailoverIP](https://github.com/biapy/howto.biapy.com/blob/master/various/OnlineFailoverIP) script, which I based mine on.  
To [@yennor](https://github.com/yennor) for [proxmox hetzner failover ip scripts](https://github.com/yennor/proxmox-hetzner-failover-ip), which I used as base for pve-migration-monitor.
