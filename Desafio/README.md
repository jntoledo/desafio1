# **Desafio Mandic**

**Nesse tutorial iremos instalar e consigurar o Nginx, Magento, Wordpress e Tomcat.**

## **Instalando Nginx**

Primeiro vamos instalar o epel, para isso execute o comando:

```sh
yum -y install epel-release
```

Após isso vamos instalar o Nginx

```sh
yum -y install nginx
```

Após a instalação vamos iniciar o Nginx e adicionar ele para que ele inicie com o Boot.

```sh
systemctl start nginx
systemctl enable nginx
```

Agora o Nginx esta iniciado na porta 80, para chegar, utilize o comando:

```sh
netstat -plntu
```
Caso apareça 'command not found'instale o net-toolslike:

```sh
yum -y install net-tools
```

## **Agora vamos instaalar o PHP-FPM**

Vamos fazer o Download do repositório do webtatic, para isso execute o comando:

```sh
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

Agora, rode o comando abaixo para instalar o PHP-FPM7, com todos os pacotes e extensões necessários para o Magento e Wordpress.

```sh
yum -y install php70w-fpm php70w-mcrypt php70w-curl php70w-cli php70w-mysql php70w-gd php70w-xsl php70w-json php70w-intl php70w-pear php70w-devel php70w-mbstring php70w-zip php70w-soap
```

Após finalizar a instalação do PHP-FPM7, nós precisamos configurar ele.Vamos começar a configuração pelo arquivo php.ini logo depois o arquivo de configuração do php-fpm que é o 'www.conf'.

Alterar o php.ini com o **vim** no caminho abaixo.

```sh
vim /etc/php.ini
```

Tire o comentário da linha: cgi.fix_pathinfo (linha 762) e altere o valor para 0.

> cgi.fix_pathinfo=0

Configure agora o **memory limit, max execution time** e habilite o **zlib output compression**, conforme abaixo.

```sh
memory_limit = 512M
max_execution_time = 1800
zlib.output_compression = On
```
Tire o comentário da linha **session path directory** e altere conforme abaixo.

> session.save_path = "/var/lib/php/session"

Save and exit.

Agora, vamos editar o **www.conf** com o vim.

```sh
vim /etc/php-fpm.d/www.conf
```

PHP-FPM7 precisa rodar com o usuário e o grupo do **nginx** altere conforme abaixo.

```sh
user = nginx
group = nginx
```

Nós vmaos robar o **php-fpm** com o arquivo de soket e não com a porta servidor. Altere o valor da linha **listen** com o caminho abaixo.

> listen = /var/run/php/php-fpm.sock

O arquivo de socket precisa possuir o usuário e grupo do 'nginx'. Tire o comentário das linhas e altere conforme abaixo.

```sh
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

