
# Dentro da pasta MODELS:
    Criar:
    usuario_schema.py

    artigo_schema.py


    artigo_models.py
        Importar:

        from typing import Optional
        from pydantic import BaseModel, HttpUrl


        class ArtigosSchema(BaseModel):
            
            id:Optional[int] = None
            titulo:str
            descricao:str
            url_fonte:HttpUrl
            usuario_id:Optional[int]
            
            class Config:
                
                orm_mode = True


    
    usuario_schema.py
        Importar:
        from typing import Optional
        from typing import List
        from pydantic import BaseModel, EmailStr
        from schemas.artigo_schema import ArtigosSchema


        Criar os SCHEMAS:
        # Schema contendo os dados básicos do usuário:
        # Será utilizado para filtrar os Usuarios. 
        class UsuarioSchemaBase(BaseModel):
    
            id:Optional[int] = int
            nome:str
            sobrenome:str
            email:EmailStr
            eh_admin:bool = False
            
            class Config:
                orm_mode = True
                        UTILIZADO PORQUE ELE É BASEADO EM BANCO DE DADOS


        # Schemas alternativos pra quando precisarmos desses outros dados:
        class UsuarioSchemaCreate(UsuarioSchemaBase):
            senha: str
                    # Será utilizado para a criação de usuarios
                    # UsuarioSchemaCreate está herdando os atributos de UsuarioSchema
                


        class UsuarioSchemaArtigos(UsuarioSchemaBase):
            artigos:Optional[List[ArtigosSchema]]
                    # Irá conter os parametros de UsuarioSchemaBase + os artigos


        class UsuarioSchemaUp(UsuarioSchemaBase):
            nome:Optional[str]
            sobrenome:Optional[str]
            email:Optional[EmailStr]
            senha:Optional[str]
            eh_admin:Optional[bool]
                    # Será utilizado para atualizar as informações 
                    # os parametros são opcionais (Optional) porque eu posso alterar um parametro expecifico, e não alterar os demais.

