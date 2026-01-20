# Ubuntu firewall setup with nginx as a webserver

**Install nginx**

- run `sudo apt install nginx`

**enable ufw (ubuntu firewall)**

- run `sudo ufw enable`

**view ufw status**

- run `sudo ufw status`

Before testing Nginx, the firewall software needs to be adjusted to allow access to the service. Nginx registers itself as a service with ufw upon installation, making it straightforward to allow Nginx access.

**List the application configurations that ufw knows how to work with by typing**

- run `sudo ufw app list`

**Allow an application on the firewall**

- run `sudo ufw allow '<ROLE_NAME>'`

**Available applications role names**

1. Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic).
2. Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic).
3. Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic).
4. OpenSSH
5. Some other roles are ssh, http, https

In case you're working with a AWS instance allow the role OpenSSH to ensure you can ssh into the machine, else you're locked out 💀

**Allow SSH by running the following command**

- run `sudo ufw allow 'OpenSSH'`

**We can check with the systemd init system to make sure the service is running**

- run `systemctl status nginx`

**To stop your web server**

- run `sudo systemctl stop nginx`

**To start the web server when it is stopped**

- run `sudo systemctl start nginx`

**To restart the service again**

- run `sudo systemctl restart nginx`

**If you are only making configuration changes, Nginx can often reload without dropping connections.**

- run `sudo systemctl reload nginx`

**By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing:**

- run `sudo systemctl disable nginx`

**To re-enable the service to start up at boot**

- run `sudo systemctl enable nginx`

**After changes are made to the nginx configs, you can test the changes by:**

- run `sudo nginx -t`

**Alternatively, Restart the service by running**

- run `sudo service nginx restart`
