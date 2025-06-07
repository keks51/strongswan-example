## Example with https://whatismyipaddress.com/
1. get ip from https://www.nslookup.io/website-to-ip-lookup/
2. iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o eth0 --dst 104.19.222.79  -j MASQUERADE
3. save netfilter rule sets
   ```bash
   netfilter-persistent save
   netfilter-persistent reload
   ```

4. to delete use same command but replace -A with -D
