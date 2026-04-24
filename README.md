# WAF-SIEM-Integration-Lab

- `Creat by mohammad zisadul islam`
- `Update : 21/02/2026`
- `IDS&IPS Setup and integration with wazuh`

<p>
Here, one gets to see how a complete WAF solution can be deployed with NGINX, ModSecurity, OWASP CRS, and Wazuh integration. Here you get to know about the whole process of setting up, configuring, monitoring logs and performing tests by simulating real-world web attacks. This is a demonstration of how we detect, stop and analyze web attacks with WAF rules and SIEM monitoring. This is aimed at security professionals who need experience in cybersecurity and WAF. This is an end-to-end implementation exercise. Educational purpose only.
</p>

##  Working in NGINX + MODSECURITY + OWASP CRS + WAZUH INTEGRATION

*This section contains instructions on how to install ModSecurity, which is an open source Web Application Firewall (WAF). This is used to protect websites from attacks using various techniques. The process involves setting up dependencies, installing ModSecurity, configuring ModSecurity, and integrating it into different servers such as NGINX and Apache.*


### 1 ModSecurity Installation:

```
apt update
```
```
apt install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev
```
```
sudo apt-get install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev
```
```
git clone https://github.com/owasp-modsecurity/ModSecurity
```
```
cd ModSecurity/
```
```
git submodule init
```
```
git submodule update
```
```
sh build.sh
```
```
./configure --with-pcre2
```
```
make
```
```
make install
```

```
cp /path/to/ModSecurity/modsecurity.conf-recommended /usr/local/modsecurity/modsecurity.conf
```
```
cp /path/to/ModSecurity/unicode.mapping /usr/local/modsecurity/unicode.mapping
```


### File parth 
```
sudo nano /usr/local/modsecurity/modsecurity.conf
```


### 2 Installation Nginx 

`This section provides a step-by-step guide to installing Nginx, a high-performance web server and reverse proxy. It covers package installation, basic configuration, and service management. Useful for beginners and professionals setting up secure and efficient web servers. For educational purposes only.`


#### Install ModSecurity for Nginx

```
apt update
```
```
apt install libmodsecurity3 libmodsecurity-dev
```
```
apt install nginx
 ```
```
apt install nginx-module-modsecurity
 ```

## Addet file parth on top bottom 
-  File Parth

```
nano /etc/nginx/nginx.conf
```
##### Add
```
load_module modules/ngx_http_modsecurity_module.so;
```

##### Enable ModSecurity in Nginx:  Inside http {} block add:
`- modsecurity on;`
`-modsecurity_rules_file /etc/nginx/modsec/main.conf;`


### 3 Create ModSecurity Base Config

```
sudo mkdir -p /etc/nginx/modsec
```
```
/etc/nginx/modsec/modsecurity-base.conf
```

### Configure fiel addite
`All in one file parth /modsecurity-base.conf`
```
 SecRuleEngine On
 SecRequestBodyAccess On
 SecAuditEngine RelevantOnly
 SecAuditLog /var/log/modsecurity/audit.log
 SecAuditLogParts ABIJDEFHZ (no addite )
```


### 4 Install OWASP CRS
```
cd /opt
git clone https://github.com/coreruleset/coreruleset.git
mv coreruleset /opt/coreruleset
cp /opt/coreruleset/crs-setup.conf.example /opt/coreruleset/crs-setup.conf
```

### Create Main ModSecurity Rule Loader

```
nano  /etc/nginx/modsec/main.conf
```

### Addet in rules file

`All in one Rules `

```
 Include /etc/nginx/modsec/modsecurity-base.conf
 Include /opt/coreruleset/crs-setup.conf
 Include /opt/coreruleset/rules/*.conf
```

### 5 Fix SecDefaultAction Error

```SecDefaultActions can only be placed once per phase```

```grep -n SecDefaultAction /opt/coreruleset/crs-setup.conf```

### Addet and addite this parth 
- That is Rules 
```
SecDefaultAction "phase:1,log,auditlog,pass"
SecDefaultAction "phase:2,log,auditlog,pass"
```


### 6 Enable Blocking Mode
```/etc/nginx/modsec/modsecurity-base.conf```

*ON tihis Secctuion*

`SecRuleEngine On`

  ### Chack status and test for nginx 2.28.4v
```
nginx -t
systemctl reload nginx
```



  ##### 6 Test WAF Blocking with Tarminal
```curl http://Your IP/```
```curl "http://192.168.3.1/?id=1%20OR%201=1"```

##### Check audit log
```tail -f /var/log/modsecurity/audit.log```

##### Fix Paranoia Level
```/opt/coreruleset/crs-setup.conf```

