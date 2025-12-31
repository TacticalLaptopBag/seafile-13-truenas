# Running Seafile Behind a Reverse Proxy

Trying to get Seafile running under a reverse proxy such as [Nginx Proxy Manager][npm] (NPM)
is rather tricky, and requires additional configuration after the initial setup.

This guide will assume you are trying to use [NPM][npm] as your reverse proxy,
and that [NPM][npm] is handling your HTTPS certificates.
However, this should theoretically apply to other reverse proxy methods.

## Setup

First, run through the Seafile setup as usual.
In TrueNAS apps, go to Discover Apps, then hit the three dots and click `Install from YAML`.

Paste the entire contents of [docker-compose.yml](docker-compose.yml) into the window,
name the app whatever you want, go through the config in `x-common-data`,
and click save.

## Fix

Once done, if you try to login to your Seafile instance from the public URL you configured,
you'll find a CSRF token error.

Shut down Seafile from the TrueNAS Apps interface before continuing with the below steps.

Now, all we need to do to fix this is modify some Seafile configuration files.
Open a shell in TrueNAS and paste this, make sure to change the value of SEAFILE_VOLUME:
```bash
export SEAFILE_VOLUME=/opt/seafile-data  # Change this to your `seafile_volume` directory!
sudo nano $SEAFILE_VOLUME/seafile/conf/seahub_settings.py
sudo nano $SEAFILE_VOLUME/seafile/conf/ccnet.conf
```

Upon running the command, you should first see the `seahub_settings.py` file,
and it should look something like this:
```py
# -*- coding: utf-8 -*-
SECRET_KEY = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

TIME_ZONE = 'Etc/UTC'
```
Add these lines to the bottom of the file, make sure to replace `127.0.0.1` with your `seafile_server_hostname`.
```py
SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
FILE_SERVER_ROOT = "https://127.0.0.1/seafhttp"
CSRF_TRUSTED_ORIGINS = ["https://127.0.0.1", "http://127.0.0.1"]
```
Hit CTRL+X, Y, and enter. You should then be brought to a new file.
Paste this into it, make sure to replace `127.0.0.1` with your `seafile_server_hostname`:
```ini
[General]
SERVICE_URL = "https://127.0.0.1"
```
Hit CTRL+X, Y, and enter. 

That's it! Simply start Seafile from the TrueNAS Apps interface again and
wait for the `seafile` container log to say
```
Seahub is started
Done.
```
Once you see that, navigate to your Seafile instance from the public URL and login with the initial admin account you configured.
You're now running Seafile behind a reverse proxy!


<!-- links -->
[npm]: https://nginxproxymanager.com/
