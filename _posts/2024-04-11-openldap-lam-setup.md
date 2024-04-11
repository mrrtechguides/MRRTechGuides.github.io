---
layout: post
title: OpenLDAP and LAMPro on Ubuntu 23.10
excerpt_image: "/assets/images/post/post2/post-2-1.png"
categories: Development
tags: [OpenLDAP, LAM, LAMPRO, LDAPAccountManager, Ubuntu, Linux, Account Management, Authorization, Accounts, Active Directory]
top: 2
---

OpenLDAP (Lightweight Directory Access Protocol) is an open-source implementation of the LDAP protocol, which is a directory service protocol used for accessing and maintaining distributed directory information services over a network. It's commonly used for centralized authentication, authorization, and information lookup services in a networked environment. OpenLDAP provides a platform-independent, standards-based infrastructure for managing and organizing directory data.

LAM (LDAP Account Manager), also known as LAM Pro, is a web-based LDAP administration tool specifically designed for managing LDAP directories. It provides a user-friendly interface for administrators to manage users, groups, organizational units, and other directory objects. LAM simplifies tasks such as creating, modifying, and deleting directory entries, as well as managing access control and permissions.

Both OpenLDAP and LAM are powerful tools for managing directory services within an infrastructure for several reasons:

1.  **Centralized Authentication and Authorization**: OpenLDAP allows organizations to centralize user authentication and authorization processes. This means users can access multiple services and resources using a single set of credentials stored in the LDAP directory.
    
2.  **Scalability**: OpenLDAP is highly scalable and can handle large volumes of directory data and user requests. This makes it suitable for both small organizations and large enterprises.
    
3.  **Standards Compliance**: OpenLDAP adheres to industry standards, particularly the LDAP protocol, ensuring compatibility with a wide range of applications and systems.
    
4.  **Web-based Management Interface**: LAM provides a user-friendly web interface for managing the LDAP directory, reducing the complexity of directory administration tasks and making them accessible to non-technical users.
    
5.  **Role-based Access Control**: Both OpenLDAP and LAM support role-based access control (RBAC), allowing administrators to define granular permissions and access levels for different users and groups.
    
6.  **Customizability**: OpenLDAP and LAM are highly customizable, allowing administrators to extend their functionality and integrate them with other systems and applications as needed.

