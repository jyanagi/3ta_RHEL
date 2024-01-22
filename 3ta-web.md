These configurations have been validated on Red Hat Enteprise Linux 9.3, with minimal packages installed. 

## Configure the Web Server ##

Download and Install NGINX

```
sudo dnf install nginx -y
```

Modify the NGINX configuration file
```
sudo vi /etc/nginx/nginx.conf
```

Starting from `#Settings for a TLS enabled server` replace the SSL configurations with the following...

Don't forget to replace `<Your Web Server FQDN>` and `<Your Application Server FQDN>` with the appropriate FQDN or IP address information. Since we are using Secure HTTP, this configuration example uses the FQDN.
```
   #Settings for a TLS enabled server.

    server {
        listen       443 ssl;
        server_name  <Your Web Server FQDN>;

        ssl_certificate /etc/pki/tls/certs/server.crt;
        ssl_certificate_key /etc/pki/tls/private/server.key;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;

        location / {
           rewrite ^/$ /cgi-bin/app.py last;
        }

        location /cgi-bin/app.py {
           proxy_pass https://<Your Application Server FQDN>:8443/cgi-bin/app.py;
           proxy_set_header Host $host;
           proxy_redirect http:// https://;
        }

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

## Server Certificates ##
 
There is an assumption that you already know how to generate SSL certificates for your web server(s), and that you are able to receive both the private key and the certificate file in pem format.

In my example, I had both the `server.key` and `server.crt` files for my SSL certificate located in the `~/certs` directory. Your source directory may be different. Copy your Server Certificate to the `/etc/pki/tls/certs/` directory, and your Private Key to the `/etc/pki/tls/private/` directory.
```
sudo cp ~/certs/server.crt /etc/pki/tls/certs/
sudo cp ~/certs/server.key /etc/pki/tls/private/
```
Enable and Start NGINX
```
systemctl enable nginx
systemctl start nginx
```
# Verify that you are able to view the database via `curl`: #
```
curl -k https://localhost
```
If you see HTML tags, such as the following excerpt, you are successful.
```
</body></html>

<tr>
<td style=font-family:sans-serif;text-align:left;border:1px;border-style:solid;bordercolor:#A9A9A9;background-color:#FFFFFF;border-radius:5px;>
25  </td>
<td style=font-family:sans-serif;text-align:left;border:1px;border-style:solid;bordercolor:#A9A9A9;background-color:#FFFFFF;border-radius:5px;>
Los Pollos Hermanos  </td>
<td style=font-family:sans-serif;text-align:left;border:1px;border-style:solid;bordercolor:#A9A9A9;background-color:#FFFFFF;border-radius:5px;>
Breaking Bad  </td>
<td style=font-family:sans-serif;text-align:left;border:1px;border-style:solid;bordercolor:#A9A9A9;background-color:#FFFFFF;border-radius:5px;>
$1.3 billion  </td>
</tr>

</body></html>

</body></html>
```
