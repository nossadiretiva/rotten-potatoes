# Aplicação Rotten-Potatoes

A Rotten-Potatoes é uma aplicação web construída em ***Python*** que realiza avaliações em filmes cadastrados.

## Criando a imagem para executar o container

Foi criado dentro do projeto um arquivo `Dockerfile` contendo o passo-a-passo (receita de bolo) de criação da imagem *Docker* que posteriormente será utilizada para execução da aplicação em um container.

    FROM  python:3.10.1-slim
    RUN  pip  install  --upgrade  pip
    WORKDIR  /app
    
    COPY  ./requirements.txt  ./requirements.txt
    RUN  pip3  install  -r  requirements.txt
    RUN  pip  install  gunicorn
    
    COPY  .  .
    EXPOSE  5000
    
    ENV  FLASK_APP=./app.py
    ENV  FLASK_ENV=development
    
    CMD  ["gunicorn",  "-c",  "config.py",  "--bind",  "0.0.0.0:5000",  "app:app"]

## Ignorando arquivos desnecessários na criação da imagem

No diretório da aplicação podem existir arquivos indesejados e/ou desnecessários que seriam copiados com o comando `COPY` e que podem ser ignorados durante o processo de criação da imagem *Docker*.

Para isso foi criado no mesmo diretório onde se encontram os arquivos do projeto o arquivo `.dockerignore` que possui uma lista (com o mesmo padrão do arquivo `.gitignore`) com os diretórios/arquivos que serão ignorados no momento da execução da cópia dos arquivos para dentro da imagem.

## Criando a aplicação em uma única linha de comando

Para criação da aplicação (site e banco de dados) em uma única linha de comando precisamos utilizar o *Docker Compose*, onde vamos gerar a imagem *Docker* a ser utilizada para criar e executar o container da aplicação e criar e executar o container com o banco de dados da aplicação.

Foi criado, dentro do diretório da aplicação, um arquivo chamado `docker-compose` contendo o script *yaml* de criação dos objetos necessários para execução da aplicação, como mostrado abaixo:

    version: '3.8'
    
    services:
       app:
          build:
             context: src
             dockerfile: Dockerfile
          container_name: app-rotten-potatoes
          image: nossadiretiva/app-rotten-potatoes:v1
          ports:
             - 5000:5000
          networks:
             - rotten_network
          depends_on:
             - mongodb
          environment:
             MONGODB_DB: admin
             MONGODB_HOST: rotten-mongodb
             MONGODB_PORT: 27017
             MONGODB_USERNAME: rottenuser
             MONGODB_PASSWORD: rottenpwd
    
       mongodb:
          container_name: rotten-mongodb
          image: mongo:5.0.5
          ports:
             - 27017:27017
          networks:
             - rotten_network
          volumes:
             - mongodb_vol:/data/db
          environment:
             MONGO_INITDB_ROOT_USERNAME: rottenuser
             MONGO_INITDB_ROOT_PASSWORD: rottenpwd
             MONGO_INITDB_DATABASE: admin
    
    volumes:
       mongodb_vol:
    
    networks:
       rotten_network:
          driver: bridge

## Realizando teste com o *Docker Compose*

Para realização dos testes com script configurado no arquivo `docker-compose`, executamos o seguinte comando:

    docker-compose up -d

> Esse comando é para criar todos os objetos configurados no arquivo `docker-compose`.

Após a execução do comando teremos os seguintes containers em execução:

    CONTAINER ID   IMAGE                                  COMMAND                  CREATED       STATUS       PORTS                      NAMES
    f8b7745ecb3a   nossadiretiva/app-rotten-potatoes:v1   "gunicorn -c config.…"   6 hours ago   Up 6 hours   0.0.0.0:5000->5000/tcp     app-rotten-potatoes
    3a452da8c865   mongo:5.0.5                            "docker-entrypoint.s…"   6 hours ago   Up 6 hours   0.0.0.0:27017->27017/tcp   rotten-mongodb

Quando a aplicação for acessada através da URL http://localhost:5000 teremos o seguinte resultado:

![Página de teste](https://github.com/nossadiretiva/imagens/blob/master/teste_rotten_potatoes.png?raw=true)