### Install and Configure SLAPD

 * Install slapd and ldap utilities packages
   `sudo apt install slapd ldap-utils`
 * Configure slapd with organization information
     `sudo dpkg-reconfigure slapd`
     * Omit OpenLDAP server configuration?  `No`
     * DNS Domain name: ex. `organization.com`
     * Organization name: ex. `organization`
     * Administrator password
     * Do you want the database to be removed when salpd is purged? (it's up to your preference)
     * Move old database? `Yes`
 * Configure ldap configuration file
`sudo vim /etc/ldap/ldap.conf`
	 * Set Base to dc=organization,dc=com
	 * Set URI to ldaps:/// ldapi:// ldap://
* Configure Groups module
	* `sudo vim base-groups.ldif`
		```
		dn: ou=People,dc=organization,dc=com
		objectClass: organizationalUnit
		ou: People

		dn: ou=Groups,dc=organization,dc=com
		objectClass: organizationalUnit
		ou: Groups
		```
	* `sudo ldapadd -x -D cn=admin,dc=organization,dc=com -W -f base-groups.ldif`
* Configure Password policy module
	* Create LDIF file for password policy module and add the content below
	  `sudo vim load-ppolicy-mod.ldif`
		```
		dn: cn=module{0},cn=config
		changetype: modify
		add: olcModuleLoad
		olcModuleLoad: ppolicy.la
		```
	* Run command to load the module to openldap
	 `sudo ldapadd -Y EXTERNAL -H ldapi:/// -f load-ppolicy-mod.ldif`
	* Install password quality tool that can be configured to be used for password quality checking
	 `sudo apt-get install libpwquality-tools`

### Secure OpenLDAP with TLS/SSL
* Create certificates according to organization needs.
	```
	sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/ldapserver.key -out /etc/ssl/ldapserver.crt
	sudo chown openldap:openldap /etc/ssl/{ldapserver.crt,ldapserver.key}
	sudo cp /etc/ssl/ldapserver.crt /usr/local/share/ca-certificates
	sudo cp /etc/ssl/ldapserver.crt /usr/share/ca-certificates
	```
* Edit LDAP configuration file to use the CA Certificate
`sudo vim /etc/ldap/ldap.conf`
	* Change TLS CACERT to `/etc/ssl/certs/ldapserver.pem`
* In home directory create the TLS LDIF file to load to openLDAP
	* Create file
	`vim tls.ldif`
	* Add the follwing contets to the file to map to SSL certificates
		 ```
		 dn: cn=config
		changetype: modify
		add: olcTLSCACertificateFile
		olcTLSCACertificateFile: /etc/ssl/certs/ldapserver.pem
		-
		add: olcTLSCertificateFile
		olcTLSCertificateFile: /etc/ssl/ldapserver.crt
		-
		add: olcTLSCertificateKeyFile
		olcTLSCertificateKeyFile: /etc/ssl/ldapserver.key

		 ``` 
 * Load SSL certificates to openLDAP
 `sudo ldapadd -Y EXTERNAL -H ldapi:/// -f tls.ldif`
 `sudo sed -i 's|/etc/ssl/certs/ca-certificates.crt|/etc/ssl/ldapserver.crt|' /etc/ldap/ldap.conf`
 * Setup OpenLDAP to use LDAPS( LDAP over SSL) instead of LDAP
  `sudo vim /etc/default/slapd`
	  * Under SLAPD_SERVICES add LDAPS:/// and remove LDAP:///
  * Restart the SLAPD Service
  `sudo systemctl restart slapd`

### LDAP Account Manager using NGINX (LAM)
If using PRO Version download PRO Version from LAM Pro website, if not download free version: Website: https://www.ldap-account-manager.org/lamcms/releases
* Make work directory to install LAM
`mkdir /home/user/lam`
`cd lam`
* Copy LAM Deb package from website over to ldap server under work directory.
*  Install the package
	`sudo dpkg -I deb_package`
* Fix broken install to download dependencies needed for the package to install successfully
`sudo apt --fix-broken install`
* Install NGINX Package
`sudo apt install nginx`
* Remove apache and mark hold libraries needed for LAM to work
	  ```
		 sudo apt-mark hold libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0
		 sudo apt purge apache2*
		 sudo rm -r /var/lib/apache2
		sudo apt autoremove
		```
* Setup SSL certificates for NGINX web server hosting LAM
	`sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt`
* Install PHP files
	 `sudo apt install php8.2-common php8.2-fmp`
* Edit the nging default site to load LAM
	`sudo vim/etc/nginx/sites-available/default`
	* Replace the file with the following content
		```
		server {
		    listen 80;
		    server_name 192.168.160.74;
		 
		    # Redirect HTTP to HTTPS
		    return 301 https://$host$request_uri;
		}
		server {
		    listen 443 ssl;
		    server_name 192.168.160.71;
		 
		    # SSL certificate configuration
		    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
		    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
		 
		    # LAM root directory
		    root /usr/share/ldap-account-manager/;
		    index index.php index.html;
		    
			# PHP-FPM configuration
		    location ~ \.php$ {
		        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
		        fastcgi_param SCRIPT_FILENAME$document_root$fastcgi_script_name;
				include fastcgi_params;
		    }
		    
		    # Deny access to sensitive files
		    location ~ /\.ht {
		        deny all;
		    }
		}
		```
* Modify permissions for lam configuration files
	`sudo chown www-data:www-data /usr/share/ldap-account-manager`
* Restart NGINX
 `sudo systemctl restart nginx`

Now the user should be able to visit https://server_address top login to LAM.

### Configure LAM through WEB

* Select LAM Configuration in the top left
* Click edit settings - password is lam
	* Add license if you have the pro version
	* Change the master password to be more secure
	* Click save and then go back to LAM Configuration page
* Click on server profiles, password is lam
	* Make the following changes:
		*	Server address: `ldaps:///`
		*	Tree suffic: `dc=organization,dc=com`
		*	List of valid users: `cn=admin,dc=organization,dc=com`
		*	Change password for server profile to be more secure
		*	Click on the second tab : Account Types and update active account type. 
			*	Groups : dc=Groups, dc=organization, dc=com
			*	Users: dc=People, dc=organization, dc=com
		*	Save
	*	On login screen type admin credentials for LDAP Admin Account and you should be able to login and manage OpenLDAP thorugh LAM


In summary, OpenLDAP and LAM are powerful tools for managing directory services, providing centralized authentication and authorization, scalability, standards compliance, ease of administration, and flexibility to meet the needs of diverse infrastructures.