Finalmente, tire o comentário do **PHP-FPM environment** conforme abaixo.
```sh
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Save and exit.

Crie agora um novo diretório para o path de sessão e também para o arquivo php sock php. Altere também de permissão para o usuário e grupo 'nginx' realizar alterações na pasta.

```sh
mkdir -p /var/lib/php/session/
chown -R nginx:nginx /var/lib/php/session/
mkdir -p /run/php/
chown -R nginx:nginx /run/php/
```

PHP-FPM7 esta completamente instalado e configurado. Vamos startar o daemon e habilitar para ele iniciar no boot. Execute os comandos abaixo.
```sh
systemctl start php-fpm
systemctl enable php-fpm
```
## **Vamos instalar o MySQL**

Um dos requisitos para instalar o Magento 2.1, é ter a versão atual do MySQL. Nesse tutorial vamos instalar a versão 5.7. Vamos instalar através do reṕsitório official, vamos adicionar o repositório primeiro. Faça o download e adicione o novo instalador do MySQL 5.7.

```sh
yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm.
yum install mysql-community-server
```
Quando finalizar a instalação inicie ele e adicione para que ele inicie com o boot.
```sh
systemctl start mysqld
systemctl enable mysqld
```

MySQL 5.7 vem instalado com uma senha padrão de root. Para visualiar a senha execute o comando abaixo para visualizar e anotar a mesma.

```sh
grep 'temporary' /var/log/mysqld.log
```
Com a senha anotada, vamos trocar a senha de root. Conecte-se ao shell do mysql com o usuário root e a senha padrão.
```sh
mysql -u root -p
TYPE DEFAULT PASSWORD
```
Altere a senha padrão. Precisamos alterar a mesma para uma senha segura. Conforme comando abaixo.
```sh
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Senhasegura@aqui';
flush privileges;
```

A senha padrão do root foi alterada. Agora vamos criar um user do banco para o magento.

Precisamos criar um novo banco com o nome **magento**, e também o nome de usuário e senha para ele. No caso iremos criar usuário:magento e senha:Magento123@.Vamos dar total privilégios ao usuário magento ao banco.

```sh
create database magentodb;
create user magentouser@localhost identified by 'Magento123@';
grant all privileges on magentodb.* to magentouser@localhost identified by 'Magento123@';
flush privileges;
```
MySQL 5.7 foi instalado e configurado.

## **Vamos Instalar agora o Magento**

**Primeiro vamos Instalar o PHP Composer**

Para instalar execute os comandos:

```ssh
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
composer -v
```

**Vamos instalar o Magento**

Vamos fazer o download e extrair o Magento.

```sh
cd /var/www/
wget https://github.com/magento/magento2/archive/2.1.zip
```
Caso não tenha instalado, instale o unzip

yum -y install unzip

Extraia o magento e renomeie o direitório para 'magento2'

unzip 2.1.zip
mv magento2-2.1 magento2

**Instale as dependências do PHP**

cd magento2
composer install -v

Configure Magento Virtual Host

Vá até o diretório do Nginx e crie um novo **virtual host** chamado **magento.conf** no diretório **conf.d**.

cd /etc/nginx/
vim conf.d/magento.conf

Copie e cole a configuração abaixo.

upstream fastcgi_backend {
        server  unix:/run/php/php-fpm.sock;
}

server {

        listen 80;
        server_name magento.hakase-labs.com;
        set $MAGE_ROOT /var/www/magento2;
        set $MAGE_MODE developer;
        include /var/www/magento2/nginx.conf.sample;
}
Save and exit.

Agora teste a cofniguração, e depois reinicie o serviço do Nginx.

nginx -t
systemctl restart nginx

**Instalando o Magento 2.1**

**Antes de tudo acesso o sie do mangento e crie uma conta.**

Vá até o diretório **magento2** para instalar o Magento com o comando:

cd /var/www/magento2

Rode o comando abaixo e se certifique de ter configurado corretamente. Conforme instruções.

bin/magento setup:install --backend-frontname="adminlogin" \
--key="biY8vdWx4w8KV5Q59380Fejy36l6ssUb" \
--db-host="localhost" \
--db-name="magentodb" \
--db-user="magentouser" \
--db-password="Magento123@" \
--language="en_US" \
--currency="USD" \
--timezone="America/New_York" \
--use-rewrites=1 \
--use-secure=0 \
--base-url="http://magento.hakase-labs.com" \
--base-url-secure="https://magento.hakase-labs.com" \
--admin-user=adminuser \
--admin-password=admin123@ \
--admin-email=admin@newmagento.com \
--admin-firstname=admin \
--admin-lastname=user \
--cleanup-database
Change value for:

--backend-frontname: Pagina do login de admin do Magento
--key: key gerada na criação da conta
--db-host: host do banco localhost
--db-name: nome do banco 'magentodb'
--db-user: usuário do banco 'magentouser'
--db-password: senha do usuário 'Magento123@'
--base-url: URL da sua loja
--admin-user: coloque um nome para o usuário admin
--admin-password: senha para esse usuário admin
--admin-email: e-mail da conta criada

Magento 2.1 está instalado. Rode o comando abaixo, para alterar a permissão do diretório, para deixar o usuário e grupo **Nginx** com permissão de acesso total a pasta.

chmod 700 /var/www/magento2/app/etc
chown -R nginx:nginx /var/www/magento2
Configure Magento Cron

Esse cronjob precisar do **Magento indexer**. Crie um novo usuário cronjob **nginx**.

crontab -u nginx -e

Cole a configuração abaixo.

* * * * * /usr/bin/php /var/www/magento2/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/magento2/var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/magento2/update/cron.php >> /var/www/magento2/var/log/update.cron.log
* * * * * /usr/bin/php /var/www/magento2/bin/magento setup:cron:run >> /var/www/magento2/var/log/setup.cron.log
Save and exit.

Configure agora o **SELinux and Firewalld**. Execute os comandos abaixo.

sestatus

SELinux is in 'Enforcing' mode.

Instale o SELinux management tool 'policycoreutils-python' com o comando yum abaixo.

yum -y install policycoreutils-python

Vá até o diretório '/var/www/' directory.

cd /var/www/

Rode os comandos abaixo, para mudar a segurança para o Magento e seus arquivos.

```sh
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/app/etc(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/var(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/pub/media(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/pub/static(/.*)?'
restorecon -Rv '/var/www/magento2/'
```

Instale o firewalld package se você não tiver instalado no seu servidor.

yum -y install firewalld
systemctl start firewalld
systemctl enable firewalld

Abra as portas para HTTP and HTTPS para acessar a URL do Magento.

firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

Verifique as portas abertas com o comando abaixo.

firewall-cmd --list-all

Abra a sua URL do magento para testar. Magento já esta instalado e configurado.

## **Vamos instalar o Wordpress**
