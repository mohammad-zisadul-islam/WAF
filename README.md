# WAF-SIEM-Integration-Lab


Here, one gets to see how a complete WAF solution can be deployed with NGINX, ModSecurity, OWASP CRS, and Wazuh integration. Here you get to know about the whole process of setting up, configuring, monitoring logs and performing tests by simulating real-world web attacks.

This is a demonstration of how we detect, stop and analyze web attacks with WAF rules and SIEM monitoring. This is aimed at security professionals who need experience in cybersecurity and WAF.

This is an end-to-end implementation exercise. Educational purpose only.


#  NGINX + MODSECURITY + OWASP CRS + WAZUH INTEGRATION #

This section contains instructions on how to install ModSecurity, which is an open source Web Application Firewall (WAF). This is used to protect websites from attacks using various techniques. The process involves setting up dependencies, installing ModSecurity, configuring ModSecurity, and integrating it into different servers such as NGINX and Apache.


# 1 ModSecurity Installation:

<pre> $ apt update </pre>
<pre> $ apt install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev </pre>

<pre> $ sudo apt-get install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev </pre>
<pre> $  git clone https://github.com/owasp-modsecurity/ModSecurity </pre>
<pre> $  cd ModSecurity/ </pre>
<pre> $  git submodule init </pre>
<pre> $  git submodule update </pre>
<pre> $  sh build.sh </pre>
<pre> $  ./configure --with-pcre2 </pre>
<pre> $  make </pre>
<pre> $  make install </pre>

<pre> $ cp /path/to/ModSecurity/modsecurity.conf-recommended /usr/local/modsecurity/modsecurity.conf </pre>
<pre> $ cp /path/to/ModSecurity/unicode.mapping /usr/local/modsecurity/unicode.mapping </pre>


# File parth : 
<pre> $ sudo nano /usr/local/modsecurity/modsecurity.conf </pre>


# 2 Installation Nginx 

This section provides a step-by-step guide to installing Nginx, a high-performance web server and reverse proxy. It covers package installation, basic configuration, and service management. Useful for beginners and professionals setting up secure and efficient web servers. For educational purposes only.


* Install ModSecurity for Nginx

<pre> $ apt update </pre>
<pre> $ apt install libmodsecurity3 libmodsecurity-dev </pre>
<pre> $ apt install nginx </pre>
<pre> $ apt install nginx-module-modsecurity </pre>


# Addet file parth on top bottom 
* File Parth 
<pre> $ nano /etc/nginx/nginx.conf </pre>
* Add : 
<pre> $ load_module modules/ngx_http_modsecurity_module.so; </pre>

# Enable ModSecurity in Nginx:  Inside http {} block add:
* modsecurity on;
* modsecurity_rules_file /etc/nginx/modsec/main.conf;


# 3 Create ModSecurity Base Config: 

<pre> $  sudo mkdir -p /etc/nginx/modsec </pre>
<pre> $ /etc/nginx/modsec/modsecurity-base.conf </pre>

# Configure fiel addite : 
All in one file parth /modsecurity-base.conf

* SecRuleEngine On
* SecRequestBodyAccess On
* SecAuditEngine RelevantOnly
* SecAuditLog /var/log/modsecurity/audit.log
* SecAuditLogParts ABIJDEFHZ (no addite )



# 4 Install OWASP CRS:

<pre> $ cd /opt </pre>
<pre> $ git clone https://github.com/coreruleset/coreruleset.git </pre>
<pre> $ mv coreruleset /opt/coreruleset </pre>
<pre> $ cp /opt/coreruleset/crs-setup.conf.example /opt/coreruleset/crs-setup.conf </pre>


# Create Main ModSecurity Rule Loader:

<pre> $ nano  /etc/nginx/modsec/main.conf </pre>

# Addet in rules file:

 All in one Rules 
