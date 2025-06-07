
```text

| 177.177.177.177 | === | 192.164.45.32 | === | 3.14.76.25 | -- internet
      client                   sun                 moon          
```

## Sun configuration
### System
1. configure system. see [configure system](configure_system.md)
2. sun doesn't require to configure iptables. skipping.
3. generate certs. use sun's static ip and replace <name> with **sun** see [generate certs](generate_certs.md)

### Strongswan

1. clear conf 
   ```bash
   echo > /etc/swanctl/swanctl.conf
   ```

2. Create conf. 
   Replace <sun_public_static_ip> and <moon_public_static_ip> with sun's and moon's public static ips.
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
               id = <sun_public_static_ip>
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
               id = <sun_public_static_ip>
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
   
       sun_to_moon {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           local {
               auth = pubkey
               id = <sun_public_static_ip>
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
   Replace <sun_public_static_ip> and <moon_public_static_ip> with sun's and moon's public static ips.

   ```bash
   cat > /etc/swanctl/swanctl.conf <<EOL
   
   connections {
       moon_to_sun {
           version = 2
           unique=never
           proposals = aes128-sha1-modp1024,aes128-sha1-modp2048,aes256-sha1-modp1024,aes128-sha256-ecp256,aes256-sha384-ecp384
           remote_addrs = <sun_public_static_ip>
           local {
               auth = pubkey
               id = <moon_public_static_ip>
           }
           remote {
               auth = pubkey
               id = <sun_public_static_ip>
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
       rsa-sun_to_moon {
           file = moon_deb.pem
           secret = 
       }
   }
   
   EOL
   ```
   
### exchange certificates between machines
- copy sun's /etc/swanctl/x509/sun_deb.pem to moon's /etc/swanctl/x509/
- copy moon's /etc/swanctl/x509/moon_deb.pem to sun's /etc/swanctl/x509/
Res should be like:
- on sun side
   ```bash
   ls -l /etc/swanctl/x509/
   ```

   ```text
   moon_deb.pem
   sun_client.pem
   sun_deb.pem
   ```

- on moon side
   ```bash
   ls -l /etc/swanctl/x509/
   ```

   ```text
   sun_deb.pem
   moon_client.pem
   moon_deb.pem
   ```

### Start sun strongswan
- see [start strongswan](start_strongswan.md)

### Start moon strongswan
- see [start strongswan](start_strongswan.md)

### check connection between machines
- on sun side
   ```bash
   swanctl --list-sas
   ```
  
   output should be like
   ```text
   suntomoon: #1, ESTABLISHED, IKEv2, fffd03503bf7bd28_i fa0e4e0b0b0f4f29_r*
   local  '192.164.45.32' @ 10.129.4.11[4500]
   remote '3.14.76.25' @ 3.14.76.25[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 1088s ago, rekeying in 12665s
   highway: #1, reqid 1, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
     installed 1088s ago, rekeying in 2153s, expires in 2872s
     in  c9fe6161,    416 bytes,     8 packets,    37s ago
     out c823e9fa,      0 bytes,     0 packets
     local  10.10.10.0/24
     remote 0.0.0.0/0
   ```

- on moon side
   ```bash
   swanctl --list-sas
   ```

  output should be like
   ```text
   moon_to_sun: #1, ESTABLISHED, IKEv2, fffd03503bf7bd28_i* fa0e4e0b0b0f4f29_r
   local  '3.14.76.25' @ 165.22.11.13[4500]
   remote '192.164.45.32' @ 192.164.45.32[4500]
   AES_CBC-128/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
   established 1167s ago, rekeying in 13166s
   highway: #1, reqid 1, INSTALLED, TUNNEL-in-UDP, ESP:AES_CBC-128/HMAC_SHA1_96
     installed 1167s ago, rekeying in 2179s, expires in 2793s
     in  c823e9fa,      0 bytes,     0 packets
     out c9fe6161,    416 bytes,     8 packets,   115s ago
     local  0.0.0.0/0
     remote 10.10.10.0/24
   ```

- configure clients. Clients should connect to SUN. see [client conf](client_conf.md)
