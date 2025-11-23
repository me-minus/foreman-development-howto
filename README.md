# foreman-development-howto
How to setup a small podman userspace environment with foreman, foreman-puppet, smart-proxy and openvox for a developer

```
mkdir foreman
cd foreman
```

# database
## setup database
```
podman run -d --name foreman-db -e POSTGRES_PASSWORD=DBPASSWORD-ADMIN --network=foreman_default --replace docker.io/postgres:15.14-bookworm
```

## database configuration
```
podman exec -it foreman-db /bin/sh
su - postgres
psql
create user foreman with createdb  encrypted password 'DBPASSWORD';
```

# openvox / puppet
## setup server
```
podman run -d --name openvox --network=foreman_default --hostname openvox ghcr.i
o/openvoxproject/openvoxserver:8.8.0-latest
```

## configure server
```
```

# smart-proxy
# smart-proxy setup
```
podman run -it --name=smartproxy --network=foreman_default --publish 8000:8000 docker.io/library/debian:12 /bin/sh
```

# configure smart-proxy
```
apt install vim ruby npm bundler ruby-dev libxml2-dev libxslt-dev libvirt-dev libcurl4-openssl-dev libsystemd-dev libyaml-dev libkrb5-dev
git clone https://github.com/theforeman/smart-proxy
cd smart-proxy
cp config/settings.yml.example config/settings.yml
vim config/settings.yml
> uncomment :http_host and :http_port
cp config/settings.d/puppet.yml.example config/settings.d/puppet.yml
vim config/settings.d/puppet.yml
> change enabled to true
cp config/settings.d/puppet_proxy_puppet_api.yml.example config/settings.d/puppet_proxy_puppet_api.yml
vim config/settings.d/puppet_proxy_puppet_api.yml
> uncomment :foreman_url and set its value

bundle install --path vendor

bundle exec bin/smart-proxy
```

# foreman
## foreman setup
```
podman run -it --name=foreman --network=foreman_default --publish 3000:3000 dock
er.io/library/debian:12 /bin/bash
```
## foreman configuration (in container):
```
apt install procps systemd
apt install ruby nodejs npm bundler postgresql
apt install ruby-dev libxml2-dev libxslt-dev libvirt-dev libcurl4-openssl-dev libsystemd-dev libyaml-dev
apt install curl emacs


## https://theforeman.org/manuals/3.16/index.html#3.4InstallFromSource
git clone https://github.com/theforeman/foreman.git -b develop
cd foreman
cp config/settings.yaml.example config/settings.yaml
cp config/database.yml.example config/database.yml

## edit config/database.yml
  host: <foreman-db UUID> 
  username: foreman
  password: DBPASSWORD

## add puppet plugin
cd /foreman
git clone https://github.com/theforeman/foreman_puppet.git
cd foreman
echo 'gem "foreman_puppet", path: "../foreman_puppet/"' > bundler.d/foreman-puppet.local.rb

## build
gem install bundler
bundle install --with development test --path vendor
npm install

## set up database schema, precompile assets and locales
RAILS_ENV=production bundle exec rake db:create
RAILS_ENV=production bundle exec rake db:seed 
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production bundle exec rake assets:precompile locale:pack webpack:compile

## start foreman
env RAILS_SERVE_STATIC_FILES=true FOREMAN_BIND=0.0.0.0 bundle exec ./bin/rails s -e production
```

