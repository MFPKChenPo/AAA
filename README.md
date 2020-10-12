# ONOS AAA Application
## Environment
* OS: Ubuntu 16.04
* ONOS: 2.2.2
* Bazel: 1.1.0
* Java 11
## Install FreeRadius 
* You can either import the existing docker image or from scratch to build the freeradius server environment

### Import the docker image from Docker hub
```bash=
docker pull gary199722/freeradius_server
docker run -it --rm --privileged gary199722/freeradius_server /bin/bash
(docker)$ freeradius -X
```
### From scratch
* If you import the radius server docker image from Docker hub already, you can skip this step and forward to Testing Steps.
---
1. Install FreeRadius in a docker container
```bash=
docker run -it --privileged --rm  ubuntu:16.04 /bin/bash
sudo apt install -y freeradius freeradius-utils vim 
```

2. Add a supplicant credential (in radius server)
> /etc/freeradius/users
```bash=
# supplicant credential file
"winlab" Cleartext-Password := "test"
           Reply-Message = "Hello, %{User-Name}"
```

3. Edit NAS information (in radius server which is in the Docker)
> /etc/freeradius/client.conf
```bash=
# Authenticator information file
client dockernet {
    # Authenticator IP Address                
    ipaddr = 172.17.0.0
    # Authenticator Password
    secret = winlab_radius_share_secret
    netmask = 24
}
```
4. Test the changes are working well in radius server(need to reboot)
```bash=
sudo radtest winlab test 172.17.0.2 0 winlab_radius_share_secret
```
5. Opening the freeradius server in debug mode
```bash=
freeradius -X
```
## Testing steps
1. Start ONOS
```bash=
ok clean
```
2. Install sadis & aaa & push aaa_netcfg
```bash=
git clone https://github.com/opencord/aaa.git
git clone https://github.com/opencord/sadis.git
git clone https://github.com/MFPKChenPo/AAA.git
```
```bash=
cd ~/sadis && git checkout 5.1.0
cd ~/aaa && git checkout 2.1.0
cd ~/AAA && mci -DskipTests  # Compile aaafwd
cd ~/sadis/app && mci        # Compile sadis
cd ~/aaa/app && mci          # Compile aaa
onos-netcfg localhost ~/AAA/onos-dhcp.json
onos-netcfg localhost ~/AAA/aaa-conf.json
onos-app localhost install! ~/sadis/app/target/sadis-app-5.1.0-SNAPSHOT.oar
onos-app localhost install! ~/aaa/app/target/aaa-app-2.1.0-SNAPSHOT.oar
onos-app localhost install! ~/AAA/app/target/aaafwd-1.0-SNAPSHOT.oar
```
> aaa-conf.json
```json=
{
    "apps": {
        "org.opencord.aaa":{
            "AAA":{
                "radiusSecret": "winlab_radius_share_secret",
                "radiusIp": "172.17.0.2",
                "radiusServerPort": "1812"
            }
        }
    }
}
```
3. Start freeradius server (In the docker container)
```bash=
docker run -it --rm --privileged gary199722/freeradius_server /bin/bash
(docker)$ freeradius -X
```


---
## ONOS AAA fwd
[Source code](https://github.com/MFPKChenPo/AAA/blob/master/src/main/java/nctu/winlab/aaafwd/AppComponent.java)

