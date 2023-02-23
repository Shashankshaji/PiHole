# Docker Pi-hole

<p align="center">
<a href="https://pi-hole.net"><img src="https://pi-hole.github.io/graphics/Vortex/Vortex_with_text.png" width="150" height="255" alt="Pi-hole"></a><br/>
</p>
<!-- Delete above HTML and insert markdown for dockerhub : ![Pi-hole](https://pi-hole.github.io/graphics/Vortex/Vortex_with_text.png) -->

- This can be run on cloud like **AWS** or locally in **Proxmox**
- Here I have used Ubuntu Server in AWS
 ### The Commands will be
```
sudo systemctl stop systemd-resolved.service
sudo systemctl disable systemd-resolved.service
```
**Now When you try to ping google.com it doesn't work because we disabled the dns service**

### Now we'll edit the resolver files
```
sudo nano /etc/resolv.conf
```
- Now in the file we chnage the nameserver from whatever is given to 8.8.8.8 and rest remains the same**
```
nameserver 8.8.8.8
```
Press Ctrl+X to save
- Now when you try to ping google it'll work
- Now we'll update our repositories using
```
sudo apt update
```
### Next we'll install docker
```
sudo apt install docker.io
```
### We'll create and  open a new file pihole.sh
```
sudo nano pihole.sh
```
## In the file paste the below script
```
#!/bin/bash

# https://github.com/pi-hole/docker-pi-hole/blob/master/README.md

PIHOLE_BASE="${PIHOLE_BASE:-$(pwd)}"
[[ -d "$PIHOLE_BASE" ]] || mkdir -p "$PIHOLE_BASE" || { echo "Couldn't create storage directory: $PIHOLE_BASE"; exit 1; }

# Note: FTLCONF_LOCAL_IPV4 should be replaced with your external ip.
docker run -d \
    --name pihole \
    -p 53:53/tcp -p 53:53/udp \
    -p 80:80 \
    -e TZ="America/Chicago" \
    -v "${PIHOLE_BASE}/etc-pihole:/etc/pihole" \
    -v "${PIHOLE_BASE}/etc-dnsmasq.d:/etc/dnsmasq.d" \
    --dns=127.0.0.1 --dns=1.1.1.1 \
    --restart=unless-stopped \
    --hostname pi.hole \
    -e VIRTUAL_HOST="pi.hole" \
    -e PROXY_LOCATION="pi.hole" \
    -e FTLCONF_LOCAL_IPV4="127.0.0.1" \
    pihole/pihole:latest

printf 'Starting up pihole container '
for i in $(seq 1 20); do
    if [ "$(docker inspect -f "{{.State.Health.Status}}" pihole)" == "healthy" ] ; then
        printf ' OK'
        echo -e "\n$(docker logs pihole 2> /dev/null | grep 'password:') for your pi-hole: http://${IP}/admin/"
        exit 0
    else
        sleep 3
        printf '.'
    fi

    if [ $i -eq 20 ] ; then
        echo -e "\nTimed out waiting for Pi-hole start, consult your container logs for more info (\`docker logs pihole\`)"
        exit 1
    fi
done;
```
### Changing the attribute of the file to be executable
```
sudo chmod u+x pihole.sh
```
### To launch the script
```
sudo ./pihole.sh
```
**It'll provide you with a password to login after installing**

### Open web browser and type your ipaddress along with /admin. For example
```
192.168.1.3/admin
```
- Login with the given password
## Now to change the password
```
sudo docker exec -it pihole bash
```
- Now your are in
- To change password 
```
pihole -a -p
```
- Type your new password and login 
