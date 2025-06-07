## Start strongswan
- use this command to start and restart
  ```bash
  systemctl restart strongswan
  ```
- check service status
  ```bash
  systemctl status strongswan
  ```
  Output example
  ```text
  ● strongswan.service - strongSwan IPsec IKEv1/IKEv2 daemon using swanctl
  Loaded: loaded (/lib/systemd/system/strongswan.service; enabled; vendor preset: enabled)
  Active: active (running) since Thu 2025-04-03 20:21:51 UTC; 2min 34s ago
  Process: 8490 ExecStartPost=/usr/sbin/swanctl --load-all --noprompt (code=exited, status=0/SUCCESS)
  Main PID: 8473 (charon-systemd)
  Status: "charon-systemd running, strongSwan 5.9.1, Linux 5.10.0-19-amd64, x86_64"
  Tasks: 17 (limit: 1132)
  Memory: 2.8M
  CPU: 91ms
  CGroup: /system.slice/strongswan.service
  └─8473 /usr/sbin/charon-systemd

    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded certificate from '/etc/swanctl/x509ca/sun_root_ca.pem'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded private key from '/etc/swanctl/private/sun_client.pem'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded private key from '/etc/swanctl/private/sun_root_ca.pem'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded private key from '/etc/swanctl/private/sun_deb.pem'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded eap secret 'eap-<user_name>-data'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded pool 'v4_ips'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: successfully loaded 1 pools, 0 unloaded
    Apr 03 20:21:51 sun-vpn swanctl[8490]: loaded connection 'gate2'
    Apr 03 20:21:51 sun-vpn swanctl[8490]: successfully loaded 1 connections, 0 unloaded
    Apr 03 20:21:51 sun-vpn systemd[1]: Started strongSwan IPsec IKEv1/IKEv2 daemon using swanctl.
    ```
    Status should be **Active: active (running)** 
- In case of any errors check logs
  ```bash
  journalctl -eu strongswan.service
  ```
  