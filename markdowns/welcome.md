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
IF @var2 = 'string'  --this object does exist in the sample database
BEGIN
    PRINT 'Hi :)';
END;

IF string = 'string' --this object does exist in the sample database
BEGIN
    PRINT 'Hi :)';
END
ELSE
BEGIN
    PRINT 'Ah :(';
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

```sql
CREATE PROCEDURE Sales.GetCustomerCountry
AS   
	SET NOCOUNT ON;

	SELECT *
	FROM Sales.vIndividualCustomer IC
	INNER JOIN Person.vStateProvinceCountryRegion CR ON IC.CountryRegionName = CR.CountryRegionName

GO

EXEC Sales.GetCustomerCountry
```

```sql
CREATE TABLE PersonReport (
	PersonReportID INT IDENTITY,
	BusinessEntityID BIGINT, 
	PersonType NVARCHAR(2), 
	FirstName NVARCHAR(100), 
	LastName NVARCHAR(100),
	CreatedDate DATETIME, 
	TransactionsCount BIGINT);
```

Ejercicios durante el taller:

```sql
EXEC Person.GetPerson @FirstName = 'Ken', @LastName = 'Sánchez'
DROP PROCEDURE Person.GetPerson

CREATE OR ALTER PROCEDURE Sales.GetCustomerCountry
AS   
	SET NOCOUNT ON;

	--exec Person.GetPerson

	SELECT BusinessEntityID, Title, FirstName, MiddleName, LastName, Suffix, PhoneNumber,
	PhoneNumberType, EmailAddress, EmailPromotion, AddressLine1, AddressLine2, City,
	StateProvinceName, PostalCode, CountryRegionName
	into #vIndividualCustomer
	FROM Sales.vIndividualCustomer;

	SELECT *
	into #vStateProvinceCountryRegion
	FROM Person.vStateProvinceCountryRegion;


	SELECT *
	FROM #vIndividualCustomer IC
	INNER JOIN #vStateProvinceCountryRegion CR ON IC.CountryRegionName = CR.CountryRegionName;

GO




CREATE or alter PROCEDURE Sales.GenerateTransactionsCount
AS
BEGIN

	SET NOCOUNT ON;

	declare @PersonReport TABLE (
		BusinessEntityID BIGINT, 
		PersonType NVARCHAR(2), 
		FirstName NVARCHAR(100), 
		LastName NVARCHAR(100),
		CreatedDate DATETIME, 
		TransactionsCount BIGINT);


	SELECT *
	INTO #Persons
	FROM (
		SELECT BusinessEntityID, PersonType, FirstName, LastName,
		RANK() OVER (PARTITION BY PersonType ORDER BY BusinessEntityID) position
		FROM Person.Person
	) T 
	WHERE position <= 300;

	INSERT INTO @PersonReport (BusinessEntityID, PersonType, FirstName, LastName, CreatedDate, TransactionsCount) 
	select P.BusinessEntityID, P.PersonType, P.FirstName, P.LastName, GETDATE(), COUNT(H.BusinessEntityID) TransactionCount
	from #Persons P
	inner join HumanResources.EmployeePayHistory H ON H.BusinessEntityID = P.BusinessEntityID
	group by P.BusinessEntityID, P.PersonType, P.FirstName, P.LastName;

	declare @BusinessEntityID BIGINT, 
		@PersonType NVARCHAR(2), 
		@FirstName NVARCHAR(100), 
		@LastName NVARCHAR(100),
		--CreatedDate DATETIME, 
		@TransactionsCount BIGINT;

	DECLARE c_EmployeeOrders cursor for
	SELECT P.BusinessEntityID, P.PersonType, P.FirstName, P.LastName, count(C.EmployeeID) TransactionsCount
	FROM #Persons P
	INNER JOIN Purchasing.PurchaseOrderHeader C ON P.BusinessEntityID = C.EmployeeID
	group by P.BusinessEntityID, P.PersonType, P.FirstName, P.LastName;

	open c_EmployeeOrders
	fetch next from c_EmployeeOrders
	into @BusinessEntityID, @PersonType, @FirstName, @LastName, @TransactionsCount

	while @@FETCH_STATUS = 0
	begin
		INSERT INTO @PersonReport (BusinessEntityID, PersonType, FirstName, LastName, CreatedDate, TransactionsCount)
		VALUES (@BusinessEntityID, @PersonType, @FirstName, @LastName, GETDATE(), @TransactionsCount);


		fetch next from c_EmployeeOrders
		into @BusinessEntityID, @PersonType, @FirstName, @LastName, @TransactionsCount

	end;
	close c_EmployeeOrders
	deallocate c_EmployeeOrders;

	insert into PersonReport
	select *
	from @PersonReport;

END;

exec Sales.GenerateTransactionsCount


select *
from Sales.PersonReport 
```
