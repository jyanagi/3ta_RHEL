These configurations have been validated on Red Hat Enteprise Linux 9.3, with minimal packages installed. 

## Configure the App Server ##

Download and Install Apache, SSL Modules, and WGET

```
sudo dnf install httpd mod_ssl wget -y
```

Modify the listening port from 80 to 8443
```
sudo sed -i 's/Listen 80/Listen 8443/g' /etc/httpd/conf/httpd.conf
```

Add the SSL and CGI Modules to the configuration
```
sudo sed -i '163a\\LoadModule ssl_module /usr/lib64/httpd/modules/mod_ssl.so\n' /etc/httpd/conf/httpd.conf
sudo sed -i '226a\\LoadModule cgi_module /usr/lib64/httpd/modules/mod_cgi.so\n' /etc/httpd/conf/httpd.conf
```

Append the following to the end of the Apache httpd.conf file for configuring the SSL module:
```
sudo sed -i '$a\Include /etc/httpd/conf.d/ssl.conf' /etc/httpd/conf/httpd.conf
```

Modify the `/etc/httpd/conf.d/ssl.conf` 

Comment out the listening address since this is already taken care of via the /etc/httpd/conf/httpd.conf file
```
sudo sed -i '5 s/^/#/' /etc/httpd/conf.d/ssl.conf
```

Change default SSL port from `443` to `8443`
```
sudo sed -i 's/VirtualHost _default_:443/VirtualHost _default_:8443/g' /etc/httpd/conf.d/ssl.conf
```

## Server Certificates ##
 
There is an assumption that you already know how to generate SSL certificates for your web server(s), and that you are able to receive both the private key and the certificate file in pem format.

In my example, I had both the `server.key` and `server.crt` files for my SSL certificate located in the `~/certs` directory. Your source directory may be different.
```
sudo cp ~/certs/server.crt /etc/pki/tls/certs/
sudo cp ~/certs/server.key /etc/pki/tls/private/
```
Modify the `ssl.conf` by replacing "localhost.crt" and "localhost.key" with your respective key names
```
sudo sed -i 's/localhost.crt/server.crt/g' /etc/httpd/conf.d/ssl.conf
sudo sed -i 's/localhost.key/server.key/g' /etc/httpd/conf.d/ssl.conf
```

Copy the full server CA certificate that signed and issued the server certificate. If your signing CA is an intermediary, ensure you also include the Root CA. The format will look like the following:

```
-----Begin Certificate-----

Server Certificate

-----End Certificate-----

-----Begin Certificate-----

Intermediate CA, if present

-----End Certificate-----

-----Begin Certificate-----

Root CA

-----End Certificate-----
```

Save this file as `server-chain.crt`

Copy the `server-chain.crt` certificate, which includes the server, intermediary [if present], and the root, and store it in the /etc/pki/tls/certs/ directory
```
sudo cp ~/certs/server-chain.crt /etc/pki/tls/certs/
```
Uncomment line 102 so that it references this full-chain certificate:
```
sudo sed -i '102 s/^.//' /etc/httpd/conf.d/ssl.conf
```
Enable and Start Apache
```
systemctl enable httpd
systemctl start httpd
```
# Configure the Application Script #

Create the CGI Binaries directory:
```
sudo mkdir /var/www/cgi-bin
```
Change ownwership of this directory
```
sudo chown apache:apache /var/www/cgi-bin
````
Download the Application script using the following wget command:
```
wget https://raw.githubusercontent.com/doug-baer/hol-3-tier-app/master/app-01a/etc/httpd/cgi-bin/app.py
```
Modify the IPs and hostnames of the web servers to fit your environment's standards.

Add execute permissions and move to the `/etc/httpd/cgi-bin` directory:
```
sudo chmod 755 app.py
sudo cp app.py /var/www/cgi-bin/app.py
```

# Verify that you are able to view the database via `curl`: #

```
curl -k https://localhost:8443/cgi-bin/app.py
```
