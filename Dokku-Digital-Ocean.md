# Dokku on DigitalOcean

:droplet: Your very own PaaS

## Create Droplet

DigitalOcean has a [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-dokku-application) - the instructions are up-to-date, despite the notice.

- Select **Dokku v0.4.4 on 14.04**
- Be sure to use an SSH key

If you have a domain, use virtualhost naming. Otherwise, Dokku will use different ports for each deploy of your app. You can add easily add a domain later.

## Set Up Server

Firewall

```sh
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

[Automatic updates](https://help.ubuntu.com/14.04/serverguide/automatic-updates.html)

```sh
sudo apt-get install unattended-upgrades
echo 'APT::Periodic::Unattended-Upgrade "1";' >> /etc/apt/apt.conf.d/10periodic
```

Swap

```sh
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```

Time zone

```sh
sudo dpkg-reconfigure tzdata
```

and select `None of the above`, then `UTC`.

## Deploy

Get the official Dokku client locally

```sh
git clone git@github.com:progrium/dokku.git ~/.dokku

# add the following to either your
# .bashrc, .bash_profile, or .profile file
alias dokku='$HOME/.dokku/contrib/dokku_client.sh'
```

Create app

```sh
dokku apps:create myapp
```

Add a `CHECKS` file

```txt
WAIT=2
ATTEMPTS=15
/
```

Deploy

```sh
git remote add dokku dokku@dokkuhost:myapp
git push dokku master
```

## Workers

Dokku only runs web processes by default. If you have workers or other process types, install [dokku-shoreman](https://github.com/statianzo/dokku-shoreman).

```sh
git clone https://github.com/statianzo/dokku-shoreman.git /var/lib/dokku/plugins/dokku-shoreman
dokku plugins-install
dokku ps:rebuildall
```

## One-Off Jobs

```sh
dokku run rake db:migrate
dokku run rails console
```

## Scheduled Jobs

Two options

1. Add a [custom clock process](https://devcenter.heroku.com/articles/scheduled-jobs-custom-clock-processes) to your Procfile

2. Or create `/etc/cron.d/myapp` with:

  ```
  * * * * * root /usr/local/bin/dokku run myapp rake task1
  0 0 * * * root /usr/local/bin/dokku run myapp rake task2
  ```

  If choosing this option, include a job to clean containers.

  ```
  0 * * * * root /user/local/bin/dokku cleanup
  ```

## Logging

[Papertrail](https://papertrailapp.com) is great and has a free plan.

### Apps

Use [Logspout](https://github.com/gliderlabs/logspout).

```sh
docker pull gliderlabs/logspout:latest
docker run --restart=always -d --name=logspout -v=/var/run/docker.sock:/tmp/docker.sock -h $(hostname) gliderlabs/logspout syslog://logs.papertrailapp.com:12345
```

### Nginx

Install remote syslog

```sh
cd /tmp
wget https://github.com/papertrail/remote_syslog2/releases/download/v0.13/remote_syslog_linux_amd64.tar.gz
tar xzf ./remote_syslog*.tar.gz
cd remote_syslog
sudo cp ./remote_syslog /usr/local/bin
```

Create `/etc/log_files.yml` with:

```sh
files:
  - /var/log/nginx/*.log
destination:
  host: logs.papertrailapp.com
  port: 12345
  protocol: tls
```

## Custom Domains

```sh
dokku domains:add www.datakick.org
```

## SSL

```sh
tar -cv server.crt server.key > archive-of-certs.tar
dokku nginx:import-ssl < archive-of-certs.tar
```

[More info](http://progrium.viewdocs.io/dokku/nginx)

## Memcached

```sh
git clone https://github.com/rlaneve/dokku-link /var/lib/dokku/plugins/link
git clone https://github.com/jezdez/dokku-memcached-plugin /var/lib/dokku/plugins/memcached
dokku plugins-install
dokku memcached:create myapp
dokku config:set myapp MEMCACHE_SERVERS=memcachedhost:11211
```

## Redis

```sh
git clone https://github.com/luxifer/dokku-redis-plugin /var/lib/dokku/plugins/redis
dokku plugins-install
dokku redis:create myapp
```

## TODO

- database
- [monitoring](https://www.brianchristner.io/how-to-setup-docker-monitoring/)

## Bonus

Find great Docker projects at [Awesome Docker](https://github.com/veggiemonk/awesome-docker).

## Resources

- [Additional Recommended Steps for New Ubuntu 14.04 Servers](https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers)
