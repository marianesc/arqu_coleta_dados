# Pipeline de arquitetura e coleta de dados

## Informações do autor
* Discente: Mariane Silva de Carvalho
* Curso: MBA em Engenharia e Análise de Dados
* Instituição: Centro Universitário de João Pessoa (UNIPÊ)

** Guia de execução
Requisitos:
* Python 3.11
* Docker
* Setar variáveis de ambiente do Minio e postgres no .env

```
# Criação do ambiente virtual
py -3.11 -m venv .venv
venv\Scripts\activate.bat

# Instalação das dependencias
pip install -r requirements.txt

# Criação do conteiner docker
cd docker
docker compose up -d
```

## Parte 3 · Análise

* Qual regra da Silver mandou mais registros para a quarentena e por quê?

A itens_produto_inexistente, porque eram informações que tinham como produto um produto que foi removido por ter inconsistências, como um valor negativo, logo esse itenfs ficaram com um produto inexistente, indo para quarentena
* Se a reconciliação não fechasse, o que você investigaria primeiro?

A reconciliação não fechou na Gold. O gate pedidos Gold = pedidos válidos Silver falhou, com 22.473 pedidos válidos na Silver contra 22.286 em vendas_diarias.

Solução adotada:

1. Ver qual número não fechava. A receita batia, mas a contagem de pedidos não.
2. Pegar a diferença de IDs entre os dois lados. Eram 187 pedidos.
3. Procurar esses IDs na quarentena. Eram pedidos com status válido, mas com todos os itens rejeitados (itens dos produtos de preço negativo). Sem item válido, não há receita, e o pedido não aparece em vendas_diarias.
4. Corrigir a regra. A reconciliação passou a contar só os pedidos válidos que têm pelo menos um item válido.

* Que dado pessoal existe na sua arquitetura e como ele foi tratado (mascarado, removido, restrito)?

Dados dos clientes: nome completo, CPF, e-mail e telefone.

CPF e e-mail foram anonimizados ou transformados em hash para evitar exposição. As camadas bronze e silvere tem acesso restrito. Na camada Gold há apenas métricas consolidadas, sem dados pessoais.

* O que você mudaria nesta arquitetura se o volume de dados fosse 1.000 vezes maior?

Usaria o processamento distribuido em nuvem com o objetivo de ter uma escalabilidade horizontal; Aplicaria o particionamento por datas no iceberg; Substituiria o Postgres por um motor analíticos colunar para melhoria de desempenho.