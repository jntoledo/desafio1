Desafio Mandic
Nesse tutorial iremos instalar e consigurar o Nginx, Magento, Wordpress e Tomcat.

Instalando Nginx
Primeiro vamos instalar o epel, para isso execute o comando:

yum -y install epel-release
Após isso vamos instalar o Nginx

yum -y install nginx
Após a instalação vamos iniciar o Nginx e adicionar ele para que ele inicie com o Boot.

systemctl start nginx
systemctl enable nginx
Agora o Nginx esta iniciado na porta 80, para chegar, utilize o comando:

netstat -plntu
Caso apareça 'command not found'instale o net-toolslike:

yum -y install net-tools
Agora vamos instaalar o PHP-FPM
Vamos fazer o Download do repositório do webtatic, para isso execute o comando:

rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
Agora, rode o comando abaixo para instalar o PHP-FPM7, com todos os pacotes e extensões necessários para o Magento e Wordpress.

yum -y install php70w-fpm php70w-mcrypt php70w-curl php70w-cli php70w-mysql php70w-gd php70w-xsl php70w-json php70w-intl php70w-pear php70w-devel php70w-mbstring php70w-zip php70w-soap
Após finalizar a instalação do PHP-FPM7, nós precisamos configurar ele.Vamos começar a configuração pelo arquivo php.ini logo depois o arquivo de configuração do php-fpm que é o 'www.conf'.

Alterar o php.ini com o vim no caminho abaixo.

vim /etc/php.ini
Tire o comentário da linha: cgi.fix_pathinfo (linha 762) e altere o valor para 0.

cgi.fix_pathinfo=0

Configure agora o memory limit, max execution time e habilite o zlib output compression, conforme abaixo.

memory_limit = 512M
max_execution_time = 1800
zlib.output_compression = On
Tire o comentário da linha session path directory e altere conforme abaixo.

session.save_path = "/var/lib/php/session"

Salve e saia.

Agora, vamos editar o www.conf com o vim.

vim /etc/php-fpm.d/www.conf
PHP-FPM7 precisa rodar com o usuário e o grupo do nginx altere conforme abaixo.

user = nginx
group = nginx
Nós vmaos robar o php-fpm com o arquivo de soket e não com a porta servidor. Altere o valor da linha listen com o caminho abaixo.

listen = /var/run/php/php-fpm.sock

O arquivo de socket precisa possuir o usuário e grupo do 'nginx'. Tire o comentário das linhas e altere conforme abaixo.

listen.owner = nginx
listen.group = nginx
listen.mode = 0660
Finalmente, tire o comentário do PHP-FPM environment conforme abaixo.

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
Salve e saia.

Crie agora um novo diretório para o path de sessão e também para o arquivo php sock php. Altere também de permissão para o usuário e grupo 'nginx' realizar alterações na pasta.

mkdir -p /var/lib/php/session/
chown -R nginx:nginx /var/lib/php/session/
mkdir -p /run/php/
chown -R nginx:nginx /run/php/
PHP-FPM7 esta completamente instalado e configurado. Vamos startar o daemon e habilitar para ele iniciar no boot. Execute os comandos abaixo.

systemctl start php-fpm
systemctl enable php-fpm
Vamos instalar o MySQL
Um dos requisitos para instalar o Magento 2.1, é ter a versão atual do MySQL. Nesse tutorial vamos instalar a versão 5.7. Vamos instalar através do reṕsitório official, vamos adicionar o repositório primeiro. Faça o download e adicione o novo instalador do MySQL 5.7.

yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm.
yum install mysql-community-server
Quando finalizar a instalação inicie ele e adicione para que ele inicie com o boot.

systemctl start mysqld
systemctl enable mysqld
MySQL 5.7 vem instalado com uma senha padrão de root. Para visualiar a senha execute o comando abaixo para visualizar e anotar a mesma.

grep 'temporary' /var/log/mysqld.log
Com a senha anotada, vamos trocar a senha de root. Conecte-se ao shell do mysql com o usuário root e a senha padrão.

mysql -u root -p
TYPE DEFAULT PASSWORD
Altere a senha padrão. Precisamos alterar a mesma para uma senha segura. Conforme comando abaixo.

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Senhasegura@aqui';
flush privileges;
A senha padrão do root foi alterada. Agora vamos criar um user do banco para o magento.

Precisamos criar um novo banco com o nome magento, e também o nome de usuário e senha para ele. No caso iremos criar usuário:magento e senha:Magento123@.Vamos dar total privilégios ao usuário magento ao banco.

create database magentodb;
create user magentouser@localhost identified by 'Magento123@';
grant all privileges on magentodb.* to magentouser@localhost identified by 'Magento123@';
flush privileges;
MySQL 5.7 foi instalado e configurado.

Vamos Instalar agora o Magento
Primeiro vamos Instalar o PHP Composer

Para instalar execute os comandos:

curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
composer -v
Vamos instalar o Magento

Vamos fazer o download e extrair o Magento.

