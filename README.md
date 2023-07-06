# php-fpm on Cacti via SNMP

With higher scale sites php-fpm has become popular to boost efficiency and enable scaling out the application layer (eg. multiple dedicated application servers with php-fpm) and goes hand-in-hand with Nginx which is specifically designed to be extremely efficient at serving large numbers of concurrent requests and trivial to configure to work with php-fpm.

In the same way as previous articles on Nginx on Cacti and Apache on Cacti, this article following on from my previous one on SNMP basics takes look at php-fpm graphing on Cacti.

## php-fpm Stats URL

Like with Apache and Nginx, php-fpm has an easy to configure status URL which provides the few statistics about php-fpm that we monitor here. Configuring php-fpm to provide this requires commenting in the line in /etc/php5/fpm/pool.d/www.conf (at least on Debian) as follows:

```
pm.status_path = /status
```

Then to pass this through Nginx (this also provides the ping URL for testing php-fpm is alive), in the appropriate vhost to put something like:

```
location ~ ^/(status|ping)$ {
        access_log off;
        allow ::1;
        allow 127.0.0.1;
        deny all;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        include fastcgi_params;
}
```

After reloading the configurations a request to http://localhost/status should provide a list of statistics that with a bit of processing with grep and sed can easily be turned into a list of numbers for snmpd to digest. Fortunately I've done that for you :-)

I put the snmpd extension script php-fpm-stats in /etc/snmp/ and set it executable then add the lines from snmpd.conf.cacti-php-fpm to /etc/snmp/snmpd.conf to call the script for appropriate SNMP requests.

Restart snmpd and you should be able to get basic stats via SNMP.

## Cacti Templates

I have generated some basic Cacti Templates for php-fpm.

Simply import the template cacti_host_template_php-fpm.xml, and add the graphs you want to the appropriate device graphs in Cacti. It should just work if your SNMP is working correctly for that device (ensure other SNMP parameters are working for that device).