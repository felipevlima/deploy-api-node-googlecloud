<h1 align="center">
  <img alt="GCloud" title="Deploy Node.JS on Google Cloud Platform" src="https://www.ixreach.com/wp-content/uploads/Official-Google-Cloud.png" width="200px" />
</h1>

<h3 align="center">
  Deploy Node.JS on Google Cloud Platform
</h3>

<p align="center">List Version: 1.0.0-alpha</p>

## :computer: Criando uma instancia e onfigurando Firewall

- [ ] Criar instancia no Compute Engine
	- [ ] Firewall - Allow HTTP e Allow HTTPS
	- [ ] Na sua maquina local, entre em ```$ cd ~/.ssh/``` e rode o comando 
	- [ ] ```$ ssh-keygen```
		- [ ] no nome bote id_rsa e de enter para o resto
	- [ ] ```$ cat id_rsa.pub```
	- [ ] copiar a public_key
	- [ ] Entrar em configuração da instancia e adicionar na ssh Keys alterando o final da key para o usuário que você vai acessar dentro da instância (no caso deploy)

- [ ] Criar regra de Firewall para liberar porta 3333
	- [ ] Vá em VPC Network -> Firewall rules -> Create Firewall Rules
		- [ ] Escolha um nome
		- [ ] Em network deixe como default
		- [ ] Priority = 1000
		- [ ] Direction of traffic = Ingress
		- [ ] Action on match = Allow
		- [ ] Targets = All instances in the network
		- [ ] Source filter = IP ranges
		- [ ] Protocol and ports = Specified protocols and ports -> tcp: 3333
		- [ ] Save

## Configurando servidor

- [ ] Acesse sua maquina virtual no ssh web
	- [ ] Altere para user root ```sudo su```
	- [ ] ```$ apt update```
	- [ ] ```$ apt upgrade```
	- [ ] ```$ adduser deploy```
	- [ ] ```$ user mod -aG sudo deploy```
	- [ ] ```$ cp ~/.ssh/authorized_keys /home/deploy```
	- [ ] ```$ cd /home/deploy```
	- [ ] ```$ mkdir .ssh```
	- [ ] ```$ cd .ssh/```
	- [ ] ```$ cp ~/.ssh/authorized_keys .```
	- [ ] ```$ chown deploy:deploy authorized_keys```
	- [ ] ```$ exit```
	- [ ] Desconecte da sua maquina virtual

- [ ] Acesse sua maquina virtual com usuário deploy
	- [ ] Digite no terminal da sua maquina ```deploy@ip_da_maquina```
	- [ ] ```$ git -v```
	- [ ] ```$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -```
	- [ ] ```$ sudo apt install nodejs```
	- [ ] ```$ node -v```
	- [ ] ```$ npm -v```

- [ ] Instale o Docket com os seguintes comandos:


```console
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```console
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

```console
$ sudo apt install docker-ce
```

```console
$ sudo systemctl status docker
```

## Clonando a aplicação

- [ ] ```$ git clone https://github.com/GiuSantos0/backend-auth app```
- [ ] ```$ cd app```
- [ ] ```$ npm install```
- [ ] ```$ cp .env.example .env```
- [ ] ```$ vim .etc```
	- [ ] Alterar NODE_ENV para production
	- [ ] Botar o ip da maquina em APP_URL=http://ip:3333


## Configurando Serviços

- [ ] ```$ docker ps``` vai dar permission denied, execute os comandos:
	- [ ] ```$ sudo groupadd docker```
	- [ ] ```$ sudo usermod -aG docker $USER```
	- [ ] deslogar e logar na maquina

- [ ] ```$ cd app```
- [ ] Criar serviços (exemplos a seguir)

- postgres

```
$ docker run --name postgres -e POSTGRES_PASSWORD=bootcampdeploy -p 5432:5432 -d -t postgres
```

- mongodb

```
$ docker run --name mongo -p 27017:27017 -d -t mongo
```

