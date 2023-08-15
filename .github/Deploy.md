# Deploy Laravel com Nginx e MySQL

## Instalação do PHP

Primeiro vamos instalar a o PHP 8.1 e as e as extensões necessárias para o Laravel e o MySQL.

```bash
sudo apt install php8.1-fpm php-mysql php-mbstring php-xml php-bcmath php-curl
```

## Instalação do Composer

Primeiro vamos instalar as extensões necessárias para o Composer.

```bash
sudo apt install php-cli unzip
```

Agora vamos baixar o Composer.

```bash
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
```

Agora vamos verificar se o Composer foi baixado corretamente.

```bash
HASH="$(curl -sS https://composer.github.io/installer.sig)"
echo $HASH
php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

Agora vamos instalar o Composer.

```bash
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

## Instalação do Node

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

## Configurando o MySQL

Vamos entrar no MySQL.

```bash
mysql -u root -p
```

Agora vamos criar um usuário para o Laravel.

```sql
CREATE USER 'laravel'@'localhost' IDENTIFIED BY 'password';
```

Agora vamos criar um banco de dados para o Laravel.

```sql
CREATE DATABASE laravel;
```

Agora vamos dar as permissões necessárias para o usuário do Laravel.

```sql
GRANT ALL PRIVILEGES ON laravel.* TO 'laravel'@'localhost';
```

Agora vamos sair do MySQL.

```sql
exit
```

## Instalação do Nginx

Primeiro vamos instalar o Nginx.

```bash
sudo apt install nginx
```

Agora vamos habilitar o Nginx no Firewall.

```bash
sudo ufw allow 'Nginx Full'
```

## Preparando a aplicação

Vamos entrar na pasta do Nginx.

```bash
cd /var/www/
```

Vamos clonar a aplicação.

```bash
sudo git clone https://github.com/treinaweb/treinaweb-projeto-pratico-deploy-laravel-vps.git
```

Vamos dar as permissões necessárias para a pasta da aplicação.

```bash
sudo chown -R treinaweb:www-data treinaweb-projeto-pratico-deploy-laravel-vps
```

Vamos renomear e entrar na pasta da aplicação.

```bash
sudo mv treinaweb-projeto-pratico-deploy-laravel-vps laravel
cd laravel
```

Agora vamos instalar as dependências do Laravel.

```bash
composer install
```

Agora vamos criar o arquivo de configuração do Laravel.

```bash
cp .env.example .env
```

Precisamos editar o arquivo de configuração do Laravel, altere as seguintes linhas.

```bash
nano .env
```

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://endereço_do_servidor
DB_CONNECTION=mysql
DB_HOST=endereço_do_servidor
DB_DATABASE=nome_do_banco_de_dados
DB_USERNAME=usuário_do_banco_de_dados
DB_PASSWORD=senha_do_usuário
```

Agora vamos gerar a chave da aplicação.

```bash
php artisan key:generate
```

Agora vamos criar as tabelas do banco de dados.

```bash
php artisan migrate
```

Agora vamos criar o link simbólico para o armazenamento público.

```bash
php artisan storage:link
```

Agora vamos instalar as dependências do front-end.

```bash
npm install
```

Agora vamos compilar os assets.

```bash
npm run build
```

Também precisamos dar as permissões necessárias para a pasta de armazenamento e de cache.

```bash
sudo chown -R www-data:www-data storage
sudo chown -R www-data:www-data bootstrap/cache
```

## Otimização do Laravel

Vamos otimizar o autoloader do Composer para que ele possa rapidamente encontrar as classes necessárias para carregar a aplicação.

```bash
composer install --optimize-autoloader --no-dev
```

Vamos realizar o cache dos arquivos de configuração, rotas, eventos e views.

```bash
php artisan config:cache
php artisan event:cache
php artisan route:cache
php artisan view:cache
```

## Configurando o Nginx

Vamos criar o arquivo de configuração do Nginx.

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Agora vamos adicionar o seguinte conteúdo.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /var/www/laravel/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Agora vamos criar o link simbólico para o arquivo de configuração.

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

Agora vamos testar a configuração do Nginx.

```bash
sudo nginx -t
```

Agora vamos reiniciar o Nginx.

```bash
sudo systemctl restart nginx
```

## Configurando SSL

Primeiro vamos instalar o Certbot.

```bash
sudo snap install --classic certbot
```

Agora vamos gerar o certificado.

```bash
sudo certbot --nginx -d www.example.com
```

Agora vamos reiniciar o Nginx.

```bash
sudo systemctl restart nginx
```
