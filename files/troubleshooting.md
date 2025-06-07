### Client cannot connect to server

### Client is connected but no internet access
1. check input/output traffic
   ```bash
   swanctl --list-sas
   ```
   if no output then no client is connected.

2. check ip forwarding is enabled
    ```bash
   sysctl -a | grep net.ipv4.conf.all.forwarding
    ```
   output should be the same as
    ```text
   net.ipv4.conf.all.forwarding = 1
    ```

3. check MASQUERADING
   ```bash
   iptables -nvL -t nat
   ```
   if output is like
   ```text
    ...
    Chain POSTROUTING (policy ACCEPT 814 packets, 66345 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     all  --  *      ens5    10.10.10.0/24        0.0.0.0/0            policy match dir out pol ipsec
    0     0 MASQUERADE  all  --  *     ens5    10.10.10.0/24        0.0.0.0/0
   ```
   pkts are 0 then check that rule is applied to correct interface
   ```bash
   ifconfig -a | grep flags
   ```
   lo - is loopback.
   ```text
   eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
   lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
   ```
   If your interface is not the same as in the MASQUERADE rule than delete this rule and applied new.
   In current example incorrect interface is ens5. Correct is eth0.
   Delete ens5
   ```bash
     iptables -t nat -D POSTROUTING -s 10.10.10.0/24 -o ens5 -m policy --pol ipsec --dir out -j ACCEPT
     iptables -t nat -D POSTROUTING -s 10.10.10.0/24 -o ens5 -j MASQUERADE
   ```
   check that no POSTROUTING rules exist
   ```bash
   iptables -nvL -t nat
   ```
   output should be like
   ```text
   ...
   Chain POSTROUTING (policy ACCEPT 814 packets, 66345 bytes)
   pkts bytes target     prot opt in     out     source
   ```
   Apply correct rule with eth0
   ```bash
   iptables -t nat -D POSTROUTING -s 10.10.10.0/24 -o eth0 -m policy --pol ipsec --dir out -j ACCEPT
   iptables -t nat -D POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
   iptables -S
   ```

4. trace the traffic
   check which rules are hit by the traffic
   ```bash
   iptables -nvL
   ```
   Output should be like
   ```text
    Chain INPUT (policy ACCEPT 43685 packets, 1885K bytes)
    pkts bytes target     prot opt in     out     source               destination         
    68349   22M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    3806  226K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    24 15488 ACCEPT     udp  --  *      *       0.0.0.0/0            0.0.0.0/0            udp dpt:500
    21 21324 ACCEPT     udp  --  *      *       0.0.0.0/0            0.0.0.0/0            udp dpt:4500
    
    Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    4920 1178K ACCEPT     all  --  *      *       10.10.10.0/24        0.0.0.0/0            policy match dir in pol ipsec proto 50
    5475 6004K ACCEPT     all  --  *      *       0.0.0.0/0            10.10.10.0/24        policy match dir out pol ipsec proto 50
    
    Chain OUTPUT (policy ACCEPT 90431 packets, 18M bytes)
    pkts bytes target     prot opt in     out     source               destination
   ```
   if some rules are not hit by the traffic try to figure out why

5. if you are running in cloud than check that ports are open.
6. in sun_to_moon conf ensure that certificates are copied to each other
