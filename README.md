````markdown
# 🐾 Projeto SeuPet - Load Balancer com NGINX e Docker Compose

Este projeto demonstra uma aplicação React servida com NGINX em um ambiente com balanceamento de carga, utilizando Docker Compose para orquestrar os containers.

---

## ⚙️ Passo a passo de instalação

### 1️⃣ Clone o repositório

```bash
git clone https://www.github.com/Luna-Alves/SeuPet
cd SeuPet
````

### 2️⃣ Instale as dependências

```bash
npm install
```

### 3️⃣ Gere o build de produção

```bash
npm run build
```

### 4️⃣ Copie a pasta `build` para o diretório raiz do repositório

A pasta `build` será utilizada pelos containers NGINX para servir os arquivos estáticos da aplicação React.

---

## 🔧 Configuração do NGINX

### nginx.conf (responsável pelo Load Balancer)

```nginx
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    server_names_hash_bucket_size 64;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;

    upstream nodes {
        server node1:80;
        server node2:80;
        server node3:80;
        server node4:80;
        server node5:80;
    }

    server {
        listen 80;
        server_name seupet;

        location / {
            proxy_pass http://nodes;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    include /etc/nginx/conf.d/default.conf;
}
```

### default.conf (responsável por servir os arquivos estáticos em cada node)

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
}
```

### Explicação:

* **nginx.conf:** configura o Load Balancer com algoritmo de Round Robin.
* **default.conf:** configura cada container NGINX para servir os arquivos estáticos da aplicação React.

---

## 📦 docker-compose.yml

O Docker Compose será responsável por orquestrar os containers da aplicação: cria 5 servidores NGINX que servem o React e 1 NGINX que faz o balanceamento de carga.

```yaml
version: '3'

services:
  nginx:
    image: nginx:latest
    container_name: nginx-loadbalancer
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - node1
      - node2
      - node3
      - node4
      - node5

  node1:
    image: nginx:latest
    container_name: node1
    volumes:
      - ./build:/usr/share/nginx/html
      - ./default.conf:/etc/nginx/conf.d/default.conf

  node2:
    image: nginx:latest
    container_name: node2
    volumes:
      - ./build:/usr/share/nginx/html
      - ./default.conf:/etc/nginx/conf.d/default.conf

  node3:
    image: nginx:latest
    container_name: node3
    volumes:
      - ./build:/usr/share/nginx/html
      - ./default.conf:/etc/nginx/conf.d/default.conf

  node4:
    image: nginx:latest
    container_name: node4
    volumes:
      - ./build:/usr/share/nginx/html
      - ./default.conf:/etc/nginx/conf.d/default.conf

  node5:
    image: nginx:latest
    container_name: node5
    volumes:
      - ./build:/usr/share/nginx/html
      - ./default.conf:/etc/nginx/conf.d/default.conf
```

---

## 🚀 Executando a aplicação

Dentro do diretório do projeto, execute:

```bash
docker-compose up
```

Os containers serão inicializados e o Load Balancer estará disponível na porta 80.

---

## 🌐 Como acessar a aplicação

Como o domínio `seupet` não existe, é necessário configurar o mapeamento local no arquivo `/etc/hosts`.

1️⃣ Abra o arquivo:

```bash
sudo nano /etc/hosts
```

2️⃣ Adicione a seguinte linha:

```
127.0.0.1 seupet
```

3️⃣ Salve e feche.

Agora, acesse a aplicação via navegador:

```
http://seupet
```

> **Atenção:** Não utilize `https`, pois não há configuração de SSL. Use apenas `http://`.

---

✅ Pronto! Seu Load Balancer com Docker Compose e NGINX está funcionando.

---