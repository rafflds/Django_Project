## Guia de início rápido: Compose e Django
Este guia de início rápido demonstra como usar o Docker Compose para configurar e executar um aplicativo Django/PostgreSQL simples. Antes de começar, instale o Compose .

Defina os componentes do projeto
Para este projeto, você precisa criar um Dockerfile, um arquivo de dependências Python e um docker-compose.ymlarquivo. (Você pode usar uma extensão .ymlou .yamlpara este arquivo.)

Crie um diretório de projeto vazio.

Você pode nomear o diretório com algo fácil de lembrar. Este diretório é o contexto da imagem do seu aplicativo. O diretório deve conter apenas recursos para construir essa imagem.

Crie um novo arquivo chamado Dockerfileno diretório do seu projeto.

O Dockerfile define o conteúdo da imagem de um aplicativo por meio de um ou mais comandos de construção que configuram essa imagem. Depois de construída, você pode executar a imagem em um contêiner. Para obter mais informações Dockerfile, consulte o guia do usuário do Docker e a referência do Dockerfile .

Adicione o seguinte conteúdo ao arquivo Dockerfile.

# syntax=docker/dockerfile:1
FROM python:3
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
Isso Dockerfilecomeça com uma imagem pai do Python 3 . A imagem pai é modificada adicionando um novo codediretório. A imagem pai é modificada posteriormente com a instalação dos requisitos do Python definidos no requirements.txtarquivo.

Salve e feche o Dockerfile.

Crie um requirements.txtno diretório do seu projeto.

Este arquivo é usado pelo RUN pip install -r requirements.txtcomando em seu arquivo Dockerfile.

Adicione o software necessário ao arquivo.

Django>=3.0,<4.0
psycopg2>=2.8
Salve e feche o requirements.txtarquivo.

Crie um arquivo chamado docker-compose.ymlno diretório do seu projeto.

O docker-compose.ymlarquivo descreve os serviços que compõem seu aplicativo. Neste exemplo, esses serviços são um servidor web e um banco de dados. O arquivo de composição também descreve quais imagens Docker esses serviços usam, como elas se vinculam e quaisquer volumes que possam precisar ser montados dentro dos contêineres. Finalmente, o docker-compose.ymlarquivo descreve quais portas esses serviços expõem. Consulte a docker-compose.ymlreferência para obter mais informações sobre como esse arquivo funciona.

Adicione a seguinte configuração ao arquivo.

services:
  db:
    image: postgres
    volumes:
      - ./data/db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    environment:
      - POSTGRES_NAME=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    depends_on:
      - db
Este arquivo define dois serviços: O dbserviço e o webserviço.

Observação:

