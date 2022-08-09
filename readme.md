# Network UPS Tools for Rock Linux - A Guide
### Diffrences between a debian install and a rhel install
* yum vs dnf
* diffrent install location `/etc/nut/` vs `/etc/ups/`
* firewall ports
* the lack of `a2enmod`

---
### Install Dependencies 
sudo dnf update -y  
sudo dnf config-manager --set-enabled crb  
sudo dnf install epel-release vim usbutils -y  
  
---
### Install NUT and check scanner  
sudo dnf install nut -y  
sudo nut-scanner -U  
```shell
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
  
---
### Configure NUT
sudo vim /etc/ups/ups.conf  
```shell
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
```shell
MONITOR tripplite@localhost 1 admin secret master
```
sudo vim /etc/ups/upsd.conf  
```shell
LISTEN 0.0.0.0 3493 
```
sudo vim /etc/ups/nut.conf  
```shell
MODE=netserver
```
sudo vim /etc/ups/upsd.users  
```shell
[monuser]
  password = secret
  admin master
```
  
---
### Reboot or restart
sudo shutdown -r now  
  
or
  
sudo systemctl start nut-driver  
sudo systemctl start nut-monitor  
sudo systemctl start nut-server  
sudo upsdrvctl stop  
sudo upsdrvctl start  
  
---
### Test NUT
upsc tripplite@localhost  
```shell
battery.charge: 100
battery.runtime: 1485
battery.type: PbAc
battery.voltage: 26.9
battery.voltage.nominal: 24.0
device.mfr: Tripp Lite
device.model: TRIPP LITE UPS
device.serial: FW-2473 A
device.type: ups
driver.name: usbhid-ups
driver.parameter.pollfreq: 30
driver.parameter.pollinterval: 1
driver.parameter.port: auto
driver.parameter.productid: 2009
driver.parameter.synchronous: no
driver.parameter.vendorid: 09ae
driver.version: 2.7.4
driver.version.data: TrippLite HID 0.82
driver.version.internal: 0.41
input.frequency: 59.7
input.voltage: 119.0
input.voltage.nominal: 120
output.frequency.nominal: 60
output.voltage.nominal: 120
ups.beeper.status: enabled
ups.delay.shutdown: 20
ups.mfr: Tripp Lite
ups.model: TRIPP LITE UPS
ups.power.nominal: 1500
ups.productid: 2009
ups.serial: FW-2473 A
ups.status: OL
ups.timer.reboot: 65535
ups.timer.shutdown: 65535
ups.vendorid: 09ae
ups.watchdog.status: 0
```
  
---
### Install CGI Web Application
sudo dnf install httpd nut-cgi -y  
sudo vim /etc/ups/hosts.conf  
```shell
MONITOR tripplite@localhost "Tripp Lite 1500VA SmartUPS - Rack"
```
sudo systemctl restart httpd  
sudo vim /etc/ups/upsset.conf  
```shell
I_HAVE_SECURED_MY_CGI_DIRECTORY
```

---
### Move files from `nut-cgi-bin` to `cgi-bin`
rmdir cgi-bin/  
mv nut-cgi-bin/ cgi-bin  
  
---
### Update Firewall  
sudo firewall-cmd --zone=public --permanent --add-service=http  
sudo firewall-cmd --zone=public --permanent --add-service=https  
sudo firewall-cmd --reload  
  
---
### Visit
http://your.ip.adddress/cgi-bin/upsstats.cgi
  
---
### Sources:
https://docs.technotim.live/posts/NUT-server-guide/  
https://www.youtube.com/watch?v=vyBP7wpN72c  
