
# No arquivo main.py:
    Importar:
    from fastapi import FastAPI
    from core.configs import settings
    from api.v1.api import api_router

    app = FastAPI(title='Curso Fast API')
            Cria uma nova instância da classe FastAPI com o título "Curso Fast API".

    app.include_router(api_router, prefix=settings.API_V1_STR)
            Adiciona o ENDPOINT principal da API à instância do objeto FastAPI, 
            definindo o prefixo das rotas da API de acordo com as configurações definidas no arquivo "configs.py"
    
    
    if __name__ == '__main__':
        import uvicorn
        uvicorn.run('main:app',
                    host='0.0.0.0',
                    port=8000,
                    log_level='into',
                    reload=True)





