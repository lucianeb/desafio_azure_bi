<h1>
    <span> Processamento de Dados Simplificado com Power BI</span>
</h1>

## 👩‍💻 Tecnologias utilizadas no projeto
- <img src="https://github.com/lucianeb/icons/blob/main/azure_data_studio.jpg" alt="Azure Data Studio" width="40" height="40"/> Azure Data Studio

- <img src="https://github.com/microsoft/PowerBI-Icons/blob/main/SVG/Power-BI.svg" alt="Power BI" width="40" height="40"/> Power BI

- <img src="https://github.com/lucianeb/icons/blob/main/visualst.png" alt="Visual Studio" width="40" height="40"/> Visual Studio

- <img src="https://github.com/lucianeb/icons/blob/main/azure.png" alt="Microsoft Azure" width="40" height="40"/> Microsoft Azure

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
O script corrigido ficou assim :
```
-- Criar esquema se não existir
IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'azure_company')
BEGIN
    EXEC sp_executesql N'CREATE SCHEMA azure_company';
END;

-- Remover tabelas existentes, se existirem
IF OBJECT_ID('azure_company.employee', 'U') IS NOT NULL
    DROP TABLE azure_company.employee;

IF OBJECT_ID('azure_company.departament', 'U') IS NOT NULL
    DROP TABLE azure_company.departament;

IF OBJECT_ID('azure_company.dept_locations', 'U') IS NOT NULL
    DROP TABLE azure_company.dept_locations;

IF OBJECT_ID('azure_company.project', 'U') IS NOT NULL
    DROP TABLE azure_company.project;

IF OBJECT_ID('azure_company.works_on', 'U') IS NOT NULL
    DROP TABLE azure_company.works_on;

IF OBJECT_ID('azure_company.dependent', 'U') IS NOT NULL
    DROP TABLE azure_company.dependent;

-- Criar tabela employee no esquema azure_company
CREATE TABLE azure_company.employee (
    Fname varchar(15) NOT NULL,
    Minit char,
    Lname varchar(15) NOT NULL,
    Ssn char(9) NOT NULL, 
    Bdate date,
    Address varchar(30),
    Sex char,
    Salary decimal(10,2),
    Super_ssn char(9),
    Dno int NOT NULL,
    CONSTRAINT chk_salary_employee CHECK (Salary > 2000.0),
    CONSTRAINT pk_employee PRIMARY KEY (Ssn)
);

ALTER TABLE azure_company.employee 
    ADD CONSTRAINT fk_employee 
    FOREIGN KEY (Super_ssn) REFERENCES azure_company.employee (Ssn)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION;

CREATE TABLE azure_company.departament (
    Dname varchar(15) NOT NULL,
    Dnumber int NOT NULL,
    Mgr_ssn char(9) NOT NULL,
    Mgr_start_date date, 
    Dept_create_date date,
    CONSTRAINT chk_date_dept CHECK (Dept_create_date < Mgr_start_date),
    CONSTRAINT pk_dept PRIMARY KEY (Dnumber),
    CONSTRAINT unique_name_dept UNIQUE (Dname),
    FOREIGN KEY (Mgr_ssn) REFERENCES azure_company.employee (Ssn)
);

ALTER TABLE azure_company.departament 
    DROP CONSTRAINT IF EXISTS departament_ibfk_1;

ALTER TABLE azure_company.departament 
    ADD CONSTRAINT fk_dept FOREIGN KEY (Mgr_ssn) REFERENCES azure_company.employee (Ssn)
    ON UPDATE CASCADE;

CREATE TABLE azure_company.dept_locations (
    Dnumber int NOT NULL,
    Dlocation varchar(15) NOT NULL,
    CONSTRAINT pk_dept_locations PRIMARY KEY (Dnumber, Dlocation),
    CONSTRAINT fk_dept_locations FOREIGN KEY (Dnumber) REFERENCES azure_company.departament (Dnumber)
);

ALTER TABLE azure_company.dept_locations 
    DROP CONSTRAINT IF EXISTS fk_dept_locations;

ALTER TABLE azure_company.dept_locations 
    ADD CONSTRAINT fk_dept_locations FOREIGN KEY (Dnumber) REFERENCES azure_company.departament (Dnumber)
    ON DELETE CASCADE
    ON UPDATE CASCADE;

CREATE TABLE azure_company.project (
    Pname varchar(15) NOT NULL,
    Pnumber int NOT NULL,
    Plocation varchar(15),
    Dnum int NOT NULL,
    PRIMARY KEY (Pnumber),
    CONSTRAINT unique_project UNIQUE (Pname),
    CONSTRAINT fk_project FOREIGN KEY (Dnum) REFERENCES azure_company.departament (Dnumber)
);

CREATE TABLE azure_company.works_on (
    Essn char(9) NOT NULL,
    Pno int NOT NULL,
    Hours decimal(3,1) NOT NULL,
    PRIMARY KEY (Essn, Pno),
    CONSTRAINT fk_employee_works_on FOREIGN KEY (Essn) REFERENCES azure_company.employee (Ssn),
    CONSTRAINT fk_project_works_on FOREIGN KEY (Pno) REFERENCES azure_company.project (Pnumber)
);

DROP TABLE IF EXISTS azure_company.dependent;

CREATE TABLE azure_company.dependent (
    Essn char(9) NOT NULL,
    Dependent_name varchar(15) NOT NULL,
    Sex char,
    Bdate date,
    Relationship varchar(8),
    PRIMARY KEY (Essn, Dependent_name),
    CONSTRAINT fk_dependent FOREIGN KEY (Essn) REFERENCES azure_company.employee (Ssn)
);

SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'azure_company';

EXEC sp_columns @table_name = 'employee', @table_owner = 'azure_company';
EXEC sp_columns @table_name = 'departament', @table_owner = 'azure_company';
EXEC sp_columns @table_name = 'dept_locations', @table_owner = 'azure_company';
EXEC sp_columns @table_name = 'project', @table_owner = 'azure_company';
EXEC sp_columns @table_name = 'works_on', @table_owner = 'azure_company';
EXEC sp_columns @table_name = 'dependent', @table_owner = 'azure_company';
```

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
- A verificação se há departamentos sem gerente foi realizada da seguinte forma :
```
-- Verificação de departamentos sem gerente
-- Adicição de gerente fictício se necessário
UPDATE azure_company.departament
SET Mgr_ssn = '888665555'
WHERE Mgr_ssn IS NULL;
```
- O único colaborador sem gerente é James Borg.
<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/james.jpg" alt="Colaborador sem gerente" width="800" height="200"/>
  
