# Essa automação foi criada para executar em ambientes Centos 7.


## Para executar a configuração, você precisa alterar algumas configs.

1º No arquivo **ansible.cfg**, altere o caminho da linha **inventory** e colocar o caminho que esta salvo esse projeto na sua máquina.

2º Adicione o IP ou nome da máquina que será executado esse playbook. ( se for conectar via ssh na máquina, coloquei a chave ao lado do ip, conforme no documento).

3º Vá até a pasta: **roles > nginx > defaults > main.yml** e altere a variável **seu_site** e coloque o IP ou endereço do seu blog.

Feito isso, basta executar o playbook **blog.yml** que irá instalar e configurar o wordpress e a máquina.