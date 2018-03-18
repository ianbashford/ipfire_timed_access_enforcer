# ipfire_timed_access_enforcer
IPFire Timed Access only affects new connections; this neatly deals with established connections.  See [my blog post for more details](https://www.ianbashford.net/post/ipfire_timed_access_enforcer/) 

## Overview
IPFire timed access is useful for controlling childrens access to the internet.  Unfortunately this only affects new connections (due to the ACCEPT ESTABLISHED rule in the CONNTRACK chain being assessed first).
This simple set of scripts will check every 5 minutes to see if a timed access rule has kicked in -- if it has it flushes the conntrack connections for that IP Address.  Those connections are then re-assessed by the timed access rule and denied, neatly booting the children off the internet no more than 5 mins after the allotted time.

## Rule Restrictions
The scripts are not sophisticated.  They won't work if you have one rule for all days.  If you have the same time constraints every day, just create two rules; one for Monday - Saturday, and one for Sunday.  (The rules are scraped to find rules effective today).
They also won't work if you have >1 rule for a day.

## Setting up the rules
I have the following configured:
A hosts group, with the MAC address of each of the children's machines.
The DHCP server binds those MAC addresses to the same IP Address.
Four rules per MAC Address in the following order (example times):
  1. ACCEPT Sunday 16.00 - 18.00
  2. ACCEPT Mon - Thu 16.30 - 18.00
  3. ACCEPT Fri - Sat 16.30 - 20.00
  4. REJECT ALL
(I use reject not drop so it fails, rather than just hangs)

## Script Configuration
In /etc/fcron.cyclic/conntrack, add one line for each of the machines you're controlling e.g.
  /usr/local/bin/drop_connections user1

In /usr/local/bin/drop_connections, you need to edit these lines to match the MAC address
user1_mac=<user_1_mac_address>

In /usr/local/bin/clear_conntrack, you need to put in the IP address that's assigned to that MAC
  conntrack -D -s <ip_address_for_user_1>

Two users are templated in the files.

## Effect
The clear conntrack will only kick in after the timed access has expired.  In the example above, on Sunday, the first time the connections will be cleared is the first run after 18.00.  The script will actually run every 5 minutes for the hour after timed access expires.  I don't see this as an issue as no new connections will be allowed so it'll have no effect.  However it won't clear connections while access is allowed, so there's no unnecessary clearing. 

## Logging
Some logs are put in /var/log/messages, so you can see what's going on
