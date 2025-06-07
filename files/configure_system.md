This instruction is created for Debian11. Other versions may require additional configuration steps which are not
described.

1. Become sudo
    ```bash
    sudo -i
    ```
2. update
    ```bash
    apt-get update && apt-get upgrade -y
    ```
3. install required packages
    ```bash
    apt-get install charon-systemd strongswan-libcharon strongswan-swanctl strongswan libcharon-extra-plugins charon-cmd strongswan-pki iptables-persistent -y
    ```

4. Enable forwarding, prevent MITM attacks, restrict ICMP and restrict PMTU search
   modify /etc/sysctl.conf
   add lines
   ```bash
   echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
   echo "net.ipv4.conf.all.accept_redirects=0" >> /etc/sysctl.conf
   echo "net.ipv4.conf.all.send_redirects=0" >> /etc/sysctl.conf
   echo "net.ipv4.ip_no_pmtu_disc=1" >> /etc/sysctl.conf
   ```

   exit with save and load settings
   ```bash
    sysctl -p
   ```


