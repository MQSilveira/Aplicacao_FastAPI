
# Dentro da pasta CORE:
    Criar:
    configs.py
        Configurações gerais que iremos utilizar em nosso projeto

    database.py
        Onde pegamos informações do banco de dados

    deps.py
        Onde juntamos todas as dependências que temos em nosso projeto/aplicação

############auth.py

#############security.py


# No arquivo configs.py:
    Importar:
    from typing import List
    from pydantic import BaseSettings
    from sqlalchemy.ext.declarative import declarative_base

    Criar uma classe que irá herdar BaseSettings, que foi importado de pydantic (class Settings(BaseSettings):)

    # Na classe BaseSettings():
    API_V1_STR: str = '/api/v1'
        Variavel contendo a "versão da API"
        Será utilizada como variavel de acesso 
    
    DB_URL: str = 'postgresql+asyncpg://marcos:root@localhost:5435/postgres'
        Informações do nosso BANCO DE DADOS
        usuario = marcos
        senha = root
        servidor = localhost
        porta = 5435
        nome do banco = postgres        (database)
    
    DBBaseModel = declarative_base()
        declarative_base() é a função faz com que nossos módulos herdem e tenham todos os recursos do SQL ALCHEMY nos módulos
    
    JWT_SECRET:str = 'WlcFWRhdK2ggPUUdtl_aE68Wb-fk3ipz1e8B68evw4o'
        Quando trabalhamso com token em APIs o padrão que utilizamos é JWT (JSON Web Token)
        Cada API que criamos, fazemos a utilização de um segredo (senha), a partir dessa senha o HASH será criado.
        NÃO DEVEMOS COMPARTILHAR O SEGREDO COM NINGUÉM, PORQUE SE ALGUEM TIVER ACESSO AO ALGORITMO UTILIZADO E AO SEGREDO, ELE PODE DESCOBRIR O HASH.

        NO TERMINAL:
            import secrets
            token:str = secrets.token_urlsafe(32)
                # = WlcFWRhdK2ggPUUdtl_aE68Wb-fk3ipz1e8B68evw4o
                ESSE TOKEN É UM SECREDO QUE PODEMOS UTILIZAR EM JWT_SECRET

    
    ALGORITHM:str = 'HS256'
        HS256 é referente ao Hash SHA-256 (algoritmo de hash seguro de 256 bits usado para proteção criptográfica)

    ACCESS_TOKEN_EXPIRE_MINUTES:int = 60 * 24 * 7
        Informa o tempo em minutos pra expiração do token (esse token vai ser válido por quanto tempo? 60m x 24h x 7d = 1 semana em minutos)

    class Config:
        case_sesitive = True
        # Essa configuração diz que é importanter "manter"
        # Se for tudo MAIÚSCULO manter 
        # Se for muto MINÚSCULO manter

#CONFIGS.PY
from typing import List
from pydantic import BaseSettings
from sqlalchemy.ext.declarative import declarative_base


class Settings(BaseSettings):
    
    API_V1_STR:str = '/api/v1'
    DB_URL:str = 'postgresql+asyncpg://marcos:root@localhost:5435/postgres'
    DBBaseModel = declarative_base()
    
    JWT_SECRET:str = 'WlcFWRhdK2ggPUUdtl_aE68Wb-fk3ipz1e8B68evw4o'
    """
    import secrets
    token:str = secrets.token_urlsafe(32)
    """
    
    ALGORITHM:str = 'HS256'
    
    ACCESS_TOKEN_EXPIRE_MINUTES:int = 60 * 24 * 7
    
    class Config:
        case_sensitive = True
        
        
settings = Settings()


# No arquivo database.py:
    # CONFIGURAÇÕES IGUAL AS ANTERIORES.

    # Importar:
    from sqlalchemy.orm import sessionmaker
    from sqlalchemy.ext.asyncio import create_async_engine
    from sqlalchemy.ext.asyncio import AsyncEngine
    from sqlalchemy.ext.asyncio import AsyncSession

    from core.configs import settings       # Intância do objeto criado em settings.py

    Criar a engine de conexão:
    engine: AsyncEngine =create_async_engine(settings.DB_URL)
        Estamos passando DB_URL que está em settings.
        DB_URL contém as informações do nosso BANCO DE DADOS

    Cria nossa Session:
    O Session é oque abre e fecha a conexão com o BANCO DE DADOS
    Session: AsyncSession = sessionmaker(
        autocommit=False,
        autoflush=False,
        expire_on_commit=False,
        class_=AsyncSession,
        bind=engine
    )

DATABASE.PY
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.asyncio import create_async_engine
from sqlalchemy.ext.asyncio import AsyncEngine
from sqlalchemy.ext.asyncio import AsyncSession
from core.configs import settings


