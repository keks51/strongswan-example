## Generate root, debian and client certificates.

- go to cert dir
  ```bash
  cd /etc/swanctl/
  ```

- generate root ca. replace <public_static_ip> and <machine_name> with machine's static ip and name
  ```bash
  ipsec pki --gen --type rsa --size 4096 --outform pem > private/<machine_name>_root_ca.pem
  ```
  ```bash
  ipsec pki --self --ca --lifetime 3650 --in private/<machine_name>_root_ca.pem \
  --type rsa --digest sha256 \
  --dn "CN=<public_static_ip>" \
  --outform pem > x509ca/<machine_name>_root_ca.pem
  ```
  (DO NOT COPY BELOW) For EXAMPLE the command should be like:
  ```text
  ipsec pki --gen --type rsa --size 4096 --outform pem > private/sun_root_ca.pem
   
  ipsec pki --self --ca --lifetime 3650 --in private/sun_root_ca.pem \
  --type rsa --digest sha256 \
  --dn "CN=192.164.45.32" \
  --outform pem > x509ca/sun_root_ca.pem
  ```

- generate host ca. replace <public_static_ip> and <machine_name> with machine's static ip and name
  ```bash
  ipsec pki --gen --type rsa --size 4096 --outform pem > private/<machine_name>_deb.pem
  ```
  ```bash
  ipsec pki --pub --in private/<machine_name>_deb.pem --type rsa |
  ipsec pki --issue --lifetime 3650 --digest sha256 \
  --cacert x509ca/<machine_name>_root_ca.pem --cakey private/<machine_name>_root_ca.pem \
  --dn "CN=<public_static_ip>" \
  --san <public_static_ip> \
  --flag serverAuth --outform pem > x509/<machine_name>_deb.pem
  ```
- generate client ca. Replace <machine_name> with machine's name. Execute as it is.
  ```bash
  ipsec pki --gen --type rsa --size 4096 --outform pem > private/<machine_name>_client.pem
  ```
  ```bash
  ipsec pki --pub --in private/<machine_name>_client.pem --type rsa |
  ipsec pki --issue --lifetime 3650 --digest sha256 \
  --cacert x509ca/<machine_name>_root_ca.pem --cakey private/<machine_name>_root_ca.pem \
  --dn "CN=<machine_name>_client" --san <machine_name>_client \
  --flag clientAuth \
  --outform pem > x509/<machine_name>_client.pem
  ```
