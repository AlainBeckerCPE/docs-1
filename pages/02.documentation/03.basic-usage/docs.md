---
title: 'Basic Usage'
taxonomy:
    category:
        - docs
---

After the installation wizard (or package manager) has finished, the application server starts automatically. You can now use it without limitations.

Below you will find basic instructions on how to make use of appserver.io. After the installation, you might want to look at some apps. An example application is accessible via `http://127.0.0.1:9080/example`, which is bundled with appserver.io installation.

Start your preferred browser and check out various possibilities. To enter through the login use
the default user credentials `appserver/appserver.i0`.

## Service availability

appserver.io does expose several [servers](../configuration#server-configuration) which are reachable using their respective address and port.
Per default, we only allow for local access using the `localhost` address `127.0.0.1`.

If the servers' availability has to be changed, it can be done using the appropriate configuration file as [described here](../configuration#server-configuration) by altering the `address` param.

Please also make sure that the configured port is forwarded within your environment.

Server availability can be tested using tools like `telnet`, `CURL` or something similar.
On a successful request, a response should be given. The configured [access log](../configuration#optional-configuration) will show the handled request.

## Start and Stop Scripts

In combination with the application server we deliver several other standalone processes that are needed for proper
functioning of different features.

For these processes, we provide the following start and stop scripts for all nix like operating systems.
The scripts work the same way, as any others do, in regards to the OS.

| Script    | Description |
| ----------| ----------- |
| `appserver` | The central process that starts the application server. |
| `appserver-php5-fpm`    | PHP-fpm + appserver configuration. Our default FastCGI backend. Others might be added the same way. |
| `appserver-watcher`     | SA watchdog that monitors filesystem changes and manages application server restarts. |

Using a typical setup, all three of these processes should run, in order to enable appserver's full feature set. To
ultimately execute appserver.io, only the application server process is really needed. However, you will miss simple on-the-fly
deployment (`appserver-watcher`) and might have problems with legacy applications.

Depending on the FastCGI Backend, you may want to use `appserver-php5-fpm` for other
processes e.g. supplying you with a [hhvm](http://hhvm.com/) backend.

Currently, we support three different types of init scripts, which support the commands `start`, `stop`,
`status` and `restart` (additional commands might be available on other systems).

 * **Mac OS X (LAUNCHD)**:
The LAUNCHD launch daemons are located within the application server's installation at `/opt/appserver/sbin`.
They can be used with the schema `/opt/appserver/sbin/<DAEMON> <COMMAND>`

* **Debian, Raspbian, CentOS, ...(SystemV)**:
They are commonly known and located in `/etc/init.d/` and support the commands mentioned above provided
in the form `/etc/init.d/<DAEMON> <COMMAND>`

* **Fedora, ... (systemd)**:
Systemd init scripts can be used using the `systemctl` command with the syntax `systemctl <COMMAND> <DAEMON>`

* **Windows**:
Windows users will find the same three daemons as services within their Windows service management tool where they can be individually controlled.

## Setup Script

appserver.io comes with a simple setup mechanism that sets the correct file system permissions for your environment needs.

```bash
sudo /opt/appserver/server.php -s <MODE>
```

There are three modes you can use to setup appserver.io to your environment needs.

| Mode      | Description |
| ----------| ----------- |
| `install` | The *install* setup mode will be triggered when installing appserver.io on your system. It sets a flag `/opt/appserver/etc/appserver/.is_installed` to indicate if the application server has been already installed. If the flag file exists the *install* setup mode will not execute. |
| `prod`    | Use this mode to use the application server in production mode. The *install* mode, which is executed on first time installation, represents the *prod* mode. |
| `dev`     | Set the correct filesystem permissions for your user account and let appserver.io process run as a current user that makes it a lot easier for local development. |

This is how it should be executed to be ready for local development.

```bash
sudo /opt/appserver/server.php -s dev
# Should return: Setup for mode 'dev' done successfully.
```

## Runner

Another option is the possiblity to use the appserver.io `runner`. In contrast to the default behaviour, which starts appserver.io as a daemon like `nginx` or `Apache`, the `runner` starts appserver.io on demand and expects the application that has to be started in the actual working directory. This is more a `nodejs` like behaviour.

To use the `runner`, it is necessary that the appserver.io `runner` environment has been switched to `dev` mode like described [before](#setup-script).

After switching to `dev` mode and assumed the example application has been cloned with

```sh
$ git clone https://github.com/appserver-io-apps/example
```

the runner can be started with

```sh
$ cd example/src
$ sudo appserver-runner
```

> Using the runner mode restricts appserver.io to deploy exactly one application, even the one that has been found in the actual working directory. Beside that, the log output will be redirected to the actual terminal window, which makes debugging quite more comfortable.