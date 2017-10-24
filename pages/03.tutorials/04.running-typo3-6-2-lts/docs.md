---
title: 'Running TYPO3 6.2 LTS'
taxonomy:
    category:
        - tutorials
---

Appserver.io is a pretty cool and sophiscated infrastructure fully built upon the PHP stack. This makes it truely easy
to develop and extend the platform. Appserver.io comes with a built in webserver module with PHP-FPM therefore it is
possible to install any PHP-App you like and run it on that platform. The following guide shows you how easy it is to
install appserver.io on a Mac and run TYPO3 6.2 on it.


**Prerequisite**: *Up and running installation of MySQL*

You will need a running installation of appserver.io *(>= Version 1.0.0-rc3)*. If you are new to this
project you can easily <a href="{{site.home_url}}downloads" target="_blank" class="external no-image">download</a> and follow the
[installation guide](../../documentation/installation) for your specific OS.

After the setup has finished the appserver.io is up and running and you can call the welcome page with

[http://localhost:9080/](http://localhost:9080/)

By default, appserver.io is configured to run on port `9080` in order to not to affect any existing webserver installations.
You can easily change that in the /opt/appserver/etc/appserver.xml just by going to section

```xml
<server name="http"
	...
```

and change the port within that section for example to 80. After that restart the appserver.io which can be
done with the following command.

```bash
sudo /opt/appserver/sbin/appserverctl restart
```

Of course there is no need to change the port if you only want to check out the capabilities of this unbelivable platform.



### Installation

Download the latest TYPO3 Release from typo3.org.

To install TYPO3 we have now two options. The easiest way is to install TYPO3 without creating a vhost.
Therefore you just unpack the TYPO3 source into your webrootfolder which in case of the appserver is always the webapps
folder underneath /opt/appserver/webapps/. In that folder you will still find the already installed example app and of
course the welcome page. We are just creating a folder with name „newtypo3“

```bash
cd /opt/appserver/webapps
mkdir newtypo3
```

and unpacking the source there.

After successfully unpacking the TYPO3 sources, use the TYPO3 installer just by opening a browser and calling the URL http://127.0.0.1:9080/newtypo3/. If you do so, TYPO3 will display the message to create the install file. You can do that just by creating an empty file in the TYPO3 Webroot.

```bash
touch /opt/appserver/webapps/newtypo3/FIRST_INSTALL
```

To go ahead with the installation process you have to rectify the file permissions in order for TYPO3 to create the necessary folders.

```bash
chmod -R 775 /opt/appserver/webapps/newtypo3/
```

After correcting the file permissions call the Installation Tool by using the URL mentioned before and you will
get the main installer. There might be some warnings concerning the upload size and the php_max_execution time but
there is no need to fix that now. Move on with the installation process and type in the MySQL login credentials. You can create a new database „typo3_newtest“ for example.  After that create a backend user and you are all set.

#### Additional Infos

To correct the issues occurred during the installation you just have to change your PHP settings. If you are using
the PHP-FPM delivered with appserver.io which is configured by default you will find the php.ini in:

```bash
/opt/appserver/etc/
```

After changing the ini, you have to restart the PHP-FPM with the following command:

```bash
sudo /opt/appserver/sbin/phpfpmctl restart
```

### Installing with Virtual Host

As with any other Webserver using a vhost you first have to add the domain you like to use in your hosts file.

```bash
sudo vi /etc/hosts
```

Add the following lines there:

```bash
127.0.0.1 typo3.local
::1 typo3.local
fe80::1%lo0 typo3.local
```

Afterwards you had to add the vhost to the webserver config of the appserver which you also find in the appserver.xml.
You will find two <server> nodes in the xml. The first is for the http and second for the https definitions. If don't
need a SSL-Certificate just scroll to the the <virtualHosts> Section and add the following host configuration for
TYPO3 there.

```xml
<virtualHost name="typo3.local">
	<params>
		<param name="admin" type="string">
			info@appserver.io
		</param>
		<param name="documentRoot" type="string">
			webapps/newtypo3
		</param>
	</params>
	<rewrites>
    	<rewrite condition="-d{OR}-f{OR}-l" target="" flag="L" />
		<rewrite condition="index\.php" target="" flag="L" />
		<rewrite condition="^(.+)\.(\d+)\.(php|js|css|png|jpg|gif|gzip)$" target="$1.$3" flag="L" />
		<rewrite condition="^/(typo3|t3lib|fileadmin|typo3conf|typo3temp|uploads|favicon\.ico)" target="" flag="L" />
		<rewrite condition="^typo3$" target="typo3/index_re.php" flag="L" />
		<rewrite condition=".*" target="index.php" flag="L" />
	</rewrites>
</virtualHost>
```

After adding the Vhost restart the appserver and you should also be able to request TYPO3 over the name based virtual host.