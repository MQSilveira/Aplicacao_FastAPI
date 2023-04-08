
# Cria ambiente virtual
    python -m venv venv

# Ativa ambiente virtual
    venv\Scripts\activate

# Instalar FastAPI
    pip install fastapi

# Instalar uvicorn
    pip install uvicorn

# Instalar psycopg2-binary: Pra trabalhar com Postgresql
    pip install psycopg2-binary

# Instalar sqlalchemy 
    pip install sqlalchemy

# Instalar asyncpg: Pra trabalhar com banco de dados assíncrono
    pip install asyncpg

# Instalar python-jose[cryptography]:
    pip install python-jose[cryptography]

# Instalar passlib:
    pip install passlib

# Instalar python-multipart:
    pip install python-multipart

# Instalar pytz:
    pip install pytz

# Criar o requirements.txt
    pip freeze > requirements.txt


# Criar os diretórios:
    api : Onde vamos trabalhar diretamente na nossa API

    core : Nosso "núcleo" do projeto/sistema. Irá conter arquivos de utilização comum para a API

    models : Irá conter os models do projeto

    schemas : "Pega um objeto SQLALCHEMY, BANCO DE DADOS, PYTHON, e transforma em JSON para que nossa API possa trabalhar" (faz o serviço reverso também). Esse processo é chamado de serialização e desserialização. Esse processo é feito com o PYDANTIC.

    criar_tabelas.py

    main.py








