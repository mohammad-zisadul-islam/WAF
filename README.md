# WAF
This repository provides an example of how you can combine NGINX, ModSecurity, OWASP CRS, and Wazuh to create a robust web security and monitoring system. Topics include installation, logging, threat detection, and alerts. Perfect for SOC analysts and security professionals. This content is intended for educational use only.
#  NGINX + MODSECURITY + OWASP CRS + WAZUH INTEGRATION #

# ModSecurity Installation:
<pre>apt update </pre>
<pre> apt install git g++ apt-utils autoconf automake build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre2-dev libtool libxml2-dev libyajl-dev pkgconf zlib1g-dev </pre>

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