Isso usa o servidor de desenvolvimento integrado para executar seu aplicativo na porta 8000. Não use isso em um ambiente de produção. Para mais informações, veja a documentação do Django {:target=" blank" rel="noopener" class=" ”}.

Salve e feche o docker-compose.ymlarquivo.

Crie um projeto Django
Nesta etapa, você cria um projeto inicial do Django construindo a imagem a partir do contexto de construção definido no procedimento anterior.

Mude para a raiz do diretório do seu projeto.

Crie o projeto Django executando o comando docker compose run da seguinte maneira.

sudo docker compose run web django-admin startproject composeexample .
Isso instrui o Compose a ser executado django-admin startproject composeexample em um contêiner, usando a webimagem e a configuração do serviço. Como a webimagem ainda não existe, o Compose a cria a partir do diretório atual, conforme especificado pela build: .linha no arquivo docker-compose.yml.

Depois que a webimagem do serviço é criada, o Compose a executa e executa o django-admin startprojectcomando no contêiner. Este comando instrui o Django a criar um conjunto de arquivos e diretórios representando um projeto Django.

Após a docker composeconclusão do comando, liste o conteúdo do seu projeto.

$ ls -l

drwxr-xr-x 2 root   root   composeexample
drwxr-xr-x 3 root   root   data
-rw-rw-r-- 1 user   user   docker-compose.yml
-rw-rw-r-- 1 user   user   Dockerfile
-rwxr-xr-x 1 root   root   manage.py
-rw-rw-r-- 1 user   user   requirements.txt
Se você estiver executando o Docker no Linux, os arquivos django-admincriados serão de propriedade do root. Isso acontece porque o contêiner é executado como usuário root. Altere a propriedade dos novos arquivos.

Não altere a permissão da pasta de dados onde o Postgres contém seu arquivo, caso contrário o Postgres não poderá ser iniciado devido a problemas de permissão.

sudo chown -R $USER:$USER composeexample manage.py
Se você estiver executando o Docker no Mac ou Windows, você já deverá possuir a propriedade de todos os arquivos, incluindo aqueles gerados por django-admin. Liste os arquivos apenas para verificar isso.

$ ls -l

total 32
-rw-r--r--  1 user  staff  145 Feb 13 23:00 Dockerfile
drwxr-xr-x  6 user  staff  204 Feb 13 23:07 composeexample
-rw-r--r--  1 user  staff  159 Feb 13 23:02 docker-compose.yml
-rwxr-xr-x  1 user  staff  257 Feb 13 23:07 manage.py
-rw-r--r--  1 user  staff   16 Feb 13 23:01 requirements.txt
Conecte o banco de dados
Nesta seção, você configura a conexão de banco de dados para Django.

No diretório do seu projeto, edite o composeexample/settings.pyarquivo.

Substitua o DATABASES = ...pelo seguinte:

# settings.py

import os

[...]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_NAME'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': 5432,
    }
}
Essas configurações são determinadas pela imagem do Docker postgres especificada em docker-compose.yml.

Salve e feche o arquivo.

Execute o comando docker compose up no diretório de nível superior do seu projeto.

$ docker compose up

djangosample_db_1 is up-to-date
Creating djangosample_web_1 ...
Creating djangosample_web_1 ... done
Attaching to djangosample_db_1, djangosample_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".

<...>

web_1  | July 30, 2020 - 18:35:38
web_1  | Django version 3.0.8, using settings 'composeexample.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
Neste ponto, seu aplicativo Django deve estar rodando na porta 8000do seu host Docker. No Docker Desktop para Mac e no Docker Desktop para Windows, acesse http://localhost:8000em um navegador da web para ver a página de boas-vindas do Django.

Exemplo de Django

Observação:

Em determinadas plataformas (Windows 10), pode ser necessário editar ALLOWED_HOSTS internamente settings.pye adicionar o nome do host ou endereço IP do Docker à lista. Para fins de demonstração, você pode definir o valor como:

ALLOWED_HOSTS = ['*']
Este valor não é seguro para uso em produção. Consulte a documentação do Django para mais informações.

Listar contêineres em execução.

Em outra janela do terminal, liste os processos do Docker em execução com o comando docker psou docker container ls.

$ docker ps

CONTAINER ID  IMAGE       COMMAND                  CREATED         STATUS        PORTS                    NAMES
def85eff5f51  django_web  "python3 manage.py..."   10 minutes ago  Up 9 minutes  0.0.0.0:8000->8000/tcp   django_web_1
678ce61c79cc  postgres    "docker-entrypoint..."   20 minutes ago  Up 9 minutes  5432/tcp                 django_db_1
Encerre os serviços e limpe usando um destes métodos:

Pare o aplicativo digitando Ctrl-C no mesmo shell onde você o iniciou:

Gracefully stopping... (press Ctrl+C again to force)
Killing test_web_1 ... done
Killing test_db_1 ... done
Ou, para um desligamento mais elegante, mude para um shell diferente e execute docker compose down no nível superior do diretório do projeto de amostra do Django.

$ docker compose down

Stopping django_web_1 ... done
Stopping django_db_1 ... done
Removing django_web_1 ... done
Removing django_web_run_1 ... done
Removing django_db_1 ... done
Removing network django_default
Depois de encerrar o aplicativo, você pode remover com segurança o diretório do projeto Django (por exemplo, rm -rf django).

Mais documentação do Compose
Visão geral do Docker Compose
Instale o Docker Compose
Introdução ao Docker Compose
Referência de linha de comando do Docker Compose
Compor referência de arquivo
Aplicativo de amostra incrível do Compose Django
