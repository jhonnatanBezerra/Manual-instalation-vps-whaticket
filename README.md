# jbtalk-backend

## Realizando atualizações na vps

Atulizações de pacotes e instalação de libs que serão utilizadas pela vps.

```javascript
sudo apt update && sudo apt upgrade -y
```

```javascript
 sudo apt-get install -y libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils git
```

```javascript
sudo apt-get install build-essential
```

## Instalando fail2ban...

**fail2ban** serve para evitar acessos a vps por força bruta, quando um ip tenta acessar a vps e erra a senha varias vezes esse pacote joga o ip para uma black list

```javascript
sudo apt install fail2ban -y && sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### Configurando fail2ban...

```javascript
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## Configurando firewall...

Ative o firewall da vps para entrada e saida de informações controladas.

```javascript
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## Instalando NodeJS 20...

```javascript
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

## Instalando o redis ...

```javascript
sudo apt install redis
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

## Instalando o postgres ...

```javascript
sudo apt-get install postgresql postgresql-contrib -y
sudo systemctl start postgresql.service
sudo passwd postgres
crie-uma-senha
sudo su - postgres
psql -c "ALTER USER postgres WITH PASSWORD 'minha-senha'"
createdb nome-do-banco-de-dados;
exit
```

## Instalando Nginx...

```javascript
sudo apt install nginx -y
```

## Configurando Nginx...

```javascript
sudo rm /etc/nginx/sites-enabled/default
```

### Criando o arquivo de configuração do backend

```javascript
sudo nano /etc/nginx/sites-available/backend
```

### para o backend cole o codigo abaixo

substitua **meubackend.com.br** pelo dominio do seu backend

```javascript
server {
  server_name meubackend.com.br;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

### Criando o arquivo de configuração do backend

```javascript
sudo nano /etc/nginx/sites-available/frontend
```

### para o frontend cole o codigo abaixo

substitua **meufrontend.com.br** pelo dominio do seu backend

```javascript
server {
  server_name meufrontend.com.br;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

### Cria os links simbólicos

```javascript
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled
```

### Adiciona o tamanho máximo do arquivo no nginx.conf

```javascript
sudo nano /etc/nginx/nginx.conf
client_max_body_size 20M; // pode alterar para mais MB se desejar
```

### Testa a configuração e reinicia o nginx

```javascript
sudo nginx -t
sudo service nginx restart
```

## Instalando certbot...

```javascript
sudo apt install snapd -y
sudo snap install --classic certbot
sudo apt update
```

## Instalando o pm2 globalmente...

```javascript
sudo npm install -g pm2
sudo pm2 startup ubuntu -u 'root' // se etiver logado via ssh com outro usuario substirua root pelo usuario logado
```

## PREPARAÇÃO DO PROJETO

Envie os arquivos do seu projeto via SFTP ou via git clone, em seguida navegue ate a pasta de destino do **backend** e rode o seguinte comando

```javascript
npm install
```

### cria o arquivo .env

```javascript
sudo nano .env
```

### preencha os dados do arquivo .env conforme necessario

```javascript
NODE_ENV=
BACKEND_URL=http://localhost
FRONTEND_URL=http://localhost:3000
PROXY_PORT=443
PORT=8080

DB_DIALECT=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USER=user
DB_PASS=senha
DB_NAME=db_name

JWT_SECRET=kZaOTd+YZpjRUyyuQUpigJaEMk4vcW4YOymKPZX0Ts8=
JWT_REFRESH_SECRET=dBSXqFg9TaNUEDXVp6fhMTRLBysP+j2DSqf7+raxD3A=

REDIS_URI=redis://:123456@127.0.0.1:6379
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000

USER_LIMIT=10000
CONNECTIONS_LIMIT=100000
CLOSED_SEND_BY_ME=true

GERENCIANET_SANDBOX=false
GERENCIANET_CLIENT_ID=Client_Id_Gerencianet
GERENCIANET_CLIENT_SECRET=Client_Secret_Gerencianet
GERENCIANET_PIX_CERT=certificado-Gerencianet
GERENCIANET_PIX_KEY=chave pix gerencianet


VERIFY_TOKEN=whaticket

FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET=
```

### roda o build e sobe as migrations e seeds

```javascript
npm run build
npx sequelize db:migrate
npx sequelize db:seed:all
```

### adiciona o servidor backend no pm2

```javascript
pm2 start dist/server.js --name backend --max-memory-restart 400M
sudo pm2 save
```

### Navegue para a pasta frontend

```javascript
cd ../frontend
```

### instala as dependencias do frontend

```javascript
npm install
```

### cria o arquivo .env

```javascript
sudo nano .env
```

### preencha os dados do arquivo .env conforme necessario

```javascript
REACT_APP_BACKEND_URL=
REACT_APP_HOURS_CLOSE_TICKETS_AUTO = 24
REACT_APP_FACEBOOK_APP_ID=
```

Realize o build do frontend

```javascript
npm run build
```

### adiciona o servidor frontend no pm2

```javascript
pm2 start server.js --name frontend --max-memory-restart 400M
sudo pm2 save
```

## Gerando o certificado ssl

```javascript
sudo certbot --nginx
```

---
