# gitlab-ce unembedded nginx (ubuntu)
Gitlab CE installation with unembedded Nginx

**Note**: This has been tested on ubuntu 18.04 server and might work with other versions of ubuntu too. No other distros has been tested and it probably isn't going to work on them.


## Configure DNS

In this tutorial I am using `gitlab.yourdomain.com` as example of address we are going to use to access our gitlab ce instance. Make sure this record is available on DNS and is pointing to your server.

## Install required packages

```
sudo apt install curl openssh-server ca-certificates postfix
```

When installing `postfix` you might be prompted to select **General Type Of Email Configuration**, Select `Internet Site` and enter fully qualified domain name of your site.

## Install Gitlab

Below command adds all the required package repositories.

```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

Now you can install gitlab.

```
sudo apt install gitlab-ce
```

## Configure Gitlab Main URL

The main configuration of GitLab is in the '/etc/gitlab' directory. Go to that directory and edit the configuration file 'gitlab.rb' with nano (or your prefered editor)

```
cd /etc/gitlab
nano gitlab.rb
```

In the GitLab configuration go to line 9 'external_url' and change the URL to your URL 'gitlab.yourdomain.com'.

```
external_url 'https://gitlab.yourdomain.com'
```
Close editor and save changes.

## Generate SSL Let's encrypt and DHPARAM Certificate

We will enable the HTTPS protocol for GitLab. I will use a free SSL certificates provided by let's encrypt for the gitlab domain name.

```
sudo apt install letsencrypt -y
```

When the installation is complete, generate a new certificate for the gitlab domain name with the command below.

```
letsencrypt certonly --standalone --agree-tos --no-eff-email --agree-tos --email info@yourdomain.com -d gitlab.yourdomain.com
```

**Note**: You might need to stop your already running nginx or stop services that run on port 80 before you run the above command. Also make sure you are entering valid email address.

Example output of the command:

```
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/gitlab.yourdomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/gitlab.yourdomain.com/privkey.pem
```

Please save or remember where your files are persisted. You are gonna need them later.

now lets generate dhparam pem files in the ssl directory with the openssl command.

```
mkdir -p /etc/gitlab/ssl/
sudo openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
chmod 600 /etc/gitlab/ssl/*
```

## Use your own nginx

As you might already know, gitlab comes with its own embedded nginx. But you can change gitlab configuration so it doesnt use its own nginx. Then you can add proper reverse proxy configuration to your own running nginx so you can access gitlab.

### Tell gitlab to not to use embedded nginx

Edit your gitlab.rb file.

```
sudo nano /etc/gitlab/gitlab.rb
```
Add below configuration. Remember most of the key names are already available. You can just search for the keys and remove the comments and edit them.

```
web_server['external_users'] = ['www-data', 'gitlab-www']

nginx['enable'] = false

gitlab_rails['trusted_proxies'] = [ '192.168.1.0/24', '192.168.2.1', '2001:0db8::/32']
```

- On ubuntu, the default user for nginx is `www-data`, so on first line we are letting this user to access gitlab-ce socket.
- We disabled embedded nginx with second command
- **IMPORTANT** add your server local ip to the array of third command.

### Nginx proxy

Copy the contents of `nginx.conf` file of this repository and paste it as your nginx configuration, under `/etc/nginx/sites-enable/gitlab.conf`. Remember to change the lines of `gitlab.yourdomain.com` to valid address.

Also check if `ssl_dhparam`, `ssl_certificate`, `ssl_certificate_key` are pointing to valid files.

---

## Final step

Now run reconfiguration for gitlab:

```
sudo gitlab-ctl reconfigure
```

and restart nginx

```
sudo service nginx restart
```

You are done.

---

#### sources

- [https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-16-04/](https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-16-04/)
- [https://docs.gitlab.com/omnibus/settings/nginx.html](https://docs.gitlab.com/omnibus/settings/nginx.html)

---
