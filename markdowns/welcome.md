# Workshop: Creación de procedimientos almacenados en T-SQL

## Pasos previos al taller:
1. Descargar la base de datos [SQL Server Developer](https://www.microsoft.com/es-es/sql-server/sql-server-downloads)
2. Descargar la base de datos [AdventureWorks](https://learn.microsoft.com/es-es/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)
3. Instalar algun programa para conectarse a la base de datos ([SQL Server Management Studio](https://learn.microsoft.com/es-es/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16), Azure Data Studio, DBeaver u otro)
4. Importar la base de datos AdventureWorks en SQL Server.
    - Click derecho sobre "Databases"
    - Click en la opción "Restore Database"
    - Seleccionar la opción "Device" y seleccionar el archivo (AdventureWorks2022.bak) descargado.
    - La base de datos AdventureWorks debe aparecer en la lista de bases de datos.


## Que es T-SQL (Transact SQL)
Proporciona un lenguaje de programación sólido con características que permiten almacenar temporalmente valores en variables, aplicar la ejecución condicional de comandos, pasar parámetros a procedimientos almacenados y controlar el flujo de los programas.


## Que es un Procedimiento Almacenado
Son grupos de instrucciones que se pueden usar y reutilizar siempre que se necesiten, pueden devolver resultados, manipular datos y realizar acciones administrativas en el servidor. 
Pueden contener sentencias DDL y DML.
Algunas ventajas de los Procedimientos almacenados:
- Reutilización del código
- Seguridad
- Mejorar el rendimiento
- Mantenimiento inferior

::: Sintaxis en T-SQL:
### Declarar Variables
```sql
DECLARE @var1 AS INT = 99;
DECLARE @var2 AS NVARCHAR(255);
SET @var2 = N'string';
```

### IF
```sql
IF OBJECT_ID('HR.Employees') IS NULL --this object does exist in the sample database
BEGIN
    PRINT 'The specified object does not exist';
END;

IF OBJECT_ID('HR.Employees') IS NULL --this object does exist in the sample database
BEGIN
    PRINT 'The specified object does not exist';
END
ELSE
BEGIN
    PRINT 'The specified object exists';
END;
```

### Ciclos
```sql
DECLARE @empid AS INT = 1, @lname AS NVARCHAR(20);
WHILE @empid <=5
   BEGIN
	SELECT @lname = lastname FROM HR.Employees
		WHERE empid = @empid;
	PRINT @lname;
	SET @empid += 1;
   END;
```
:::

## Sintaxis creación de Stored Procedures
```sql
CREATE PROCEDURE <ProcedureName>
   @<ParameterName1> <data type>,
   @<ParameterName2> <data type>
AS   

   SET NOCOUNT ON;
   SELECT <your SELECT statement>;
GO

EXECUTE <ProcedureName> N'<Parameter 1 value>, N'<Parameter x value>;  
GO
```

Ejemplo:
```sql
CREATE PROCEDURE Person.GetPerson
    @LastName nvarchar(50),
    @FirstName nvarchar(50)
AS   

    SET NOCOUNT ON;
    SELECT FirstName, LastName, Title
    FROM Person.Person
    WHERE FirstName = @FirstName AND LastName = @LastName;
GO
```

