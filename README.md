# Block IPs on Unifi controllers

## Distributed under MIT license

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Project Notes
**Author:** Carlos Talbot (@tusc69 on ubnt forums)

# About

This docker container allows you to block and unblock IP addresses from your Unifi firewall list at certain times during the day. This can be useful when you have kids in
the household and you want to restrict their Internet usage. Using the great php client class from malle-pietje ( https://github.com/Art-of-WiFi/UniFi-API-client ),
this script will automatically add and delete IP addresses from a firewall group defined on your Unifi controller. This firewall group has a predefined rule in the "WAN OUT" section to automatically
drop traffic from any IP on the list.

The script makes Unifi RESTful API calls to dynamically update the firewall group with IPs you pass to it. The scheduler is esentially cron.

# Preparation

The docker containter can run on a Linux amd64 system or directly on a UDM/UDM Pro. Before you pull down the image you'll need to grab a couple of files first. For the example
below we will be running directly on a UDM which explains the paths. If running on a Linux amd64 server just replace /mnt/data with a persistent path on your server.

```
mkdir -p /mnt/data/blockips-unifi
curl -Lo /mnt/data/blockips-unifi/config.php https://github.com/tusc/blockips-unifi/blob/master/config.php?raw=true
curl -Lo /mnt/data/blockips-unifi/crontab https://github.com/tusc/blockips-unifi/blob/master/crontab?raw=true
```
You then need to log into the Unifi controller, under classic settings go to Routing and Firewall, Firewall, Groups. Create a new group called "Block_Group". 
![Blockgroup](/images/blockgrp.png)
You need to populate it with at least one fake IP address as you cannot have empty firewall groups. Pick an address you don't use, for example an RFC1918 address not part of your subnet.

![Blockgroup](/images/blockgrp2.png)

Now go to Firewall, Rule IPv4, WAN OUT. Create a new firewall rule.

![Blockgroup](/images/firewall1.png)

 You can call it whatever you want, as long as it references the firewall block group we previously created. Make sure you select the Drop Action and select the Block_Group at the
 bottom under Source. Leave everything else as the default.
 
 ![Blockgroup](/images/firewall2.png)
 
 At this point we're ready proceed to configuring the files we downloaded at the beginning of this section. Using you preferred editor go ahead and open up config.php. 
 You'll need to populate your unifi credentials for the controller (controlleruser, controllerpassword) as well as the url (controllerurl).
 
 In addition, make sure the site name is correct along with the firewall group name and firewall rule name. If you used the defaults above for the firewall group and firewawll rule then
 there's no need to change the values.
 
 Finally, we need to edit the crontab file that schedules when the scripts run to add and remove IPs from the firewall group. You should see two examples for addidtions and deletions.
 In the next section you'll want to make sure you specify your local timezone as this reflects when the cron jobs will run. Replace the example IP addresses in the crontab file with the ones
 you want to block at the designated time.

# Installing

In order to start the containter you will need to type the  command below: <br/>
**NOTE** Make sure to update the timezone variable TZ to your local timezone. You can find a list here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

```
docker run -it -d --name blockips-unifi  -e "TZ=America/Chicago" \
   -v /mnt/data/blockips-unifi/config.php:/config.php \
   -v /mnt/data/blockips-unifi/crontab:/etc/crontabs/root \
  tusc/blockips-unifi:latest
```
This will download the latest image to your server.

If you're running this containter on a UDM and have you reboot, you can manually restart the containter with the commmand below:
```
docker start blockips-unifi
```
Fortunately you can also take advantage of boostchicken's great tool to automatically start a Docker container after a reboot:
https://github.com/boostchicken/udm-utilities/tree/master/on-boot-script

If you're interested in compiling your own version I have a Dockerfile available here: https://github.com/tusc/blockips-unifi/blob/master/Dockerfile

# Validating configuration

You can test your credentials are valid by adding a test IP to the block list (container needs to be up and running):

```
docker exec -it blockips-unifi php /add_block_firewall.php 192.168.200.10
```
Watch the firewall group page on the Unifi controller and you should see the count go up by one. Try to access the Internet with that address.

Conversly, removing the IP from the list can be done with this command:

```
docker exec -it blockips-unifi php /del_block_firewall.php 192.168.200.10
```

# Uninstalling

To remove the docker instance and image you'll need to type the following at the UDM ssh prompt:


```
docker stop blockips-unifi
docker rm blockips-unifi
docker rmi docker.io/tusc/blockips-unifi
```
