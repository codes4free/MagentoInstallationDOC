# Dependências Magento2 2.4.6p2

- [PHP 8.2](#PHP)
- [Composer 2.2](#Composer)
- [Nginx 1.22](#Nginx)
- [ElasticSearch 8.5](#ElasticSearch)
- [OpenSearch 2.5](#OpenSearch)
- [MariaDB 10.6](#MariaDB)
- [RabbitMQ 3.11](#RabbitMQ)
- [Redis 7.0](#Redis)
- [Varnish 7.3](#Varnish)
- [Magento 2.4.6 p2](#Magento)
- [Magento ElasticSearch](#Magento-ElasticSearch)
- [Magento OpenSearch](#Magento-OpenSearch)


## Nginx 
```
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \ | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \ http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \ | sudo tee /etc/apt/sources.list.d/nginx.list
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \ | sudo tee /etc/apt/preferences.d/99nginx
sudo apt update
sudo apt install nginx=1.22.*
```

## PHP
```
sudo apt update
sudo apt install -y libxml2-dev libzip-dev
sudo add-apt-repository ppa:ondrej/php
sudo apt update
<<<<<<< HEAD
sudo apt install -y php8.2 php8.2-fpm php8.2-bcmath php8.2-common php8.2-curl php8.2-xml php8.2-gd php8.2-intl php8.2-cli php8.2-mbstring php8.2-mysql php8.2-soap php8.2-xsl php8.2-zip

Altere os valores abaixo nos arquivos citados
=======
sudo apt install -y php8.2 php8.2-fpm php8.2-bcmath php8.2-ctype php8.2-curl php8.2-dom php8.2-fileinfo php8.2 filter php8.2 gd php8.2 hash php8.2-iconv php8.2-intl php8.2 json php8.2 libxml php8.2-mbstring php8.2-openssl php8.2-pcre php-mysql php8.2-simplexml php8.2-soap php8.2-sockets php8.2-sodium php8.2-spl php8.2-tokenizer php8.2-xmlwriter php8.2-xsl php8.2-zip php8.2-zlib
#verificar modulos instalados
php --modules
>>>>>>> 42f7a933a45eb2b2f6906679f55bad6af67b34d6
sudo nano /etc/php/8.2/cli/php.ini
    memory_limit=2G
    realpath_cache_size=10M
    realpath_cache_ttl=7200
    date.timezone=America/Sao_Paulo

sudo nano /etc/php/8.2/fpm/php.ini
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

## Config Nginx 
```
sudo nano /etc/nginx/nginx.conf
    server_names_hash_bucket_size 64;
sudo systemctl restart nginx
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
sudo nano /etc/nginx/sites-available/ecommerce

// escrever dentro o conteúdo abaixo
upstream fastcgi_backend
{
     server  unix:/run/php/php8.2-fpm.sock;
}
server {
        listen 80;
        listen [::]:80;
        server_name svpr-ecommerce3 svpr-ecommerce3.microware.com.br;
        set $MAGE_ROOT /var/www/ecommerce;
        include /var/www/ecommerce/nginx.conf.sample;
        client_max_body_size 2M;

        access_log /var/log/nginx/magento.access;
        error_log /var/log/nginx/magento.error;
}
// fim

sudo ln -s /etc/nginx/sites-available/ecommerce /etc/nginx/sites-enabled/
```

## OpenSearch
```

wget https://artifacts.opensearch.org/releases/bundle/opensearch/2.5.0/opensearch-2.5.0-linux-x64.deb
sudo dpkg -i opensearch-2.5.0-linux-x64.deb
sudo systemctl enable opensearch
sudo systemctl start opensearch
sudo systemctl status opensearch

# Edit the sysctl config file
sudo nano /etc/sysctl.conf

# Add a line to define the desired value
# or change the value if the key exists,
# and then save your changes.
vm.max_map_count=262144

# Reload the kernel parameters using sysctl
sudo sysctl -p

# Verify that the change was applied by checking the value
cat /proc/sys/vm/max_map_count

./opensearch-tar-install.sh
curl -X GET https://localhost:9200 -u 'admin:admin' --insecure

reposta esperada:

{    "name":"hostname",
    "cluster_name":"opensearch",
    "cluster_uuid":"QqgpHCbnSRKcPAizqjvoOw",
    "version":{       "distribution":"opensearch",
       "number":<version>,
       "build_type":<build-type>,
       "build_hash":<build-hash>,
       "build_date":<build-date>,
       "build_snapshot":false,
       "lucene_version":<lucene-version>,
       "minimum_wire_compatibility_version":"7.10.0",
       "minimum_index_compatibility_version":"7.0.0"
    },
    "tagline":"The OpenSearch Project: https://opensearch.org/"
 }

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
flush privileges;
CREATE DATABASE magento;
SHOW DATABASES;
CREATE USER 'magento'@'localhost' IDENTIFIED BY '3Comm3rc3';
GRANT ALL PRIVILEGES ON *.* to 'magento'@'localhost';
quit;
```

## RabbitMQ
```
nano rabbitmq.sh
//inicio
#!/bin/sh

sudo apt-get install curl gnupg apt-transport-https -y

## Team RabbitMQ's main signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor | sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null
## Community mirror of Cloudsmith: modern Erlang repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null
## Community mirror of Cloudsmith: RabbitMQ repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg > /dev/null

## Add apt repositories maintained by Team RabbitMQ
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases
##
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu jammy main

## Provides RabbitMQ
##
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu jammy main
EOF

## Update package indices
sudo apt-get update -y

## Install Erlang packages
sudo apt-get install -y erlang-base=1:25* \
                        erlang-asn1=1:25* erlang-crypto=1:25* erlang-eldap=1:25* erlang-ftp=1:25* erlang-inets=1:25* \
                        erlang-mnesia=1:25* erlang-os-mon=1:25* erlang-parsetools=1:25* erlang-public-key=1:25* \
                        erlang-runtime-tools=1:25* erlang-snmp=1:25* erlang-ssl=1:25* \
                        erlang-syntax-tools=1:25* erlang-tftp=1:25* erlang-tools=1:25* erlang-xmerl=1:25*

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server=3.11.* -y --fix-missing
//fim

sudo chmod 777 rabbitmq.sh
./rabbitmq.sh

```

## Redis
```
wget https://download.redis.io/releases/redis-7.0.13.tar.gz
tar -xzvf redis-7.0.13.tar.gz
cd redis-7.0.13
./configure
make
sudo make install
redis-cli ping
redis-server
```

## Varnish
```
curl -s https://packagecloud.io/install/repositories/varnishcache/varnish73/script.deb.sh | sudo bash
sudo apt-get install varnish=7.3.0-1~jammy
sudo systemctl start varnish
sudo systemctl status varnish
```


## Magento
```
cd /var/www
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.6-p2 /var/www/ecommerce
Public Key: 7d8df27f946bc44bfdbdb008cafa315b
Private Key: ee253cef3714a26598c0cf6447b5dce2

sudo nginx -s reload
sudo nginx -t

cd /var/www/ecommerce
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :www-data .
chmod u+x bin/magento
``` 

## Config Magento
``` 
<<<<<<< HEAD
bin/magento setup:install --base-url=http://svpr-ecommerce3.microware.com.br --backend-frontname=admin --db-host=localhost --db-name=magento --db-user=magentoUser --db-password=3C0mm3rc3 --admin-firstname=eCommerce --admin-lastname=Microware --admin-email=ecommerce@microware.com.br --admin-user=admin --admin-password=3C0mm3rc3 --language=pt_BR --currency=BRL --timezone=America/Sao_Paulo --use-rewrites=1 --search-engine=opensearch --opensearch-host=localhost --opensearch-port=9200 --opensearch-index-prefix=magento2
=======
bin/magento setup:install --base-url=http://svhr-ecommerce2.microware.com.br --backend-frontname=admin --db-host=localhost --db-name=magento --db-user=magentoUser --db-password=3C0mm3rc3 --admin-firstname=eCommerce --admin-lastname=Microware --admin-email=ecommerce@microware.com.br --admin-user=admin --admin-password=3C0mm3rc3 --language=pt_BR --currency=BRL --timezone=America/Sao_Paulo --use-rewrites=1 --search-engine=elasticsearch8 --elasticsearch-host=localhost --elasticsearch-port=9200 --elasticsearch-index-prefix=magento2

``` 

## Magento OpenSearch
``` 
bin/magento setup:install --base-url=http://svhr-ecommerce2.microware.com.br --backend-frontname=admin --db-host=localhost --db-name=magento --db-user=magento --db-password=3C0mm3rc3 --admin-firstname=eCommerce --admin-lastname=Microware --admin-email=ecommerce@microware.com.br --admin-user=admin --admin-password=3C0mm3rc3 --language=pt_BR --currency=BRL --timezone=America/Sao_Paulo --use-rewrites=1 --search-engine=opensearch --opensearch-host=localhost --opensearch-port=9200 --opensearch-index-prefix=magento2
>>>>>>> 42f7a933a45eb2b2f6906679f55bad6af67b34d6

``` 