cd /var/www/
wget https://github.com/magento/magento2/archive/2.1.zip
Caso não tenha instalado, instale o unzip

yum -y install unzip
Extraia o magento e renomeie o direitório para 'magento2'

unzip 2.1.zip
mv magento2-2.1 magento2
Instale as dependências do PHP

cd magento2
composer install -v
Configure Magento Virtual Host

Vá até o diretório do Nginx e crie um novo virtual host chamado magento.conf no diretório conf.d.

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
Salve e saia do arquivo.

Agora teste a configuração, e depois reinicie o serviço do Nginx.

nginx -t
systemctl restart nginx
Instalando o Magento 2.1

Antes de tudo acesso o sie do mangento e crie uma conta.

Vá até o diretório magento2 para instalar o Magento com o comando:

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
Magento 2.1 está instalado. Rode o comando abaixo, para alterar a permissão do diretório, para deixar o usuário e grupo Nginx com permissão de acesso total a pasta.

chmod 700 /var/www/magento2/app/etc
chown -R nginx:nginx /var/www/magento2
Vmaos configurar o Magento Cron. Esse cronjob precisar do Magento indexer. Crie um novo usuário cronjob nginx.

crontab -u nginx -e
Cole a configuração abaixo.

* * * * * /usr/bin/php /var/www/magento2/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/magento2/var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/magento2/update/cron.php >> /var/www/magento2/var/log/update.cron.log
* * * * * /usr/bin/php /var/www/magento2/bin/magento setup:cron:run >> /var/www/magento2/var/log/setup.cron.log
Salve a config e saia.

Configure agora o SELinux and Firewalld. Execute os comandos abaixo.

sestatus

SELinux is in 'Enforcing' mode.
Instale o SELinux management tool 'policycoreutils-python' com o comando yum abaixo.

yum -y install policycoreutils-python
Vá até o diretório '/var/www/' directory.

cd /var/www/
Rode os comandos abaixo, para mudar a segurança para o Magento e seus arquivos.

semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/app/etc(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/var(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/pub/media(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/magento2/pub/static(/.*)?'
restorecon -Rv '/var/www/magento2/'
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

Vamos instalar o Wordpress
O conteúdo do WordPress pode ser baixado usando os comandos abaixo.

# cd / opt
# wget https://wordpress.org/latest.tar.gz
Quando o download terminar, execute o seguinte comando: extraia-o.

# tar -xvzf latest.tar.gz -C / usr / share / nginx / html /
Em seguida, altere as permissões e a propriedade das pastas de conteúdo do WordPress.

# chown -R nginx: nginx / usr / share / nginx / html / wordpress
# chmod -R 755 / usr / share / nginx / html / wordpress
Agora vamos criar um banco de dados mysql e um usuário para o wordpress. Use o seguinte conjunto de comandos para criar banco de dados e usuário.

# mysql -u root -p
mysql> CREATE DATABASE techoism_db;
mysql> GRANT ALL ON techoism_db. * para 'techoism_user' @ 'localhost' IDENTIFICADO POR 'secretpassword';
mysql> PRIVILÉGIOS DE FLUSH;
mysql> sair
Agora vamos criar o arquivo Virtual Host.Abra o arquivo de configuração do Nginx com qualquer editor adicione as seguintes linhas:

# vi /etc/nginx/conf.d/blog-juan.estudojt.tk
Copie e cole a seguinte config.

server {
        listen 80;
        server_name blog-juan.estudojt.tk;
        root   /var/www/html/blog-juan/wordpress;
        index index.php index.html;
        location / {
                     try_files $uri $uri;
                    }

        location ~ .php$ {
                try_files $uri;
                fastcgi_pass    127.0.0.1:9000;
                fastcgi_index   index.php;
                fastcgi_param SCRIPT_FILENAME /var/www/html/blog-juan/wordpress$fastcgi_script_name;
                include fastcgi_params;
                        }
}
Em seguida, reinicie o serviço nginx para refletir as alterações.

systemctl restart nginx
Configurando a instalação do WordPress. Use o seguinte comando para configurar a instalação do WordPress.

cd /var/www/html/blog-juan/wordpress
cp wp-config-sample.php wp-config.php
chmod 777 wp-config.php
chown nginx: nginx wp-config.php
Abra o arquivo de configuração do wordpress e altere a configuração do MySQL.

vim wp-config.php

// ** Configurações do MySQL - Você pode obter essas informações do seu host ** //
/ ** O nome do banco de dados para WordPress * /
define ('DB_NAME', 'techoism_db');

/ ** nome de usuário do banco de dados MySQL * /
define ('DB_USER', 'techoism_user');

/ ** senha do banco de dados MySQL * /
define ('DB_PASSWORD', 'redhat');

/ ** nome do host MySQL * /
define ('DB_HOST', 'localhost');

/ ** Database Charset para usar na criação de tabelas de banco de dados. * /
define ('DB_CHARSET', 'utf8');

/ ** O tipo Agrupar banco de dados. Não mude isso em caso de dúvida. * /
define ('DB_COLLATE', '');
Instalação do WordPress efetuada. Agora abra seu navegador e digite o endereço configurado.

http://blog-juan.estudojt.tk http://Server-IP

Para finalizar vamos instalar o Tomcat
Pré-requisitos

O usuário que você está efetuando login deve ter privilégios de sudo para poder instalar pacotes.

Instale o OpenJDK

O Tomcat 9 requer Java SE 8 ou posterior. Instalaremos o OpenJDK.

sudo yum install java-1.8.0-openjdk-devel
Vamos criar usuário e grupo do sistema com o diretório inicial /opt/tomcatque executará o serviço Tomcat:

sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
Baixaremos a versão mais recente do Tomcat 9.0.x na página de downloads do Tomcat . Navegue para o /tmp diretório e faça o download do arquivo zip do Tomcat usando o seguinte comando wget :

cd /tmp
wget https://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.27/bin/apache-tomcat-9.0.27.tar.gz
Quando o download estiver concluído, extraia o arquivo tar :

tar -xf apache-tomcat-9.0.27.tar.gz
Mova os arquivos de origem do Tomcat para ele no /opt/tomcat:

sudo mv apache-tomcat-9.0.27 /opt/tomcat/
O usuário do tomcat que configuramos anteriormente precisa ter acesso ao diretório de instalação do tomcat. Execute o seguinte comando para alterar a propriedade do diretório para usuário e grupo tomcat:

sudo chown -R tomcat: /opt/tomcat
Torne os scripts dentro do bindiretório executáveis, através do comando:

sudo sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
Precisamos criar uma unidade systemd.Para fazer o Tomcat funcionar como um serviço, abra seu editor de texto e crie um tomcat.service no caminho /etc/systemd/system/

sudo nano /etc/systemd/system/tomcat.service
Cole o seguinte conteúdo:

[Unit]
Description=Tomcat 9 servlet container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/jre"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"

Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
Salve e feche o arquivo.

Notifique o systemd de que criamos um novo arquivo de unidade, depois ative e verifique se o serviço esta iniciado.

sudo systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat
sudo systemctl status tomcat
Ele irá retornar

● tomcat.service - Tomcat 9 servlet container
   Loaded: loaded (/etc/systemd/system/tomcat.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-11-15 20:47:50 UTC; 4s ago
  Process: 1759 ExecStart=/opt/tomcat/latest/bin/startup.sh (code=exited, status=0/SUCCESS)
 Main PID: 1767 (java)
   CGroup: /system.slice/tomcat.service
Vamos ajustar o firewall para que tenhamos acesso ao tomcat na web através da porta 8080. Use os seguintes comandos para abrir a porta:

sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
Os usuários do Tomcat e suas funções são definidas no tomcat-users.xml. Se você abrir o arquivo, notará que ele é preenchido com comentários e exemplos que descrevem como configurá-lo.

sudo nano /opt/tomcat/latest/conf/tomcat-users.xml
Para adicionar um novo usuário que possa acessar a interface da web do tomcat (manager-gui e admin-gui), é necessário definir o usuário no tomcat-users.xml, como mostrado abaixo. Certifique-se de alterar o nome de usuário e a senha para algo mais seguro:

``h /opt/tomcat/latest/conf/tomcat-users.xml

```
Por padrão, a interface de gerenciamento da web do Tomcat está configurada para permitir o acesso apenas a partir do host local. Se você quiser acessar a interface da web a partir de um IP remoto ou de qualquer lugar que não seja recomendado por ser um risco à segurança, abra os seguintes arquivos e faça as seguintes alterações.

Se você precisar acessar a interface da web de qualquer lugar, abra os seguintes arquivos e comente ou remova as linhas destacadas em amarelo:

/opt/tomcat/latest/webapps/manager/META-INF/context.xml
<Context antiResourceLocking="false" privileged="true" >
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
</Context>

/opt/tomcat/latest/webapps/host-manager/META-INF/context.xml
<Context antiResourceLocking="false" privileged="true" >
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
</Context>
Se você precisar acessar a interface da Web apenas a partir de um IP específico, em vez de comentar os blocos, adicione seu IP público à lista. Digamos que seu IP público seja 10.10.10.10. Você deseja permitir o acesso apenas a partir desse IP:

/opt/tomcat/latest/webapps/manager/META-INF/context.xml
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|41.41.41.41" />
</Context>

/opt/tomcat/latest/webapps/host-manager/META-INF/context.xml
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|41.41.41.41" />
</Context>
A lista de endereços IP permitidos é uma lista separada por barra vertical |. Você pode adicionar endereços IP únicos ou usar expressões regulares.

Reinicie o serviço Tomcat para aplicar as alterações:

sudo systemctl restart tomcat
Tomcat instalado, agora vamos testar. Abra seu navegador e digite:

http://<your_domain_or_IP_address>:8080