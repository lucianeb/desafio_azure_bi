<h1>
    <span> Projeto de EBOOK do Curso de Fundamentos de IA para DEVs da DIO</span>
</h1>

## 📒 Descrição
Descrição do desafio módulo 3 – Processamento de Dados Simplificado com Power BI

1. Criação de uma instância na Azure para MySQL

2. Criar o Banco de dados com base disponível no github

3. Integração do Power BI com MySQL no Azure

4. Verificar problemas na base a fim de realizar a transformação dos dados

Inicialmente foram configuradas as regras do firewall para permitir o acesso publico ao database. Foi criado um servidor SQL no Azure denominado powerbiserverclient, seguido pela configuração do banco de dados desafio_dio utilizando o script script_bd_company.sql que definiu o database schema e constrains. Posteriormente, o banco de dados foi populado com dados utilizando o script populate_bd_company.sql, que inseriu registros nas tabelas conforme o modelo especificado. As etapas incluíram a definição de chaves primary e foregin, a verificação de integridade referencial, e a inserção de dados de exemplo para simular um ambiente de dados corporativos, permitindo futuras análises e visualizações no Power BI.
<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/azure_db.jpg" alt="Criacao da instancia Azure" width="1200" height="200"/>

- a criação do database foi realizada com o comando sqlcmd -S tcp:powerbiserverclient.database.windows.net,1433 -d desafio_dio -U powerbi -P -i script_bd_company.sql no power shell
- a população foi realizada com o comando sqlcmd -S tcp:powerbiserverclient.database.windows.net,1433 -d desafio_dio -U powerbi -P -i populate_bd_company.sql no power shell  

Os scripts apresentaram alguns problemas que foram selecionados entre eles o nome da coluna Mgr_end_date, sintaxe SQL e comandos não suportados no Azure SQL como SHOW TABLES e DESC.  

- As correções foram realizadas e os dados foram importados para o Azure Data Studio para transformação e manipulação, foi criado uma tabela employee com o nome dos departamentos associados aos colaboradores assim como dos colaboradores com seus respectivos gerentes.

<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/azure_studio_tables.jpg" alt="Criacao da instancia Azure" width="1000" height="400"/>

Após a transformação dos dados, o database foi importado pelo Power BI (desktop) para a realização de análise e relatório.



## Tecnologias utilizadas no projeto
- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/microsoftsqlserver.svg" alt="Azure Data Studio" width="40" height="40"/> Azure Data Studio

- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/powerbi.svg" alt="Power BI" width="40" height="40"/> Power BI
  
- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/visualstudio.svg" alt="Visual Studio" width="40" height="40"/> Visual Studio Visual Studio

- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/microsoftazure.svg" alt="Azure" width="40" height="40"/> Microsoft Azure









