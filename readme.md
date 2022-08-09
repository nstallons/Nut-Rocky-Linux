# Network UPS Tools for Rock Linux - A Guide

### Install Dependencies 
sudo dnf update -y  
sudo dnf config-manager --set-enabled crb  
sudo dnf install epel-release vim usbutils -y  

### Install NUT and check scanner
sudo dnf install nut -y  
sudo nut-scanner -U  
```
[nutdev1]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "09AE"
        productid = "2009"
        product = "TRIPP LITE UPS"
        serial = "FW-2473 A"
        vendor = "Tripp Lite"
        bus = "002"
```
### Configure NUT
sudo vim /etc/ups/ups.conf  
```
pollinterval = 1
maxretry = 3
[tripplite]
        driver = usbhid-ups
        port = auto
        desc = "Tripp Litt 1500VA SmartUPS"
        vendorid = 09ae
        productid = 2009
```
sudo vim /etc/ups/upsmon.conf  
```
MONITOR tripplite@localhost 1 admin secret master
```
sudo vim /etc/ups/upsd.conf  
```
LISTEN 0.0.0.0 3493 
```
sudo vim /etc/ups/nut.conf  
```
MODE=netserver
```
sudo vim /etc/ups/upsd.users  
```
[monuser]
  password = secret
  admin master
```
sudo shutdown -r now  
  
or
  
sudo systemctl start nut-driver  
sudo systemctl start nut-monitor  
sudo systemctl start nut-server  
sudo upsdrvctl stop  
sudo upsdrvctl start  
  
upsc tripplite@localhost  
  
sudo dnf install httpd nut-cgi -y  
sudo vim /etc/ups/hosts.conf  
```
MONITOR tripplite@localhost "Tripp Lite 1500VA SmartUPS - Rack"
```
sudo systemctl restart httpd  
sudo vim /etc/ups/upsset.conf  
```
I_HAVE_SECURED_MY_CGI_DIRECTORY
```
rmdir cgi-bin/  
mv nut-cgi-bin/ cgi-bin  

sudo firewall-cmd --zone=public --permanent --add-service=http  
sudo firewall-cmd --zone=public --permanent --add-service=https  
sudo firewall-cmd --reload  

https://docs.technotim.live/posts/NUT-server-guide/  
https://www.youtube.com/watch?v=vyBP7wpN72c  
