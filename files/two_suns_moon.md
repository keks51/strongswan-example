
```text

| 177.177.177.177 | === | 192.164.45.32 | === | 192.165.40.30 | === | 3.14.76.25 | -- internet
      client                  sun1                  sun2                 moon          
```

## Sun1 configuration
### System
1. configure system. see [configure system](configure_system.md)
2. sun1 doesn't require to configure iptables. skipping.
3. generate certs. use sun1's static ip and replace <name> with **sun1** see [generate certs](generate_certs.md)

### Strongswan

1. clear conf
   ```bash
   echo > /etc/swanctl/swanctl.conf
   ```

2. Create conf.
   Replace <sun1_public_static_ip> and <sun2_public_static_ip> with sun1's and sun2's public static ips.
   Replace  <user_name> and <user_password> with creds which will be used to connect
   ```bash
   cat > /etc/swanctl/swanctl.conf <<EOL
   
   pools {
       v4_ips {
           addrs = 10.10.10.0/24
           dns = 8.8.8.8, 8.8.4.4
       }
   }
   
   connections {
       # gate for android, linux or windows clients
       gate {
           version = 2
           send_cert = always
           unique=never
           pools = v4_ips
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384,aes256-sha256-modp2048,aes128gcm16-sha2_256-prfsha256-ecp256
           local_addrs=%any
           local {
               auth = pubkey
               id = <sun1_public_static_ip>
               certs = sun_deb.pem
           }
           remote {
               auth = eap-mschapv2
               eap_id = %any
           }
           children {
               everything {
                   mode = tunnel
                   local_ts = 0.0.0.0/0
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384
               }
           }
       }
   
       # gate for native ipsec clients like iphone
       gate2 {
           version = 2
           send_cert = always
           pools = v4_ips
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384,aes256-sha256-modp2048,aes128gcm16-sha2_256-prfsha256-ecp256
           local_addrs=%any
           local {
               auth = pubkey
               id = <sun1_public_static_ip>
               certs = sun_deb.pem
           }
           remote {
               auth = pubkey
           }
           children {
               everything {
                   mode = tunnel
                   local_ts = 0.0.0.0/0
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384,aes256-sha256,aes256-sha256-modp2048-modpnone,aes128gcm16-sha2_256-ecp256
               }
           }
       }
   
       sun1_to_sun2 {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           local {
               auth = pubkey
               id = <sun1_public_static_ip>
           }
           remote {
               auth = pubkey
               id = <sun2_public_static_ip>
           }
           children {
               highway {
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384
                   mode = tunnel
                   local_ts = 10.10.10.0/24
                   remote_ts = 0.0.0.0/0
               }
           }
       }
   }
   
   secrets {
       eap-<user_name>_clients {
           id = <user_name>
           secret = <user_password>
       }
       rsa-iphone_or_mac_clients {
           file = sun_deb.pem
           secret = 
       }
   }
   EOL
   ```


## Sun2 configuration
### System
1. configure system. see [configure system](configure_system.md)
2. sun2 doesn't require to configure iptables. skipping.
3. generate certs. use sun2's static ip and replace <name> with **sun2** see [generate certs](generate_certs.md)

### Strongswan

1. clear conf
   ```bash
   echo > /etc/swanctl/swanctl.conf
   ```

2. Create conf.
   Replace <sun1_public_static_ip>, <sun2_public_static_ip> and <moon_public_static_ip> with sun1's, sun2's and moon's public static ips.

   ```bash
   cat > /etc/swanctl/swanctl.conf <<EOL
   
   connections {
       sun2_to_sun1 {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           remote_addrs = <sun1_public_static_ip>
           local {
               auth = pubkey
               id = <sun2_public_static_ip>
           }
           remote {
               auth = pubkey
               id = <sun1_public_static_ip>
           }
           children {
               highway {
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384
                   mode = tunnel
                   start_action = start
                   local_ts = 0.0.0.0/0
                   remote_ts = 10.10.10.0/24
               }
           }
       }
   
       sun2_to_moon {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           local {
               auth = pubkey
               id = <sun2_public_static_ip>
           }
           remote {
               auth = pubkey
               id = <moon_public_static_ip>
           }
           children {
               highway {
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384
                   mode = tunnel
                   local_ts = 10.10.10.0/24
                   remote_ts = 0.0.0.0/0
               }
           }
       }
   }
   
   secrets {
       rsa-sun_to_sun2 {
           file = sun2_deb.pem
           secret = 
       }
   }
   
   EOL
   ```

## Moon configuration
### System
1. configure system. see [configure system](configure_system.md)
2. configure iptables. see [configure iptables](configure_ip_tables.md)
3. generate certs. use moon's static ip and replace <name> with **moon** see [generate certs](generate_certs.md)

### Strongswan

1. clear conf
   ```bash
   echo > /etc/swanctl/swanctl.conf
   ```

