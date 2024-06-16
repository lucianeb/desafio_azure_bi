<h1>
    <span> Processamento de Dados Simplificado com Power BI</span>
</h1>

## Tecnologias utilizadas no projeto
- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/microsoftsqlserver.svg" alt="Azure Data Studio" width="40" height="40"/> Azure Data Studio

- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/powerbi.svg" alt="Power BI" width="40" height="40"/> Power BI
  
- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/visualstudio.svg" alt="Visual Studio" width="40" height="40"/> Visual Studio Visual Studio

- <img src="https://github.com/simple-icons/simple-icons/blob/develop/icons/microsoftazure.svg" alt="Azure" width="40" height="40"/> Microsoft Azure

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

- As transformações com relação a mesclagem dos dados realizadas conforma descrito abaixo, utilizar MERGE foi mais adequado para sincronizar os dados entre tabelas e evitar inserções duplicadas em tabelas existentes :
```
-- Criar e mesclar employee com departament para criar uma tabela com nomes dos departamentos associados aos colaboradores
MERGE INTO azure_company.employee_with_dept AS target
USING (
    SELECT e.Ssn, CONCAT(e.Fname, ' ', e.Lname) as FullName, e.Bdate, e.Address, e.Sex, e.Salary, e.Super_ssn, d.Dname as DepartmentName
    FROM azure_company.employee e
    JOIN azure_company.departament d ON e.Dno = d.Dnumber
) AS source
ON target.Ssn = source.Ssn
WHEN MATCHED THEN
    UPDATE SET
        target.FullName = source.FullName,
        target.Bdate = source.Bdate,
        target.Address = source.Address,
        target.Sex = source.Sex,
        target.Salary = source.Salary,
        target.Super_ssn = source.Super_ssn,
        target.DepartmentName = source.DepartmentName
WHEN NOT MATCHED THEN
    INSERT (Ssn, FullName, Bdate, Address, Sex, Salary, Super_ssn, DepartmentName)
    VALUES (source.Ssn, source.FullName, source.Bdate, source.Address, source.Sex, source.Salary, source.Super_ssn, source.DepartmentName);

-- Criar e mesclar employee com nomes dos gerentes
MERGE INTO azure_company.employee_with_manager AS target
USING (
    SELECT e.Ssn, CONCAT(e.Fname, ' ', e.Lname) as FullName, e.Bdate, e.Address, e.Sex, e.Salary, CONCAT(m.Fname, ' ', m.Lname) as ManagerName, d.Dname as DepartmentName
    FROM azure_company.employee e
    LEFT JOIN azure_company.employee m ON e.Super_ssn = m.Ssn
    JOIN azure_company.departament d ON e.Dno = d.Dnumber
) AS source
ON target.Ssn = source.Ssn
WHEN MATCHED THEN
    UPDATE SET
        target.FullName = source.FullName,
        target.Bdate = source.Bdate,
        target.Address = source.Address,
        target.Sex = source.Sex,
        target.Salary = source.Salary,
        target.ManagerName = source.ManagerName,
        target.DepartmentName = source.DepartmentName
WHEN NOT MATCHED THEN
    INSERT (Ssn, FullName, Bdate, Address, Sex, Salary, ManagerName, DepartmentName)
    VALUES (source.Ssn, source.FullName, source.Bdate, source.Address, source.Sex, source.Salary, source.ManagerName, source.DepartmentName);

-- Criar e mesclar nomes de departamentos e localização
MERGE INTO azure_company.department_location AS target
USING (
    SELECT d.Dnumber, CONCAT(d.Dname, ' - ', l.Dlocation) as DepartmentLocation
    FROM azure_company.departament d
    JOIN azure_company.dept_locations l ON d.Dnumber = l.Dnumber
) AS source
ON target.Dnumber = source.Dnumber
WHEN MATCHED THEN
    UPDATE SET
        target.DepartmentLocation = source.DepartmentLocation
WHEN NOT MATCHED THEN
    INSERT (Dnumber, DepartmentLocation)
    VALUES (source.Dnumber, source.DepartmentLocation);

```
A verificação se há departamentos sem gerente foi realizada da seguinte forma :
```
-- Verificação de departamentos sem gerente
-- Adicição de gerente fictício se necessário
UPDATE azure_company.departament
SET Mgr_ssn = '888665555'
WHERE Mgr_ssn IS NULL;
```

Após a transformação dos dados, o database foi importado pelo Power BI (desktop) no modo de compatibilidade de dados Power Query, para a realização de análise e relatório.

<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/conexao1.jpg" alt="Conexão Power BI - PowerQuery" width="550" height="330"/>