engine: AsyncEngine =create_async_engine(settings.DB_URL)

Session: AsyncSession = sessionmaker(
    autocommit=False,
    autoflush=False,
    expire_on_commit=False,
    class_=AsyncSession,
    bind=engine
)


# No arquivo security.py:
    # Importar:
    from passlib.context import CryptContext
    
    Criar um contexto
        CRIPTO = CryptContext(schemas=['bcrypt'], deprecated='auto')
        Passando um schemas que é uma lista
        bcrypt schemas mas poderosos pra criação de hashs e senhas em python
        
        deprecated='auto' (descontinuada)
            Se tiver alguma coisa que foi desativado, vai fazer o tratamento automatico

    Criar a função de verificação de senha:
        def verificar_senha(senha:str, hash_senha:str) -> bool:

        ### Função para verificar se a senha está correta, comparando a senha em texto puro, informada pelo usuário, 
        e o hash da senha que estará salvo no banco de dados durante a criação da conta ###

        Recebemos uma senha e hash_senha que são str
        HASH não é a senha criptografada. HASH é como se fosse uma impressão digital da sua senha
        Vamos retornar um BOOLEAN

        return CRIPTO.verify(senha, hash_senha)
            Usando o contexto (CRIPTO) vamos verificar (verify)
            SENHA e o HASH_SENHA que foi criado atravez do contexto

    Criar uma função para gerar o HASH_SENHA:
        def gerar_hash_senha(senha:str) -> str:

        ### Função que gera e retorna o HASH da senha ###

        Recebe a senha do tipo STR e retorn uma STR porque o HASH também é STR

        return CRIPTO.hash(senha)
            Usando o contexto (CRIPTO) vamos retornar o valor de HASH (hash) do objeto (senha)


# No arquivo auth.py:
    # Importar:
    from pytz import timezone
    from typing import Optional, List
    from datetime import datetime, timedelta
    from fastapi.security import OAuth2PasswordBearer
    from sqlalchemy.future import select
    from sqlalchemy.ext.asyncio import AsyncSession
    from jose import jwt
    from models.usuario_model import UsuarioModel
        ## MODELS AINDA NÃO FOI CRIADO ##
    from core.configs import settings
    from core.security import verificar_senha
    from pydantic import EmailStr


    Criar o endpoint p login
        oauth2_schema = OAuth2PasswordBearer(tokenUrl = f'{settings.API_V1_STR}/usuarios/login')
    
        OAuth2PasswordBearer - Permite que criamos um endpoint para autenticação

        {settings.API_V1_STR}/usuarios/login - Token de acesso que vamos precisar para acessar login
    
        ## O arquivo AUTH.PY fica responsável por esse acesso ##

    async def autenticar(email:EmailStr, senha:str, db:AsyncSession) -> Optional[UsuarioModel]:
        Função assincrona para autenticar p email e senha
        
        PYDANTIC possui varios tipos de dados. Email precisa ser do tipo EmailStr

        db não irá precisar do DEPENDS porque não é um ENDPOINT da API. É uma função que vamos utilizar 

        Irá retornar um usuario/objeto do banco de dados (Optional[UsuarioModel])

    async with db as session:
        Bloco de contexto

        query = select(UsuarioModel).filter(UsuarioModel.email == email)
        Buscar o usuario pelo email

        if not usuario:
            return None
            Se não tiver nenhum usuario, retorne none
            Se encontrei , irá para o proximo if

        if not verificar_senha(senha, usuario.senha):
            return None
            Vou verificar a senha que está vindo como parametro (senha) e vou comparar com a senha do usuario (usuario.senha) que foi encontrado no banco de dados (verificar_senha será o hash da senha)