- redis

```
$ docker run --name redis -p 6379:6379 -d -t redis:alpine
```

- [ ] ```$ docker ps```

- [ ] ```$ vim .env```
- [ ] Em ```MONGO_URL``` adicionar ```mongodb://localhost:27017/bootcampnodejs```
- [ ] Em Database: 

```
DB_HOST=localhost
DB_USER=postgres
DB_PASS=bootcampdeploy
DB_NAME=bootcampnodejs
```

- [ ] Criando banco de dados:
	- [ ] ```$ docker exec -i -t postgres /bin/sh```
	- [ ] ```# su postgres```
	- [ ] ```$ psql```
	- [ ] ```postgres=# CREATE DATABASE bootcampnodejs```
	- [ ] ```postgres=# \q```
	- [ ] ```$ exit```
	- [ ] ```# exit```

## Se estiver utilizando SUCRASE

- [ ] Em sua maquina abra o projeto e acesse package.json
- [ ] adicione dois comandos em scripts:

```
…
"scripts": {
	…
	"build": "sucrase ./src -d ./dist --transforms imports",
	"start": "node dist/server.js",
}
…
```

- [ ] Fazer push para o GitHub do code atualizado
- [ ] Voltando para o servidor
- [ ] ```$ cd app/```
- [ ] ```$ git pull```
- [ ] ```$ npm run build```
- [ ] ```$ npm run start```
- [ ] Liberar porta no servidor ```$ sudo ufw allow 3333```
- [ ] Fazer migration: ```$ npx sequelize db:migrate```

## DICAS DE SSH

- [ ] ```$ sudo vim /etc/ssh/sshd_config```
- [ ] Adicionar no final do arquivo:

```
ClientAliveInterval 30
TCPKeepAlive yes
ClientAliveCountMax 99999
```

- [ ] Reiniciar Service do ssh ```$ sudo service sshd restart```
- [ ] Desconectar e contra no servidor

## Configurar NGINX

- [ ] ```$ sudo apt install nginx```
- [ ] ```$ sudo ufw allow 80```
- [ ] Redirecionar portas:
	- [ ] ```$ sudo vim /etc/nginx/sites-available/default```
	- [ ] Remover comentarios
	- [ ] Remover linha root /var/www/html
	- [ ] Remover linha index index.html index.nginx-debian.html
	- [ ] Remover todo conteúdo de location / {}
	- [ ] Em location / { } adicionar:
		- [ ] ```proxy_pass http://localhost:3333;```
		- [ ] ```proxy_http_version 1.1;```
		- [ ] ```proxy_set_header Upgrade #http_upgrade;```
		- [ ] ```proxy_set_header Connection 'upgrade';```
		- [ ] ```proxy_set_header Host $host;```
		- [ ] ```proxy_cache_bypass $http_upgrade;```
	- [ ] ```$ sudo service nginx restart``` 
	- [ ] ```$ sudo nginx -t``` 
- [ ] ```$ cd app```
- [ ] ```$ npm run start```

## Utilizando PM2 para manter a aplicação rodando

- [ ] ```$ sudo npm install -g pm2```
- [ ] ```$ pm2 start dist/server.js```
- [ ] Ver os services que estão rodando: ```pm2 list```
- [ ] Rodar comando caso servidor restarte ```pm2 startup systemd```
- [ ] Adicionar variável que foi passado após executar comando acima. Exemplo: ```sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u deploy --hp /home/deploy```

## Integração continua (em breve)

... 

## 🤔 Como contribuir

- Faça um fork desse repositório;
- Cria uma branch com a sua feature: `git checkout -b minha-feature`;
- Faça commit das suas alterações: `git commit -m 'feat: Minha nova feature'`;
- Faça push para a sua branch: `git push origin minha-feature`.

Depois que o merge da sua pull request for feito, você pode deletar a sua branch.

---

Make with ♥ by Felipe Vieira :wave:
