
# Dentro da pasta MODELS:
    Criar:
    __all_models.py
    artigo_model.py
    usuario_model.py    


    artigo_models.py
        Importar:

        from sqlalchemy import Column, Integer, String, ForeignKey
        from sqlalchemy.orm import relationship
        from core.configs import settings


        Criar a classe MODEL:
        class ArtigoModel(settings.DBBaseModel):
            
            __tablename__ = 'artigos'

            id = Column(Integer, primary_key=True, autoincrement=True)
            titulo = Column(String(256))
            descricao = Column(String(256))
            url_fonte = Column(String(256))
            usuario_id = Column(Integer, ForeignKey('usuarios.id'))
            criador = relationship(
                'UsuarioModel', back_populates='artigos', lazy='joined')
            


    usuario_model.py
        Importar:

        from sqlalchemy import Integer, String, Column, Boolean
        from sqlalchemy.orm import relationship
        from core.configs import settings


        Criar a classe MODEL:
        class UsuarioModel(settings.DBBaseModel):
            
            __tablename = 'usuarios'
            
            id = Column(Integer, primary_key=True, autoincrement=True)
            nome = Column(String(256), nullable=True)
            sobrenome = Column(String(256), nullable=True)
            email = Column(String(256), index=True, nullable=False, unique=True)
            senha = Column(String(256), nullable=False)
            eh_admin = Column(Boolean, default=False)
            artigos = relationship(
                'ArtigoModel',
                cascade='all,delete-orphan',
                back_populates='criador',           Referente a 'criador'
                uselist=True,                       Quando buscar, quermos uma lista
                lazy='joined')                      Utilizado para melhor performace
                        Se o usuário for removido e tiver uma lista da de artigos, irá remover todos 



    __all_models.py
        Importar:

        from models.artigo_model import ArtigoModel
        from models.usuario_model import UsuarioModel


        precisamos criar os schemas do artigo_model e usuario_model
        não faz a junção de schemas automáticos

