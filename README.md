# nft-blacklist

A Bash shell script which uses nftables sets to ban a large number of IP addresses published in IP blacklists.

## What's new

- 12/31/2023: Add more customization options using shell variables {[@henrythasler](https://github.com/henrythasler)}
- 08/26/2022: Added experimental IPv6 support and whitelists [@leshniak](https://github.com/leshniak)
- 08/24/2022: Created this fork and nftables-based version [@leshniak](https://github.com/leshniak)
- 10/17/2018: Added support for CIDR aggregation if iprange command is available
- 10/17/2018: Merged Shellcheck PR from [@extremeshok](https://github.com/extremeshok)
- 05/10/2018: Added regex filter improvements from [@sbujam](https://github.com/sbujam)
- 08/15/2017: Filtering default gateway and multicast ranges
- 01/20/2017: Ignoring "Service unavailable" HTTP status code, removed IGNORE_CURL_ERRORS 
- 11/04/2016: Documentation added to show how to prevent fail2ban from inserting its rules above the ipset-blacklist when restarting the fail2ban service
- 11/11/2015: Merged all suggestions from [@drzraf](https://github.com/drzraf)
- 10/24/2015: Outsourced the entire configuration in it's own configuration file. Makes updating the shell script way easier!
- 10/22/2015: Changed the documentation, the script should be put in /usr/local/sbin not /usr/local/bin

## Quick start for Debian/Ubuntu based installations

1. `wget -O /usr/local/sbin/nft-blacklist.sh https://raw.githubusercontent.com/leshniak/nft-blacklist/master/nft-blacklist.sh`
2. `chmod +x /usr/local/sbin/nft-blacklist.sh`
3. `mkdir -p /etc/nft-blacklist && mkdir -p /var/cache/nft-blacklist ; wget -O /etc/nft-blacklist/nft-blacklist.conf https://raw.githubusercontent.com/leshniak/nft-blacklist/master/nft-blacklist.conf`
4. Modify `nft-blacklist.conf` according to your needs. Per default, the blacklisted IP addresses will be saved to `/var/cache/nft-blacklist/blacklist.nft`
5. `apt-get install nftables`
6. Download `cidr-merger` from https://github.com/zhanhb/cidr-merger/releases
7. Create the nftables blacklist (see below). After proper testing, make sure to persist it in your firewall script or similar or the rules will be lost after the next reboot.
8. Auto-update the blacklist using a cron job

## First run, create the list

to generate the `/etc/nft-blacklist/ip-blacklist.restore`:

```sh
/usr/local/sbin/nft-blacklist.sh /etc/nft-blacklist/nft-blacklist.conf
```

## nftables filter rule

```sh
# Enable blacklists
nft -f /var/cache/nft-blacklist/blacklist.nft
```

Make sure to run this snippet in a firewall script or just insert it to `/etc/rc.local`.

## Cron job

In order to auto-update the blacklist, copy the following code into `/etc/cron.d/nft-blacklist-update`. Don't update the list too often or some blacklist providers will ban your IP address. Once a day should be OK though.

```sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
33 23 * * *      root /usr/local/sbin/nft-blacklist.sh /var/cache/nft-blacklist/nft-blacklist.conf
```

## Check for dropped packets

Using nftables, you can check how many packets got dropped using the blacklist:

```sh
leshniak@raspberrypi ~> sudo nft list counter inet blackhole blacklist_v4
table inet blackhole {
        counter blacklist_v4 {
                packets 52 bytes 2303
        }
}
leshniak@raspberrypi ~> sudo nft list counter inet blackhole blacklist_v6
table inet blackhole {
        counter blacklist_v6 {
                packets 0 bytes 0
        }
}
```

## Modify the blacklists you want to use

Edit the BLACKLIST array in /etc/nft-blacklist/nft-blacklist.conf to add or remove blacklists, or use it to add your own blacklists.

```sh
BLACKLISTS=(
"http://www.mysite.me/files/mycustomblacklist.txt" # Your personal blacklist
"http://www.projecthoneypot.org/list_of_ips.php?t=d&rss=1" # Project Honey Pot Directory of Dictionary Attacker IPs
# I don't want this: "http://www.openbl.org/lists/base.txt"  # OpenBL.org 30 day List
)
```

If you for some reason want to ban all IP addresses from a certain country, have a look at [IPverse.net's](http://ipverse.net/ipblocks/data/countries/) aggregated IP lists which you can simply add to the BLACKLISTS variable. For a ton of spam and malware related blacklists, check out this github repo: https://github.com/firehol/blocklist-ipsets
