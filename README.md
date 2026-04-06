# WAF
This repository provides an example of how you can combine NGINX, ModSecurity, OWASP CRS, and Wazuh to create a robust web security and monitoring system. Topics include installation, logging, threat detection, and alerts. Perfect for SOC analysts and security professionals. This content is intended for educational use only.
#  NGINX + MODSECURITY + OWASP CRS + WAZUH INTEGRATION #

# ModSecurity Installation:
<pre>apt update </pre>
<pre> apt install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev </pre>

$ sudo apt-get install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev
$  git clone https://github.com/owasp-modsecurity/ModSecurity
$  cd ModSecurity/
$  git submodule init
$  git submodule update
$  sh build.sh
$  ./configure --with-pcre2
$  make
$  make install

$ cp /path/to/ModSecurity/modsecurity.conf-recommended /usr/local/modsecurity/modsecurity.conf
$ cp /path/to/ModSecurity/unicode.mapping /usr/local/modsecurity/unicode.mapping
