---
title: 'Running Wordpress'
---

Appserver.io is a pretty cool and sophiscated infrastructure fully built upon the PHP stack. This makes it truely easy
to develop and extend the platform. Appserver.io comes with a built in webserver module with PHP-FPM therefore it is
possible to install any PHP-App you like and run it on that platform. The following guide shows you how easy it is to
install appserver.io on a Mac and run Wordpress on it.


**Prerequirements**: *Up and running installation of MySQL*

You will need a running installation of appserver.io *(>= Version 1.0.0-rc3)*. If you are new to this
project you can easily [download](#) and follow the
[installation guide](#) for your specific OS.

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

Of course there is no need to change the port if you only want to check out the capabilities of this tremendous platform.




##Installation:

You are now set to install and run your application on appserver.io. For that we download the latest wordpress release
from wordpress.org.

To go ahead and install wordpress we have now two options. The easiest way is to install wordpress without creating a
vhost. Therefore you just unpack the wordpress source into your webrootfolder which in case of the appserver is always
the webapps folder underneath /opt/appserver/webapps/. In that folder you will still find the already installed example
app and of course the welcome page. We are just creating a folder with name „wordpress“ and unpacking the source there.

After successfully unpacking the wordpress sources you are able to use the wordpress webinstaller just by opening a
browser and calling the URL http://127.0.0.1:9080/wordpress/. Before you step over the installation you should correct
the rights of the wordpress folder to ensure wordpress is able to write the configuration.

```bash
chmod -R 775 /opt/appserver/webapps/wordpress/
```

Now you are free to step over the installation wizard and for that it is necessary to create a database on your local
running MySQL. To create a database you can use the MySQL command line or just use another database administration tool
like phpMyAdmin. Of course you can also install phpMyAdmin on appserver.io. 
<a href="(#)">
Just read the appropriate tutorial.</a>

To create the database by the command line just use the following line

```bash
mysql -uroot -p
```

Now you are on the MySQL command line and it is pretty easy to create an empty database. Just use the following command.

```sql
CREATE DATABASE wordpress;
```

Now you are ready to install wordpress. Just follow steps on the install wizard.

If you want to use a virtual host to run wordpress simply follow the steps below. As with any other Webserver using a
vhost you first have to add the domain you'd like to use in your hosts file.

```bash
sudo vi /etc/hosts
```

Add the following lines there:

```bash
127.0.0.1 wordpress.local
::1 wordpress.local
fe80::1%lo0 wordpress.local
```

Afterwards you have to add the vhost to the webserver config of the appserver which you also find in
`/opt/appserver/etc/appserver/conf.d/virtual-hosts.xml`. There is already an example virtual host configuration
available there. Put the following configuration within the <virtualHosts> tag.

```xml
<virtualHost name="wordpress.local">
    <params>
        <param name="admin" type="string">info@appserver.io</param>
        <param name="documentRoot" type="string">webapps/wordpress</param>
    </params>
</virtualHost>
```

After adding the Vhost you have to restart the appserver and you should start with the installation like described at
the beginning of this tutorial

```bash
sudo /opt/appserver/sbin/appserverctl restart
```

If you alread installed wordpress and now you want to use the configured filename you just have to change the siteurl
in the settings menu of wordpress.