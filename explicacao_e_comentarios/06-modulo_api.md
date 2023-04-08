

# Dentro da pasta API:
    Criar o diretório:
        v1 - Versão de nossa API

        Dentro de v1 criar:
            api.py
            endpoints - Irá conter nossos endpoints


    Dentro de endpoints:
    Criar os arquivos:
        artigo.py
        usuario.py
            Cada um irá conter o endpoint do seu respectivo model
    

    # No arquivo artigo.py:
    Importar:
    from typing import List
    from fastapi import APIRouter
    from fastapi import status
    from fastapi import Depends
    from fastapi import HTTPException
    from fastapi import Response
    from sqlalchemy.ext.asyncio import AsyncSession
    from sqlalchemy.future import select
    from models.artigo_model import ArtigoModel
    from models.usuario_model import UsuarioModel
    from schemas.artigo_schema import ArtigosSchema
    from core.deps import get_session, get_current_user
        # get_current_user
            Um usuário não irá poder postar um artigo, se ele não existir
            Irá passar um TOKEN de autenticação, com esse TOKEN vamos recuperar os dados do usuário

    Criar o router:
    router = APIRouter()

    # Criar ass rotas / métodos:

    Criar o método POST:
        @router.post('/', status_code=status.HTTP_201_CREATED, response_model=ArtigosSchema)
            status_code define = Retorna o código de status HTTP 201 indicando que um novo recurso foi criado.
                
            response_model = Esquema de resposta JSON que será retornado pelo endpoint. Irá retornar um ArtigosSchema.
                
        async def post_artigo(artigo:ArtigosSchema, usuario_logado:UsuarioModel, Depends(get_current_user), db:AsyncSession = Depends(get_session)):
            artigo =  Será um objeto da classe ArtigosSchema
            
            usuario_logado = Será um objeto da classe UsuarioModel, que é injetado automaticamente pelo FastAPI a partir dos dados do token de autenticação do usuário que fez a solicitação. 
                            O token de autenticação é validado pelo middleware get_current_user antes que a função seja executada.

            db = é utilizado para executar operações de banco de dados assíncronas.
            
        Criar a instancia do objeto:
            novo_artigo:ArtigoModel = ArtigoModel(titulo = artigo.titulo, descricao = artigo.descricao, url_fonte = artigo.url_fonte, usuario_id = usuario_logado.id)
        
        db.add(novo_artigo)
            Armazena novo_artigo na memória

        await db.commit()
            Faz o commit no banco de dados. AWAIT indica que esta operação é assíncrona.
        
        return novo_artigo
            Retorna um JSON gerado automaticamente pelo FastAPI a partir do objeto novo_artigo.
            Apesar de estar instanciando um objeto de novo_artigo o retorno será um objeto de ArtigosSchema


    Criar o método GET artigos:
        @router.get('/', response_model = List[ArtigosSchema])
        async def get_artigos(db:AsyncSession = Depends[get_session]):
            
            async with db as session:
                
                query = select(ArtigoModel)
                result = await session.execute(query)
                artigos:List[ArtigoModel] = result.scalars().unique().all()
                
                return artigos


    
    Criar o método GET artigo:
    @router.get('/'{artigo_id}, response_model=ArtigosSchema, status_code = status.HTTP_200_OK)
    async def get_artigo(artigo_id:int, db:AsyncSession = Depends[get_session]):
        
        async with db as session:
            
            query = select(ArtigoModel).filter(ArtigoModel.id == artigo_id)
            result = await session.execute(query)
            artigo:ArtigoModel = result.scalars().unique().one_or_none()
            
            
            if artigo:
                return artigo
            
            else: 
                
                raise HTTPException(detail='Artigo não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)



    Criar PUT de artigo:
    @router.put('/'{artigo_id}, response_model=ArtigosSchema, status_code = status.HTTP_202_ACCEPTED)
    async def put_artigo(artigo_id:int, artigo:ArtigosSchema, db:AsyncSession = Depends[get_session], usuario_logado:UsuarioModel = Depends(get_current_user)):
        
        async with db as session:
            
            query = select(ArtigoModel).filter(ArtigoModel.id == artigo_id)
            result = await session.execute(query)
            artigo_up:ArtigoModel = result.scalars().unique().one_or_none()
            
            
            if artigo_up:
                if artigo.titulo:
                    artigo_up.titulo = artigo.titulo
                    
                if artigo.descricao:
                    artigo_up.descricao = artigo.descricao
                    
                if artigo.url_fonte:
                    artigo_up.url_fonte = artigo.url_fonte
                    
                if usuario_logado.id != artigo_up.usuario.id:
                    artigo_up.usuario_id = usuario_logado.id
                
                await session.commit()
                
                return artigo_up
            
            
            else: 
                
                raise HTTPException(detail='Artigo não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)



    Criar DELETE de artigo:
    @router.delete('/{artigo_id}', status_code = status.HTTP_204_NO_CONTENT)
    async def delete_artigo(artigo_id:int, db:AsyncSession = Depends[get_session], usuario_logado:UsuarioModel = Depends(get_current_user)):
        
        async with db as session:
            
            QUERY COM 2 FILTROS
            query = select(ArtigoModel).filter(
                ArtigoModel.id == artigo_id).filter(
                    ArtigoModel.usuario_id == usuario_logado.id)
            result = await session.execute(query)
            artigo_del:ArtigoModel = result.scalars().unique().one_or_none()
            
            
            if artigo_del:
                    
                await session.delete(artigo_del)
                await session.commit()
                
                return Response(status.HTTP_204_NO_CONTENT)
            
            
            else: 
                
                raise HTTPException(detail='Artigo não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)



    # No arquivo usuario.py:
        Importar:
        from typing import List, Optional, Any
        from fastapi import APIRouter
        from fastapi import status
        from fastapi import Depends
        from fastapi import HTTPException
        from fastapi import Response
        from fastapi.security import OAuth2PasswordRequestForm
        from fastapi.responses import JSONResponse
        from sqlalchemy.ext.asyncio import AsyncSession
        from sqlalchemy.future import select
        from models.usuario_model import UsuarioModel
        from schemas.usuario_schema import UsuarioSchemaBase
        from schemas.usuario_schema import UsuarioSchemaCreate
        from schemas.usuario_schema import UsuarioSchemaArtigos
        from schemas.usuario_schema import UsuarioSchemaUp
        from core.deps import get_session
        from core.deps import get_current_user
        from core.security import gerar_hash_senha
        from core.auth import autenticar
        from core.auth import criar_token_acesso

                from fastapi.security import OAuth2PasswordRequestForm
                    fastAPI vai identificar que determinados endpoints precisam de autenticação e
                    teremos um formulario junto para autenticar com o email e a senha


    Criar o router:
    router = APIRouter()

    # Criar ass rotas / métodos:

    Criar o método GET:
    @router.get('/logado', response_model = UsuarioSchemaBase)
    def get_logado(usuario_logado:UsuarioModel = Depends(get_current_user)):
        
        return usuario_logado
                # Vamos utiliza get_current_user para retornar os dados do Usuário. 
                # get_current_user faz a consulta ao banco de dados utilizando o TOKEN de autenticação

    
    Criar o método POST ou SIGNUP:
    @router.post('/signup', status_code = status.HTTP_201_CREATED, response_model = UsuarioSchemaBase)
    async def post_usuario( usuario:UsuarioSchemaCreate, db:AsyncSession = Depends(get_session)):
                response_model = UsuarioSchemaBase - Estamos retornando UsuarioSchemaBase
                usuario:UsuarioSchemaCreate - Estamos criando um UsuarioSchemaCreate.
                    UsuarioSchemaCreate herda os atributos de UsuarioSchemaBase
                    UsuarioSchemaCreate possui os atributos de UsuarioSchemaBase + a senha
                    Retornamos UsuarioSchemaBase porque não podemos exibir a senha do Usuário.

         Criar a instancia do objeto:
            novo_usuario:UsuarioModel = UsuarioModel(
                nome = usuario.nome,
                sobrenome = usuario.sobrenome,
                email = usuario.email,
                senha = gerar_hash_senha(usuario.senha),
                eh_admin = usuario.eh_admin)


            async with db as session:
                session.add(novo_usuario)
                await session.commit()
                
                return novo_usuario

    
    Criar o método GET usuarios:
    # GET usuarios:
    @router.get('/', response_model = List[UsuarioSchemaBase])
    async def get_usuarios(db:AsyncSession = Depends(get_session)):
        
        async with db as session:
            
            query = select(UsuarioModel)
            result = await session.execute(query)
            usuarios:List[UsuarioSchemaBase] = result.scalars().unique().all()
            
            return usuarios


    Criar o método GET usuario:
    @router.get('/{id_usuario}', response_model=UsuarioSchemaArtigos, status_code=status.HTTP_200_OK)
    async def get_usuario(usuario_id:int, db:AsyncSession=Depends[get_session]):
        
        async with db as session:
            
            query = select(UsuarioModel).filter(UsuarioModel.id == usuario_id)
            result = await session.execute(query)
            usuario:UsuarioSchemaArtigos = result.scalars().unique().one_or_none()
            
            
            if usuario:
                return usuario
            
            else:
                raise HTTPException(detail='Usuário não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)


    Criar o método PUT usuario:
    @router.put('/{id_usuario}', response_model=UsuarioSchemaBase, status_code=status.HTTP_202_ACCEPTED)
    async def put_usuario(usuario_id:int, usuario:UsuarioSchemaUp, db:AsyncSession=Depends[get_session]):
        
        async with db as session:
            
            query = select(UsuarioModel).filter(UsuarioModel.id == usuario_id)
            result = await session.execute(query)
            usuario_up:UsuarioSchemaBase = result.scalars().unique().one_or_none()
            
            if usuario_up:
                
                if usuario.nome:
                    usuario_up.nome = usuario.nome
                    
                if usuario.sobrenome:
                    usuario_up.sobrenome = usuario.sobrenome
                
                if usuario.email:
                    usuario_up.email = usuario.email
                
                if usuario.eh_admin:
                    usuario_up.eh_admin = usuario.eh_admin
                    
                if usuario.senha:
                    usuario_up = gerar_hash_senha(usuario.senha)
                
                await session.commit()
                
                return usuario_up
            
            else:
                
                raise HTTPException(detail='Usuário não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)

    Criar método DELETE usuario:
    # DELETE usuario:
    @router.delete('/{id_usuario}', status_code=status.HTTP_204_NO_CONTENT)
    async def delete_usuario(usuario_id:int, db:AsyncSession=Depends[get_session]):
        
        async with db as session:
            
            query = select(UsuarioModel).filter(UsuarioModel.id == usuario_id)
            result = await session.execute(query)
            usuario_del:UsuarioSchemaArtigos = result.scalars().unique().one_or_none()
            
            
            if usuario_del:
                
                await session.delete(usuario_del)
                await session.commit()
                
                return Response(status.HTTP_204_NO_CONTENT)
            
            else:
                raise HTTPException(detail='Usuário não encontrado',
                                    status_code=status.HTTP_404_NOT_FOUND)  


    Criar método POST login:
    @router.post('/login')
    async def login(form_data:OAuth2PasswordRequestForm = Depends(), db:AsyncSession = Depends(get_session)):
        Utilizamos um form_data do tipo OAuth2PasswordRequestForm que é uma dependencia (Depends)
        porque o login é feito com usuário e senha em um formulário
        form_data possui parametros específicos (username e password), que são doferentes de email, senha, usuario...
        POR ISSO QUE INFORMAMOS email=form_data.username, senha=form_data.password

        usuario = await autenticar(email=form_data.username, senha=form_data.password, db=db)
            Função autenticar recebe 3 parametros, email, senha e banco, para validar o usuário


        if usuario:
            
            return JSONResponse(content={'access_token':criar_token_acesso(sub=usuario.id), 'token_type':'bearer'}, status_code=status.HTTP_200_OK)
                    access_token é o tipo de token que estamos criando 
                    A função criar_token_acesso recebe como parametro o subject (sub) que será usuario.id (usupario encontrado)
                    vamos criar o token de acesso, baseado no ID dele
                    'token_type':'bearer' é o tipo de token específico que estamos utilizando. Precisamos informar para a autenticação 
 

        else:        
            raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail='Dados Incorretos!')
                    Caso não encontre o usuário


    
    Montar a API

    # No arquivo api.py:
        Importar:
        from fastapi import APIRouter
        from api.v1.endpoints import artigo
        from api.v1.endpoints import usuario

    api_router = APIRouter()
        Cria a instância da classe APIRouter que será usada para agrupar todos os endpoints da aplicação.
    
    api_router.include_router(artigo.router, prefix='/artigos', tags=['artigos'])
    api_router.include_router(usuario.router, prefix='/usuarios', tags=['usuarios'])
        Adiciona o ENDPOINT definido no módulo artigo/usuario à instância do roteador principal (api_router),
        definindo o prefixo '/artigos' e 'usuarios' para o caminho do endpoint e a tag 'artigos' e 'usuarios' para documentação da API.

    



