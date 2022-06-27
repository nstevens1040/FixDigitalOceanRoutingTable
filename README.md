# FixDigitalOceanRoutingTable
Digital Ocean VPC with pfSense gateway and a single Ubuntu back-end hosting an Nginx web server. This guide is here for my reference because I frequently need to reconfigure the routing table on both virtual machines.  
  
# On pfSense gateway
**show routing table**
```sh
netstat -rn
```
**pfSense routing table should look like this**

|Destination        |Gateway            |Flags     |Netif|
|-------------------|-------------------|----------|-----|  
|default            |161.35.176.1       |UGS      |vtnet0|
|10.0.0.0/16        |link#2             |U        |vtnet1|
|10.0.0.3           |link#2             |UHS         |lo0|
|67.207.67.2        |aa:bf:84:d1:43:a9  |UHS      |vtnet0|
|67.207.67.3        |aa:bf:84:d1:43:a9  |UHS      |vtnet0|
|127.0.0.1          |link#4             |UH          |lo0|
|161.35.176.0/20    |link#1             |U        |vtnet0|
|161.35.185.212     |link#1             |UHS         |lo0|  

**If you don't see the following row in the table, ...**

|Destination        |Gateway            |Flags     |Netif|
|-------------------|-------------------|----------|-----|  
|default            |161.35.176.1       |UGS      |vtnet0|

**then add default route to internet via VPC gateway**
```sh
route add default 161.35.176.1
```

# On backend server
**Show the routing table**
```sh
route -n
```
**The routing table should look like this.**  

|Destination     |Gateway         |Genmask         |Flags |Metric |Ref    |Use |Iface|
|--------------- |--------------  |-------------   |------|-------|-------|----|-----|
|0.0.0.0         |10.0.0.3        |0.0.0.0         |UG    |10     |0        |0 |ens4|
|0.0.0.0         |159.203.96.1    |0.0.0.0         |UG    |100    |0        |0 |ens3|
|10.0.0.0        |0.0.0.0         |255.255.0.0     |U     |1      |0        |0 |ens4|
|10.0.0.0        |0.0.0.0         |255.255.0.0     |U     |10     |0        |0 |ens4|
|159.203.96.0    |0.0.0.0         |255.255.240.0   |U     |100    |0        |0 |ens3|
|169.254.0.0     |0.0.0.0         |255.255.0.0     |U     |1000   |0        |0 |ens3|  

The row where the **Gateway** value is **159.203.96.1** should have a higher **Metric** value than the row where the Gateway value is **10.0.0.3**.  
  
If it does not, then **set the interface metric for ens4 to a lower value to increase it's priority**
```sh
ifmetric ens4 10
```
