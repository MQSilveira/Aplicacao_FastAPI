
# criar_tabelas.py:
    Importar:
    from core.configs import settings
    from core.database import engine

    Criar uma função assincrona para a criação das tabelas:
    async def create_table() -> None:

    Fazer a importação do models para a criação das tabelas:
    import models.__all_models

    Criar conexão com banco e criar as tabelas:
    async with engine.begin() as conn:
        await conn.run_sync(settings.DBBaseModel.metadata.drop_all)     # APAGA TUDO 
        await conn.run_sync(settings.DBBaseModel.metadata.create_all)   # CRIA


    Criar o bloco de execução:
    if __name__ == '__main__':
        Se name for igual a main, importa asyncio 

    asyncio.run(create_tables())
        Executa a função create_tables


