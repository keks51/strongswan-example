```text

| 177.177.177.177 | === | 192.164.45.32 | -- internet
      client                   sun          
```

### Sun configuration

## System
1. configure system. see [configure system](configure_system.md)
2. configure iptables. see [configure iptables](configure_ip_tables.md)
3. generate certs. use sun's static ip and replace <name> with **sun** see [generate certs](generate_certs.md)


## Strongswan configuration

1. clear conf 
   ```bash
   echo > /etc/swanctl/swanctl.conf
   ```

2. Create conf. 
   Replace <sun_public_static_ip> with sun's public static ip.
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
   }
   
   secrets {
       eap-<user_name>_clients {
           id = <user_name>
           secret = <user_password>
       }
       rsa-iphone_or_mac {
           file = sun_deb.pem
       }
   }
   EOL
   ```

## Start sun strongswan
- see [start strongswan](start_strongswan.md)
- configure clients. see [client conf](client_conf.md)