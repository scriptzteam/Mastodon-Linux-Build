# Mastodon-Linux-Build
Building Mastodon.

***Tested on:***
```
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.1 LTS
Release:        22.04
Codename:       jammy
```

***About Mastodon:***
```
Mastodon is free and open-source software for running self-hosted social networking services. It offers microblogging features that allow you to follow other users and post messages and images with Mastodon. It is written in Ruby and JavaScript and supports audio, video and picture posts, accessibility descriptions, polls, content warnings, animated avatars, custom emojis, and more. Mastodon offers an application for various platforms like Android and iOS.

In this tutorial, we will show you how to install Mastodon on Ubuntu 22.04.
```

Getting Started

First, it is recommended to update all your system packages with the latest version. You can do it by running the following command:

```
apt-get update -y
apt-get upgrade -y
```

After updating your system, you will need to install some dependencies required by Mastodon. You can install all of them with the following command:

```
apt-get install software-properties-common dirmngr apt-transport-https ca-certificates redis-server curl gcc g++ make imagemagick ffmpeg libpq-dev libxml2-dev libxslt1-dev file git-core libprotobuf-dev protobuf-compiler pkg-config autoconf bison build-essential libssl-dev libyaml-dev libreadline-dev libidn11-dev libicu-dev libjemalloc-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm-dev -y
```

Once all the dependencies are installed, you can proceed to the next step.
Install Node.js

Mastodon requires Node.js to be installed on your system. To install Node.js, add the Node.js repository to your server using the following command:

```
curl -sL https://deb.nodesource.com/setup_16.x | bash -
```

Once the repository has been added, install Node.js version 16 with the following command:

```
apt-get install nodejs -y
```

Next, download and add Yarn's GPG key and enable the repository with the following command:

```
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
```

Once added, update the repository and install Yarn with the following commands:

```
apt-get update -y
apt-get install yarn -y
```

Once you have finished, you can proceed to the next step.
Install and Configure PostgreSQL

Mastodon uses PostgreSQL as a database backend. You can install the latest version of PostgreSQL with the following command:

```
apt-get install postgresql postgresql-contrib -y
```

Once installed, log in to PostgreSQL with the following command:

```
su - postgres
postgres@debian:~$ psql
```

Next, create a user for Mastodon with the following command:

```
postgres=# CREATE USER mastodon CREATEDB;
```

Next, exit from the PostgreSQL shell with the following command:

```
postgres=#exit
```

Install Ruby

Mastodon uses Ruby on Rails for the back-end purpose. First, you will need to create a new system user to run the Mastodon server.

You can create it with the following command:

```
adduser --disabled-login --gecos 'Mastodon Server' mastodon
```

Once created, log in to mastodon user with the following command:

```
su - mastodon
```

Next, clone the rbenv repository with the following command:

```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

Next, set up rbenv and ruby-build with the following commands:

```
cd ~/.rbenv && src/configure && make -C src
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec bash
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

Once you have finished, install the latest version of Ruby with the following command:

```
RUBY_CONFIGURE_OPTS=--with-jemalloc rbenv install 3.0.4
```

Once installed, you should get the following output:

```
Downloading ruby-3.0.4.tar.gz...
-> https://cache.ruby-lang.org/pub/ruby/3.0/ruby-3.0.4.tar.gz
Installing ruby-3.0.4...
```

Next, set the Ruby available globally with the following command:

```
rbenv global 3.0.4
```

Next, update the gem and install bundler with the following command:

```
gem update --system
gem install bundler --no-document
```

Once you have finished, you can check the Ruby with the following command:

```
ruby --version
```

ruby 3.0.4
Configure Mastodon

First, log in to mastodon user and download the mastodon git repository with the following command:

```
su - mastodon

git clone https://github.com/tootsuite/mastodon.git ~/live
cd ~/live
```

Next, check out the latest branch with the following command:

```
git checkout $(git tag -l | grep -v 'rc[0-9]*$' | sort -V | tail -n 1)
```

You should get the following output:

