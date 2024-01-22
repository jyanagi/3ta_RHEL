These configurations have been validated on Red Hat Enteprise Linux 9.3, with minimal packages installed. 

## Configure the Database ##

All steps within this section are to be performed on the __Database__ server.

## Download Apache

```
sudo dnf install -y httpd
```

## Create the directory and make the "apache" user the owner. ##

```
sudo mkdir /var/www/db/
sudo chown apache:apache /var/www/db
```
### Start Apache and ensure it starts on system boot. ###

```
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Create the database's front-end CGI script ###

```
sudo vi /var/www/cgi-bin/data.py
```

```bash
#!/usr/bin/env python3
import cgi
import sqlite3

conn = sqlite3.connect('/var/www/db/clients.db')
curs = conn.cursor()
print("Content-type:text/plain\n\n")
form = cgi.FieldStorage()
querystring = form.getvalue("querystring")

if querystring is not None:
    queryval = "%" + querystring + "%"
    select = "SELECT * FROM clients WHERE name LIKE '" + queryval + "'"
else:
    select = "SELECT * FROM clients"

for row in curs.execute(select):
    if len(row) == 4:
        for item in row:
            print(item,'|')
        print("#")
conn.close()
```
Make the file executable

```
sudo chmod 755 /var/www/cgi-bin/data.py
```

# Install sqlite #

```
sudo dnf install sqlite
```

### Download the following SQL database file: ###

```
sudo dnf install wget -y
wget https://raw.githubusercontent.com/doug-baer/hol-3-tier-app/master/db-01a/etc/httpd/db/create_db.sql
```

Create and load the database with the following command:

```
sudo sqlite3 /var/www/db/clients.db < create_db.sql
```
### Change ownership of the databse ###

```
sudo chown apache:apache /var/www/db/clients.db
```

### Enable CGI within the configuration file ###

We must add the following line to load the CGI module:
"LoadModule cgi_module /usr/lib64/httpd/modules/mod_cgi.so"

```
sudo sed -i '178a\\LoadModule cgi_module /usr/lib64/httpd/modules/mod_cgi.so' /etc/httpd/conf/httpd.conf
```
Replace the default port of 80 to 3306 (mysql) that this database server is listening to...
```
sudo sed -i 's/Listen 80/Listen 3306/g' /etc/httpd/conf/httpd.conf
```
Replace `[DB SERVER FQDN OR IP]` with your own environment information
```
sudo sed -i '100a\\ServerName [DB SERVER FQDN OR IP]:3306' /etc/httpd/conf/httpd.conf
```
Add the following to enable access to the database directory. 

It goes right before a line that starts with `<IfModule mime_module>`

```
sudo sed -i '267a\
<Directory "/etc/httpd/db">\
    AllowOverride None\
    Options None\
    Require all granted\
</Directory>\
' /etc/httpd/conf/httpd.conf
```

### Restart Apache ###
```
sudo systemctl restart httpd
```
### Verify that you are able to view the database via `curl`: ###
```
curl http://localhost:3306/cgi-bin/data.py
```
You should see something similar to:
```
1 |
CHOAM |
Dune |
$1.7 trillion |
#
2 |
Acme Corp. |
Looney Tunes |
$348.7 billion |
#
3 |
Sirius Cybernetics Corp. |
Hitchhiker's Guide |
$327.2 billion |
#
4 |
Buy n Large |
Wall-E |
$291.8 billion |
#
5 |
Aperture Science, Inc. |
Valve |
$163.4 billion |
#
6 |
SPECTRE |
007 |
$157.1 billion |
#
7 |
Very Big Corp. of America |
Monty Python |
$146.6 billion |
#
8 |
Frobozz Magic Co. |
Zork |
$112.9 billion |
#
9 |
Warbucks Industries |
Lil' Orphan Annie |
$61.5 billion |
#
10 |
Tyrell Corp. |
Bladerunner |
$59.4 billion |
#
11 |
Wayne Enterprises |
Batman |
$31.3 billion |
#
12 |
Virtucon |
Austin Powers |
$24.9 billion |
#
13 |
Globex |
The Simpsons |
$23.7 billion |
#
14 |
Umbrella Corp. |
Resident Evil |
$22.6 billion |
#
15 |
Wonka Industries |
Charlie and the Chocolate Factory |
$21.0 billion |
#
16 |
Stark Industries |
Iron Man |
$20.3 billion |
#
17 |
Clampett Oil |
Beverly Hillbillies |
$18.1 billion |
#
18 |
Oceanic Airlines |
Lost |
$7.8 billion |
#
19 |
Brawndo |
Idiocracy |
$5.8 billion |
#
20 |
Cyberdyne Systems Corp. |
Terminator |
$5.5 billion |
#
21 |
Paper Street Soap Company |
Fight Club |
$5.0 billion |
#
22 |
Gringotts |
Harry Potter |
$4.4 billion |
#
23 |
Oscorp |
Spider-Man |
$3.1 billion |
#
24 |
Nakatomi Trading Corp. |
Die-Hard |
$2.5 billion |
#
25 |
Los Pollos Hermanos |
Breaking Bad |
$1.3 billion |
```






