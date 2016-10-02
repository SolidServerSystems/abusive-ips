# Abusive IP's

This repo maintains a list of IP addresses that routinely show abuse on our servers, most of which are attempting brute force ssh attacks. It also documents how to edit fail2ban configs to persist repeat offenders.



#### TO DO

- [x] De Dup IPs
- [x] Place IP list in separate file
- [ ] Add instructions on how to block IPS from hitting ubuntu servers
- [ ] Create online tool for parsing out IP Address from text
- [ ] Create online tool for deduplicating IP Addresses from list

---


## Persist with Fail2Ban

To make any IP bans persist, set up a blacklist and automatically load the IP Address when fail2ban starts. First, change directories into your fail2ban directory (if you don't have fail2ban, install it):

    cd /etc/fail2ban
        
You can either start from scratch with your own empty `ip.blacklist` or you you can import ours:


    # Start From Scratch
    sudo touch /etc/fail2ban/ip.blacklist

    # Or Import Our List
    wget -P /etc/fail2ban/ https://raw.githubusercontent.com/SolidServerSystems/abusive-ips/master/ip.blacklist 

Make up a backup of `/etc/fail2ban/action.d/iptables-multiport.conf` before edits:

    cp /etc/fail2ban/action.d/iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.conf.org


Let's edit the config file...

    vi /etc/fail2ban/action.d/iptables-multiport.conf
    
Adding the following line under the default `actionban` 

    echo <ip> >> /etc/fail2ban/ip.blacklist
    
The result will look something like this

    actionban = iptables -I fail2ban-<name> 1 -s <ip> -j <blocktype>
                echo <ip> >> /etc/fail2ban/ip.blacklist

Finally, let's make sure upon start fail2ban loads our iptable with rules to ban these ips. Append to `actionstart` this line:

    cat /etc/fail2ban/ip.blacklist | while read IP; do iptables -I fail2ban-<name> 1 -s $IP -j DROP; done
    
The result will look something like this

    actionstart = iptables -N fail2ban-<name>
                  iptables -A fail2ban-<name> -j RETURN
                  iptables -I <chain> -p <protocol> -m multiport --dports <port> -j fail2ban-<name>
                  cat /etc/fail2ban/ip.blacklist | while read IP; do iptables -I fail2ban-<name> 1 -s $IP -j DROP; done


Restart fail2ban and from now on your bans will persist:

    service fail2ban restart