Após a transformação dos dados, o database foi importado pelo Power BI (desktop) no modo de compatibilidade de dados Power Query, para a realização de análise e relatório.

<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/conexao1.jpg" alt="Conexão Power BI - PowerQuery" width="550" height="330"/>

No BI observamos a estrutura de dados abaixo (visualização parcial) :

<img src="https://github.com/lucianeb/desafio_azure_bi/blob/main/dadosbi.jpg" alt="Importaçao com PowerQuery BI" width="450" height="780"/>

O numero de colaboradore por gerente foi realizado através da consulta :
```
SELECT e.Super_ssn AS Gerente_Ssn, CONCAT(m.Fname, ' ', m.Lname) AS Nome_Gerente, COUNT(e.Ssn) AS Numero_Funcionarios
FROM azure_company.employee e
LEFT JOIN azure_company.employee m ON e.Super_ssn = m.Ssn
GROUP BY e.Super_ssn, m.Fname, m.Lname;
```
## Arquivo BI do desafio 
O arquivo gerado para caracterização dso dados pode ser acessado em : https://github.com/lucianeb/desafio_azure_bi/blob/main/relatorio_desafio_azure_bi_dio.pbix

## 🌐 Alguma dúvida ?

[![Gmail](https://img.shields.io/badge/Gmail-333333?style=for-the-badge&logo=gmail&logoColor=red)](mailto:lurutilae@gmail.com)











