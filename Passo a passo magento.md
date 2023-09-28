# Dependências Magento2 2.4.6p2

- Composer 2.2
- ElasticSearch 8.5
- MariaDB 10.6
- PHP 8.2
- RabbitMQ 3.11
- Redis 7.0
- Varnish 7.3
- Nginx 1.22


## PHP
```
sudo apt update
sudo apt install -y libxml2-dev libzip-dev
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.2 php8.2-fpm php8.2-bcmath php8.2-ctype php8.2-curl php8.2-dom php8.2-fileinfo php8.2-filter php8.2-gd php8.2-hash php8.2-iconv php8.2-intl php8.2-json php8.2-libxml php8.2-mbstring php8.2-openssl php8.2-pcre php8.2-pdo-mysql php8.2-simplexml php8.2-soap php8.2-sockets php8.2-sodium php8.2-spl php8.2-tokenizer php8.2-xmlwriter php8.2-xsl php8.2-zip php8.2-zlib

sudo nano /etc/php/8.2/cli/php.ini
Aumentar os valores do PHP realpath_cache_size e realpath_cache_ttl para configurações recomendadas:
memory_limit=2G
realpath_cache_size=10M
realpath_cache_ttl=7200
date.timezone=America/Sao_Paulo
```

## Composer
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --version=2.2.21
php -r "unlink('composer-setup.php');"
or

**usamos essa configuração**
sudo wget https://getcomposer.org/download/latest-2.2.x/composer.phar
sudo mv composer.phar /usr/bin/composer
sudo chmod 777 /usr/bin/composer
```

## Nginx 
```
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \ | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \ http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \ | sudo tee /etc/apt/sources.list.d/nginx.list
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \ | sudo tee /etc/apt/preferences.d/99nginx
sudo apt update
sudo apt install nginx
sudo nano /etc/nginx/nginx.conf
    server_names_hash_bucket_size 64;
sudo systemctl restart nginx
```

## ElasticSearch
```
sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.5.0-amd64.deb.sha512
shasum -a 512 -c elasticsearch-8.5.0-amd64.deb.sha512
sudo dpkg -i elasticsearch-8.5.0-amd64.deb
sudo nano /etc/elasticsearch/elasticsearch.yml
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```

## MariaDB
```
sudo apt update
sudo apt-get install wget software-properties-common dirmngr ca-certificates apt-transport-https -y
sudo apt install mariadb-server mariadb-client
mariadb --version
systemctl status mariadb
sudo mysql_secure_installation
sudo mariadb
> flush privileges
CREATE DATABASE magento;
SHOW DATABASES;
> CREATE USER 'magento'@'localhost' IDENTIFIED BY '3Comm3rc3';
> GRANT ALL PRIVILEGES ON *.* to 'magento'@'localhost';
> quit;
```

## RabbitMQ
```
sudo apt install socat logrotate init-system-helpers adduser -y
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.10.7/rabbitmq-server_3.10.7-1_all.deb
sudo dpkg -i rabbitmq-server_3.10.7-1_all.deb
sudo apt --fix-broken install -y
sudo rabbitmq-server
sudo apt purge rabbitmq-server -y
```

## Redis
```
sudo apt install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis
redis-cli ping
```

## Varnish
```
sudo apt update -y && sudo apt upgrade -y
tee /etc/apt/sources.list.d/varnishcache_varnish70.list > /dev/null <<-EOF
deb https://packagecloud.io/varnishcache/varnish70/ubuntu/ focal main
deb-src https://packagecloud.io/varnishcache/varnish70/ubuntu/ focal main
EOF

sudo apt-update -y
sudo apt install varnish -y
sudo systemctl start varnish && sudo systemctl start varnish
sudo systemctl status varnish
```


## Magento 
```
cd /var/www
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.6-p2 /var/www/magento2-2.4.6-p2
Public Key: 7d8df27f946bc44bfdbdb008cafa315b
Private Key: ee253cef3714a26598c0cf6447b5dce2

sudo nano /etc/nginx/sites-available/ecommerce
upstream fastcgi_backend
{
     server  unix:/run/php/php8.2-fpm.sock;
}
server {
        listen 80;
        listen [::]:80;
        server_name svhr-ecommerce2 svhr-ecommerce2.microware.com.br;
        set $MAGE_ROOT /var/www/magento2-2.4.6-p2;
        include /var/www/magento2-2.4.6-p2/nginx.conf.sample;
        client_max_body_size 2M;

        access_log /var/log/nginx/magento.access;
        error_log /var/log/nginx/magento.error;
}

sudo ln -s /etc/nginx/sites-available/ecommerce /etc/nginx/sites-enabled/
sudo nginx -s reload
sudo nginx -t
cd /var/www/magento2-2.4.6-p2
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :www-data .
chmod u+x bin/magento

bin/magento setup:install --base-url=http://svhr-ecommerce2.microware.com.br --backend-frontname=admin --db-host=localhost --db-name=magento --db-user=magentoUser --db-password=3C0mm3rc3 --admin-firstname=eCommerce --admin-lastname=Microware --admin-email=ecommerce@microware.com.br --admin-user=admin --admin-password=3C0mm3rc3 --language=pt_BR --currency=BRL --timezone=America/Sao_Paulo --use-rewrites=1 --search-engine=elasticsearch8 --elasticsearch-host=localhost --elasticsearch-port=9200 --elasticsearch-index-prefix=magento2

``` 