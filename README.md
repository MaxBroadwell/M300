# M300

**[Main Config Files]**  
<pre>nano /etc/apache2/sites-available/domainfile.conf  
nano /etc/bind/named.conf.local  
nano /etc/bind/db.zonefile  </pre>

<pre>sudo nano /etc/network/interfaces</pre>


**[sftp put directory recursive]**  
<pre>sftp user@192.168.1.1  
put -r file  </pre>

**[partition mounten]**  
<pre>lshw -class disk  
fdisk /dev/sdb  
    n (new partition)  
    p (primary)  
    1 (partition number)  
    First & Last cylinder default  
    w (write)  
mkfs.ext4 /dev/sdb1  
mkdir /www	(mount point)  
nano /etc/fstab  
mount -a  </pre>


## BIND
1. OS Update & BIND Installation 
<pre>sudo apt update
sudo apt upgrade

sudo apt install bind9</pre>

**BIND Configfiles**
<pre>sudo nano /etc/bind/named.conf.options
sudo nano /etc/bind/named.conf.local
sudo nano /etc/bind/db.gibbix.dmz
sudo nano /etc/bind/db.220.168.192
sudo nano /etc/bind/db.gibbix.lan
sudo nano /etc/bind/db.210.168.192
</pre>


## Apache
**Konfigs neu einlesen**
systemctl reload apache2  
**Apache neustarten**
systemctl restart apache2  
**Apache satus und Errors**
systemctl status apache2  
cat /var/log/apache2/error.log  

1. Apache installation
<pre>sudo apt install apache2</pre>

### Berechtigungen
<pre>sudo addgroup wwwadmin
sudo usermod -aG wwwadmin vmadmin
sudo usermod -aG wwwadmin www-data

sudo chown -R vmadmin:wwwadmin /www
sudo chmod -R 775 /www</pre>


### Configfiles
<pre><b>root@vmLS5:~#</b> cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/gibbix.lan.conf
<b>root@vmLS5:~#</b> nano /etc/apache2/sites-available/gibbix.lan.conf</pre>

Weitere Configfiles
<pre></pre>

**Default-Site deaktivieren & erstellte Site aktivieren**
a2dissite 000-default
a2ensite gibbix.lan



### Virtuelle Hosts
some text

<pre></pre>




## vsFTPd
**Installation**
<pre>sudo apt install vsftp</pre>

### Konfiguration
**Create FTP-User**
<pre>sudo adduser ftpuser --home /www/ --shell /bin/false</pre>
**/bin/false** in **/etc/shells** eintragen  

**[/etc/vsftpd.conf]**
folgende Änderungen:  
<pre>	chroot_local_user=YES	-> User in Homeverzeichnis einsperren
	userlist_deny=NO
	userlist_enable=YES
	userlist_file=/etc/vsftpd.user_list</pre>
 
**[/etc/vsftpd.user_list]**
Datei vsftpd.user_list erstellen und **ftpuser** eintragen
 
systemctl restart vsftpd  
  

## TLS
<pre>sudo a2enmod ssl
sudo service apache2 restart</pre>  

**[/etc/apache2/ports.conf]**
auf folgendes überprüfen
 ```   
<IfModule ssl_module>  
 	Listen 443  
</IfModule>
```
  
Config neu einlesen:
<pre>sudo service apache2 reload</pre>
  
  
**openssl installieren**
<pre>sudo apt install openssl
cd /etc/ssl</pre>

**Privaten Schlüssel und Certifacte Signing Request generieren**
<pre>openssl genrsa -out www.gibbix.dmz.key 4096
openssl req -new -sha256 -key www.gibbix.dmz.key -out www.gibbix.dmz.csr
	CountryName: CH
	State: BE
	LocalityName: Bern
	OrganizationName: gibbix
	OrganizationalUnit: M300
	CommonName: www.gibbix.dmz	(muss mit domain übereinstimmen!)</pre>

**Zertifikat ausstellen**
<pre>openssl x509 -req -days 365 -in www.gibbix.dmz.csr -signkey www.gibbix.dmz.key -out www.gibbix.dmz.crt -sha256</pre>

Entweder bei **/etc/apache2/sites-available/www.gibbix.dmz.conf** folgende Zeilen hinzufügen:
<pre>	SSLEngine on
	SSLCertificateFile /etc/ssl/www.gibbix.dmz.crt
	SSLCertificateKeyFile /etc/ssl/www.gibbix.dmz.key</pre>
  
 Oder eine weitere Datei erstellen:
 <pre>sudo nano /etc/apache2/sites-available/ssl.conf</pre>
```
sudo nano /etc/apache2/sites-available/ssl.conf
<VirtualHost *:443>
	SSLEngine on
	SSLCertificateFile /etc/ssl/www.gibbix.dmz.crt
	SSLCertificateKeyFile /etc/ssl/www.gibbix.dmz.key
	DocumentRoot /www/
</VirtualHost>
```

zum Schluss:
<pre>sudo a2ensite ssl.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
</pre>