- *Uncomment and set*
```
SecAction \
"id:900000,\
phase:1,\
pass,\
t:none,\
nolog,\
setvar:tx.blocking_paranoia_level=1"

SecAction \
"id:900001,\
phase:1,\
pass,\
t:none,\
nolog,\
setvar:tx.detection_paranoia_level=1"
```

#### Add audit.log to Wazuh agent

```nano /var/ossec/etc/ossec.conf```

##### Add inside <ossec_config>

```
<localfile>
 <log_format>syslog</log_format>
 <location>/var/log/modsecurity/audit.log</location>
</localfile>
```

- *Permission*
```
chown root:wazuh /var/log/modsecurity/audit.log
chmod 640 /var/log/modsecurity/audit.log
systemctl restart wazuh-agent
```

#### Chack log
```grep modsecurity /var/ossec/logs/ossec.log```

## 7 Create Custom Decoder (Manager Side)

```nano /var/ossec/etc/decoders/local_decoder.xml```

- *Add*

```
<decoder name="modsecurity">
 <prematch>ModSecurity:</prematch>
</decoder>
```

- <p>Create Custom Rules (Attack Classification)</p>

```nano /var/ossec/etc/rules/local_rules.xml```

  #### List site addet rules
```
<group name="modsecurity,web,">

  <rule id="110100" level="15">
    <decoded_as>modsecurity</decoded_as>
    <match>942100</match>
    <description>SQL Injection Attack Detected</description>
  </rule>

  <rule id="110101" level="14">
    <decoded_as>modsecurity</decoded_as>
    <match>941</match>
    <description>XSS Attack Detected</description>
  </rule>

  <rule id="110102" level="14">
    <decoded_as>modsecurity</decoded_as>
    <match>930</match>
    <description>Path Traversal / LFI Attack Detected</description>
  </rule>

  <rule id="110199" level="10">
    <decoded_as>modsecurity</decoded_as>
    <description>Other ModSecurity Event</description>
  </rule>

</group>
```
*Restart Your Server*
```systemctl restart wazuh-manager```

#### 8 Testing

- SQLi

```curl "http://192.168.33.140/?id=1 OR 1=1"```

- XSS
 
```curl "http://192.168.33.140/?q=<script>alert(1)</script>"```

- LFI

```curl "http://192.168.33.140/?file=../../../../etc/passwd"```

#### All chack is "OK"  {Test the Machin} 
```
grep "rules loaded" /var/log/nginx/error.log
grep SecRuleEngine /etc/nginx/modsec/modsecurity-base.conf
grep SecRuleEngine /etc/nginx/modsec/modsecurity-base.conf

```

### 9 CRS rules file parth

```nano /opt/coreruleset/crs-setup.conf```
```sudo find / -type f -name "crs-setup.conf*" 2>/dev/null```

####  Addtable file 

```nano /etc/nginx/nginx.conf```
```nano /usr/local/modsecurity/modsecurity.conf```

- Nginx variation hiding contest 

``` nano /etc/nginx/sites-available/default```

`Add`
```
error_page 403 /custom_403.html;
error_page 404 /custom_404.html;
error_page 500 502 503 504 /custom_50x.html;

location = /custom_403.html {
root /var/www/errors;
internal;
}

location = /custom_404.html {
root /var/www/errors;
internal;
}

location = /custom_50x.html {
root /var/www/errors;
internal;
}
```



### 10 Adite file

- Cerat a file parth
```
sudo mkdir -p /var/www/errors
sudo nano /var/www/errors/custom_403.html
```

- Add  
```
<!DOCTYPE html>
<html>
<head>
<title>Forbidden</title>
</head>
<body>
<h1>403 Forbidden</h1>
<p>Access denied.</p>
</body>
</html>
```



### 11 Save and  restart nginx server and test this server 

```nikto -h http://IP```
```
sqlmap -u "http://192.168.33.140/?id=1" --batch
for i in {1..50}; do curl http://IP/?id=1%20OR%201=1; done
```


#### Browser Testing For blocking Attack 

- Sql injection 
```www.exmapole.com/?id=1 OR 1=1```

- Xss attack 
```?q=<script>alert(1)</script>```

- LFI attack
```?q=<script>alert(1)</script>```


- DDoS Attack 
```hping3 -S –flood -p 80 192.168.33140```

#### 12 Manully Log chack

``` sudo tail -f /var/log/nginx/error.log```

```sudo tail -f /var/log/nginx/access.log```

```sudo tail -f /var/log/modsecurity/audit.log```

```sudo tail -f /var/log/nginx/access.log | grep 403```

```sudo less /var/log/nginx/error.log```

```sudo tail -f /var/log/nginx/error.log /var/log/modsec_audit.log```






