```
Note: switching to 'v4.0.2'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

Next, install all the dependencies with the following command:

```
bundle config deployment 'true'
bundle config without 'development test'
bundle install -j$(getconf _NPROCESSORS_ONLN)
yarn install --pure-lockfile
```

Now, setup the Mastodon with the following command:

```
RAILS_ENV=production bundle exec rake mastodon:setup
```

During the setup process, you will be asked several questions. Answer all the questions as shown below:

```
Your instance is identified by its domain name. Changing it afterward will break things.
Domain name: mastodon.example.com

Single user mode disables registrations and redirects the landing page to your public profile.
Do you want to enable single user mode? No

Are you using Docker to run Mastodon? no

PostgreSQL host: /var/run/postgresql
PostgreSQL port: 5432
Name of PostgreSQL database: mastodon_production
Name of PostgreSQL user: mastodon
Password of PostgreSQL user: 
Database configuration works! ????

Redis host: localhost
Redis port: 6379
Redis password: 
Redis configuration works! ????

Do you want to store uploaded files on the cloud? No

Do you want to send e-mails from localhost? yes
E-mail address to send e-mails "from": Mastodon <notifications@mastodon.example.com>
Send a test e-mail with this configuration right now? no

This configuration will be written to .env.production
Save configuration? Yes

Now that configuration is saved, the database schema must be loaded.
If the database already exists, this will erase its contents.
Prepare the database now? Yes

All done! You can now power on the Mastodon server ????

Do you want to create an admin user straight away? Yes
Username: admin
E-mail: test@mastodon.linuxbuz.com
You can login with the password: 159e473bbfe846eac169eafac880b3f0
You can change your password once you login.
```

Once you have finished, exit from the mastodon user with the following command:

```
exit
```

Configure Nginx as a Reverse Proxy for Mastodon

Next, you will need to install Nginx and Certbot to your system. By default, the latest version of Certbot is not available in the Ubuntu 22.04 default repository. So, you will need to add the Certbot repository to your system.

You can add it with the following command:

```
add-apt-repository ppa:certbot/certbot
```

Next, update the repository and install Certbot with Nginx by running the following command:

```
apt-get update -y
apt-get install nginx certbot -y
```

Once both packages are installed, copy the Nginx configuration file from the Mastodon directory to Nginx with the following command:

```
cp /home/mastodon/live/dist/nginx.conf /etc/nginx/sites-available/mastodon.conf
```

Next, enable the Mastodon virtual host configuration file with the following command:

```
ln -s /etc/nginx/sites-available/mastodon.conf /etc/nginx/sites-enabled/
```

By default, Mastodon virtual host configuration file is configured with domain example.com. So you will need to replace the example.com domain with your domain name in the file mastodon.conf. You can replace it with the following command:

```
sed -i 's/example.com/mastodon.example.com/g' /etc/nginx/sites-enabled/mastodon.conf
```

Next, restart the Nginx service to apply the configuration:

```
systemctl restart nginx
```

Next, download the Let's Encrypt free SSL certificate and configure Nginx to use this certificate by running the certbot command:

```
service nginx stop
pkill -f nginx 
certbot -d mastodon.example.com certonly
```

This will generate a free Let's Encrypt SSL certificate and configure the Nginx for your domain mastodon.example.com.
Create Systemd Service for Mastodon

Nginx is now installed and configured to serve Mastodon. Next, you will need to configure systemd service file for Mastodon. To do so, copy the systemd service templates from the Mastodon directory:

```
cp /home/mastodon/live/dist/mastodon-web.service /etc/systemd/system/
cp /home/mastodon/live/dist/mastodon-sidekiq.service /etc/systemd/system/
cp /home/mastodon/live/dist/mastodon-streaming.service /etc/systemd/system/
```

Next, start all services and enable them to start after reboot with the following command:

```
systemctl start mastodon-web
systemctl start mastodon-sidekiq
systemctl start mastodon-streaming
systemctl enable mastodon-web
systemctl enable mastodon-sidekiq
systemctl enable mastodon-streaming
```

Next, check the status of the Mastodon service using the following command:

```
systemctl status mastodon-web mastodon-sidekiq mastodon-streaming
```

Access Mastodon Web Interface

Now, open your web browser and type the URL https://mastodon.example.com.

You are done :)