se não retornar True, eu retorno None 
        
        return usuario
            Se passar eu retorno o usuario que foi autenticado no sistema 

    Função para criar o TOKEN:
    Essa função é interna, ela será utilizada por outra função nossa.
        def _criar_token(tipo_token:str, tempo_vida:timedelta, sub:str) => str:
            tipo_token:str
                Tipo do TOKEN (str)

            tempo_vida:timedelta
                Tempo de vida do TOKEN (timedelta)

            sub:str
                Objeto para quem vou estar criando o TOKEN (usuário)

        DADOS QUE IRÃO COMPORT O PAYLOAD:
            sp = timezone('America/Sao_Paulo')
                timezone
            
            expira = datetime.now(tz=sp) + tempo_vida
                Tempo que irá expirar (timezone que criamos + tempo de vida que a função irá receber - esse tempo será de 1 semana)

            payload['type'] = tipo_token
                payload no índice 'type' irá receber o tipo de TOKEN informado como parametro da FUNÇÃO

            payload['exp'] = expira
                 payload no índice 'exp' (expiração) irá receber a variável EXPIRA

            payload['iat'] = datetime.now(tz=sp)
                payload no índice 'iat' (issued at - gerado em) irá receber a timezone ("quando ele foi gerado")
            
            payload['sub'] = str(sub)
                payload no índice 'sub' (subject) irá receber a STR do parametro informado na funlção (pode ser o id do usuário, email, nome, será convertido p str e inserido em payload['sub'])

        return jwt.encode(payload, settings.JWT_SECRET, algorithm=settings.ALGORITHM)
            Vamos retornar jwt.encode para codificar
    
    Função para criar o TOKEN:
    Função que realmente irá criar o token.
        def criar_token_acesso(sub:str) -> str:

            return _criar_token(
        tipo_token='access_token',
        tempo_vida=timedelta(munutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    )

            irá retornar a função _criar_token

            tipo_token='access_token',
                É um token de acasso


        tempo_vida=timedelta(munutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
            Objeto do tipo timedelta informando tudo que foi configurado no settings

            JSON Web Tokens 
            Utilizado para verificar os TOKENS
            https://jwt.io/


# No arquivo deps.py:
    # Importar:
    from typing import Generator, Optional
    from fastapi import Depends, HTTPException, status
    from jose import jwt, JWTError
    from sqlalchemy.ext.asyncio import AsyncSession
    from sqlalchemy.future import select
    from pydantic import BaseModel
    from core.database import Session
    from core.auth import oauth2_schema
    from core.configs import settings
    from models.usuario_model import UsuaroioModel

    Criar classe Base baseada no BaseModel do Pydantic
        class TokenData(BaseModel): 
            username: Optional[str] = None

    Função assincrona para gerar a sessão
        async def get_session() -> Generator:
            session : AsyncSession = Session()
            
            try:
                yield session
            
            finally:
                await session.close()

    Usuário cria a conta, será necessário criar um TOKEN ao fazer o login,
    com o TOKEN de acesso ele poderá afzer algumas coisas na API

    Com o TOKEN de acesso, sabemos que ele está autorizado,
    mas não sabemos quem é.

    Criar uma função que retorne o usuário através do TOKEN:
        async def get_current_user(
            db:Session = Depends(get_session),
            token:str = Depends(oauth2_schema)
            ) -> UsuaroioModel:
        
        get_current_user = Pegar usuário atual
        
        token:str = Depends(oauth2_schema)
            Precisamos de um login

        Vamos retornar um UsuaroioModel
    
    Criar uma exeção:
        credencial_exception:HTTPException = HTTPException(
            status_code = status.HTTP_401_UNAUTHORIZED,
            detail = 'Não foi possível autenticar a credencial',
            headers = {'www-Authenticate':'Bearer'},)

        
        credencial_exception:HTTPException = HTTPException(INSTÂNCIAR UM OBJETO AQUI)
                Do tipo HTTP 

        status_code = status.HTTP_401_UNAUTHORIZED
                Status não autorizado

        detail = 'Não foi possível autenticar a credencial'

        headers = {'www-Authenticate':'Bearer'}


        ## É COMO UMA VÁRIÁVEL. CASO O USUÁRIO NÃO AUTENTIQUE, IRÁ RETORNAR ESSA EXEÇÃO

    Exceção:
        try:
            Vamos TENTAR decodificar(decode) o payload
            payload = jwt.decode(
                token,                                  TOKEN     
                settings.JWT_SECRET,                    PELO JWT_SECRET QUE VAMOS CONSEGUIR VALIDAR ESSAS INFORMAÇÕES
                alogorithms=[settings.ALGORITHM],       ESPERA UMA LISTA
                options={'verify_aud':False}            NÃO IREMOS UTILIZAR
            )
            username:str = payload.get('sub')
                        Depois de decodificar, vamos fazer acesso através do payload.get passando qual é a chave 
                        'sub' está dentro de payload
                        será nosso username
            
            if username is None:
                raise credencial_exception
                        SE USERNAME FOR NONE, RETORNO A VÁRIÁVEL CRIADA PARA AUTENTICAÇÃO

            token_data:TokenData = TokenData(username=username)

        except JWTError:
            raise credencial_exception
                    SE TIVER UM ERRO DO TIPO JWTError, RETORNO NOVAMENTE A VÁRIÁVEL CRIADA PARA AUTENTICAÇÃO
    

    Tentar pegar o usuário no sistema:
    async with db as session:
        query = select(UsuaroioModel.filter(UsuaroioModel.id == int(token_data.username)))
                    Nossa query será UsuaroioModel onde o id seja igual ao int de token_data.username

        
    if usuario is None:
        raise credencial_exception
    return usuario
                    Se o usuário não for encontrado, retornamos uma exceção, caso contrário, retornamos o usuário
