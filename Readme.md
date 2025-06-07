## Example of several strongswan configurations


### One server 

```text
| 177.177.177.177 | === | 192.164.45.32 | -- internet
     client                   sun          
```

see [sun](files/sun.md)

### Two servers 

```text

| 177.177.177.177 | === | 192.164.45.32 | === | 3.14.76.25 | -- internet
      client                   sun                 moon          
```

see [sun_moon](files/sun_moon.md)

to route traffic (which is sent to specific ip) directly from sun instead of forwarding to moon see [masquerading](files/masquarading.md).  
for example request to https://whatismyipaddress.com/ should be sent from sun. 

### Three servers

```text

| 177.177.177.177 | === | 192.164.45.32 | === | 192.165.40.30 | === | 3.14.76.25 | -- internet
      client                  sun1                  sun2                 moon          
```

see [two sun_moon](files/two_suns_moon.md)

to route traffic (which is sent to specific ip) directly from sun instead of forwarding to moon see [masquerading](files/masquarading.md).  
for example request to https://whatismyipaddress.com/ should be sent from sun.


### Troubleshooting see
[troubleshooting](files/troubleshooting.md)