* Include /etc/nginx/modsec/modsecurity-base.conf
* Include /opt/coreruleset/crs-setup.conf
* Include /opt/coreruleset/rules/*.conf

# 5 Fix SecDefaultAction Error

SecDefaultActions can only be placed once per phase

<pre> $ grep -n SecDefaultAction /opt/coreruleset/crs-setup.conf </pre>

# Addet and addite this parth 
That is Rules 

* SecDefaultAction "phase:1,log,auditlog,pass"
* SecDefaultAction "phase:2,log,auditlog,pass"

  # 6 Enable Blocking Mode:
<pre> $ /etc/nginx/modsec/modsecurity-base.conf </pre>

ON tihis Secctuion 

* SecRuleEngine On

  # Chack status and test for nginx 2.28.4v
<pre> $ nginx -t </pre>
<pre> $ systemctl reload nginx </pre>



  # 6 Test WAF Blocking:
<pre> $ curl http://Your IP/ </pre>
<pre> $ curl "http://192.168.3.1/?id=1%20OR%201=1" </pre>

  # Check audit log::
<pre> $ tail -f /var/log/modsecurity/audit.log </pre>

  # Fix Paranoia Level:
<pre> $  /opt/coreruleset/crs-setup.conf </pre>
# Uncomment and set:
* SecAction \
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
setvar:tx.detection_paranoia_level=1"  *

  # Add audit.log to Wazuh agent:

<pre> $ nano /var/ossec/etc/ossec.conf </pre>

  # Add inside <ossec_config>:

<localfile>
 <log_format>syslog</log_format>
 <location>/var/log/modsecurity/audit.log</location>
</localfile>


  # Permission
<pre> $ chown root:wazuh /var/log/modsecurity/audit.log </pre>
<pre> $ chmod 640 /var/log/modsecurity/audit.log </pre>
<pre> $ systemctl restart wazuh-agent </pre>

  # Chack log
<pre> $ grep modsecurity /var/ossec/logs/ossec.log </pre>

# 7 Create Custom Decoder (Manager Side)

<pre> $ nano /var/ossec/etc/decoders/local_decoder.xml </pre>


   # Add:
<decoder name="modsecurity">
 <prematch>ModSecurity:</prematch>
</decoder>


  # Create Custom Rules (Attack Classification)

<pre> $ nano /var/ossec/etc/rules/local_rules.xml </pre>

  # List site addet rules : 

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


<pre> $ systemctl restart wazuh-manager </pre>

# 8 Testing

# SQLi
<pre> $ curl "http://192.168.33.140/?id=1 OR 1=1" </pre>

# XSS 
<pre> $ curl "http://192.168.33.140/?q=<script>alert(1)</script>" </pre>

# LFI
<pre> $ curl "http://192.168.33.140/?file=../../../../etc/passwd" </pre>

# All chack is "OK" [ Test the machin ]
<pre> $ grep "rules loaded" /var/log/nginx/error.log </pre>
<pre> $ grep SecRuleEngine /etc/nginx/modsec/modsecurity-base.conf </pre>
<pre> $ grep SecRuleEngine /etc/nginx/modsec/modsecurity-base.conf </pre>


# 9 CRS rules file parth : 

<pre> $ nano /opt/coreruleset/crs-setup.conf </pre>
<pre> $ sudo find / -type f -name "crs-setup.conf*" 2>/dev/null </pre>

#  Addtable file 

<pre> $ nano /etc/nginx/nginx.conf </pre>
<pre> $ nano /usr/local/modsecurity/modsecurity.conf </pre>

# Nginx variation hiding contest 

<pre> $ nano /etc/nginx/sites-available/default </pre>

# Add: 

* error_page 403 /custom_403.html;
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
} *



# 10 Adite file : 

# Cerat a file parth 
<pre> $ sudo mkdir -p /var/www/errors </pre>
<pre> $ sudo nano /var/www/errors/custom_403.html </pre>

# Add : 

* <!DOCTYPE html>
<html>
<head>
<title>Forbidden</title>
</head>
<body>
<h1>403 Forbidden</h1>
<p>Access denied.</p>
</body>
</html> * 



# 11 Save and  restart nginx server and test this server 

<pre> $ nikto -h http://IP </pre>
<pre> $ sqlmap -u "http://192.168.33.140/?id=1" --batch 
for i in {1..50}; do curl http://IP/?id=1%20OR%201=1; done </pre>


# Browser Testing For blocking Attack 

# Sql injection 
<pre> $ www.exmapole.com/?id=1 OR 1=1 </pre>

# Xss attack: 
<pre> ?q=<script>alert(1)</script> </pre>

# LFI attack : 
<pre> ?q=<script>alert(1)</script> </pre>


# DDoS Attack: 
<pre> $ hping3 -S –flood -p 80 192.168.33140 </pre>

# 12 Manully Log chack : 

<pre> $ sudo tail -f /var/log/nginx/error.log </pre>
<pre> $ sudo tail -f /var/log/nginx/access.log </pre>
<pre> $ sudo tail -f /var/log/modsecurity/audit.log </pre>

<pre> $ sudo tail -f /var/log/nginx/access.log | grep 403 </pre>
<pre> $ sudo less /var/log/nginx/error.log </pre>
<pre> $ sudo tail -f /var/log/nginx/error.log /var/log/modsec_audit.log </pre>






























