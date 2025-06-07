## Android
### install certificate
1. install 'strongswan VPN Client' via play store
2. copy /etc/swanctl/x509ca/sun_root_ca.pem from sun's machine and send it to your mobile phone by messanger, for example telegram.
3. open telegram on device and press on received certificate.
4. choose 'strongswan VPN Client' and import certificate.

### configure client
1. open strongswan app
2. press 'Add VPN profile'
3. type your sun's public static ip
4. vpn type should be IKEv2 EAP (Username/Password)
5. type username and password created while configuring strongswan conf
6. save

### connect
1. press on profile