2. Create conf.
   Replace <sun2_public_static_ip> and <moon_public_static_ip> with sun2's and moon's public static ips.

   ```bash
   cat > /etc/swanctl/swanctl.conf <<EOL
   
   connections {
       moon_to_sun2 {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           remote_addrs = <sun2_public_static_ip>
           local {
               auth = pubkey
               id = <moon_public_static_ip>
           }
           remote {
               auth = pubkey
               id = <sun2_public_static_ip>
           }
           children {
               highway {
                   esp_proposals = aes128-sha1,aes256-sha1,aes128gcm128-ecp256,aes256gcm128-ecp384
                   mode = tunnel
                   start_action = start
                   local_ts = 0.0.0.0/0
                   remote_ts = 10.10.10.0/24
               }
           }
       }
   }
   
   secrets {
       rsa-sun2_to_moon {
           file = moon_deb.pem
           secret = 
       }
   }
   
   EOL
   ```

### exchange certificates between machines
- copy sun1's /etc/swanctl/x509/sun1_deb.pem to sun2's /etc/swanctl/x509/
- copy sun2's /etc/swanctl/x509/sun2_deb.pem to sun1's /etc/swanctl/x509/
- copy sun2's /etc/swanctl/x509/sun2_deb.pem to moon's /etc/swanctl/x509/
- copy moon's /etc/swanctl/x509/moon_deb.pem to sun2's /etc/swanctl/x509/

Res should be like:
   - on sun1 side
      ```bash
      ls -l /etc/swanctl/x509/
      ```
   
      ```text
      sun2_deb.pem
      sun_client.pem
      sun_deb.pem
      ```
   
   - on sun2 side
      ```bash
      ls -l /etc/swanctl/x509/
      ```
   
      ```text
      moon_deb.pem
      sun1_deb.pem
      sun2_client.pem
      sun2_deb.pem
      ```

   - on moon side
     ```bash
     ls -l /etc/swanctl/x509/
     ```

     ```text
     moon_client.pem
     moon_deb.pem
     sun2_deb.pem
     ```

### Start sun1 strongswan
- see [start strongswan](start_strongswan.md)

### Start sun2 strongswan
- see [start strongswan](start_strongswan.md)

### Start moon strongswan
- see [start strongswan](start_strongswan.md)

### check connection between machines
- on sun1 side
   ```bash
   swanctl --list-sas
   ```

  output should be like
   ```text
   sun1_to_sun2: #8, ESTABLISHED, IKEv2, d707546264c4d0f8_i* 677501a24ce75ae2_r
   local  '192.164.45.32' @ 12.124.40.21[4500]
   remote '192.165.40.30' @ 192.165.40.30[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 1817s ago, rekeying in 12162s
   highway: #11, reqid 1, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
     installed 2164s ago, rekeying in 1229s, expires in 1796s
     in  c1cf02b4,    873 bytes,     9 packets,  1379s ago
     out ce39bde8,      0 bytes,     0 packets
     local  10.10.10.0/24
     remote 0.0.0.0/0


   ```

- on sun2 side
   ```bash
   swanctl --list-sas
   ```

  output should be like
   ```text
   sun2_to_sun: #10, ESTABLISHED, IKEv2, d707546264c4d0f8_i 677501a24ce75ae2_r*
   local  '192.165.40.30' @ 14.132.20.31[4500]
   remote '192.164.45.32' @ 192.164.45.32[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 1984s ago, rekeying in 12123s
   highway: #9, reqid 1, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
     installed 2331s ago, rekeying in 1073s, expires in 1629s
     in  ce39bde8,      0 bytes,     0 packets
     out c1cf02b4,    873 bytes,     9 packets,  1545s ago
     local  0.0.0.0/0
     remote 10.10.10.0/24
   sun2_to_moon: #9, ESTABLISHED, IKEv2, bed69916ce461acf_i 739f7ef93d6c7487_r*
   local  '192.165.40.30' @ 14.132.20.31[4500]
   remote '3.14.76.25' @ 3.14.76.25[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 2209s ago, rekeying in 12065s
   highway: #10, reqid 2, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
   installed 1897s ago, rekeying in 1615s, expires in 2063s
   in  cf2134b4,    873 bytes,     9 packets,  1545s ago
   out c4b90ea6,      0 bytes,     0 packets
   local  10.10.10.0/24
   remote 0.0.0.0/0
   ```

- on moon side
   ```bash
   swanctl --list-sas
   ```

  output should be like
   ```text
   moon_to_sun2: #3, ESTABLISHED, IKEv2, bed69916ce461acf_i* 739f7ef93d6c7487_r
   local  '3.14.76.25' @ 171.22.30.220[4500]
   remote '192.165.40.30' @ 192.165.40.30[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 2254s ago, rekeying in 11814s
   highway: #5, reqid 1, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
     installed 1942s ago, rekeying in 1364s, expires in 2018s
     in  c4b90ea6,      0 bytes,     0 packets
     out cf2134b4,    873 bytes,     9 packets,  1590s ago
     local  0.0.0.0/0
     remote 10.10.10.0/24
   ```

- configure clients. Clients should connect to SUN. see [client conf](client_conf.md)
