# ETL proces datasetu Northwind

Tento repozitár obsahuje implementáciu ETL procesu v Snowflake pre analýzu dát z datasetu Northwind. Projekt sa zameriava na analýzu predajných trendov, preferencií zákazníkov a optimalizáciu logistických procesov na základe údajov o kategóriách produktov, zákazníkoch, zamestnancoch, objednávkach a dodávkach. Výsledný dátový model umožňuje multidimenzionálnu analýzu a vizualizáciu kľúčových metrík.  

## 1. Úvod a popis zdrojových dát   

Cieľom semestrálneho projektu je analyzovať dáta týkajúce sa kategórií produktov, zákazníkov, zamestnancov, objednávok a dodávok. Táto analýza umožňuje identifikovať trendy v predaji, preferenciách zákazníkov a optimalizovať procesy v rámci spoločnosti.

Zdrojové dáta pochádzajú z datasetu Northwind dostupného [tu](https://edu.ukf.sk/mod/folder/view.php?id=252869). Dataset obsahuje nasledujúce hlavné tabuľky:  

- categories
- products
- suppliers
- orderdetails
- shippers
- orders
- employees
- customers

Categories — obsahuje informácie o kategóriách produktov:
- CategoryID — unikátny identifikátor kategórie.
- CategoryName — názov kategórie.
- Description — popis kategórie.

Customers — údaje o zákazníkoch:
- CustomerID — unikátny identifikátor zákazníka.
- CustomerName — názov organizácie alebo meno zákazníka.
- ContactName — kontaktná osoba.
- Address — adresa zákazníka.
- City — mesto zákazníka.
- PostalCode — poštové smerovacie číslo.
- Country — krajina zákazníka.

Employees — informácie o zamestnancoch:
- EmployeeID — unikátny identifikátor zamestnanca.
- LastName — priezvisko zamestnanca.
- FirstName — meno zamestnanca.
- BirthDate — dátum narodenia zamestnanca.
- Photo — cesta k obrázku zamestnanca.
- Notes — dodatočné poznámky o zamestnancovi.

Shippers — údaje o spoločnostiach, ktoré zabezpečujú prepravu:
- ShipperID — unikátny identifikátor dopravcu.
- ShipperName — názov spoločnosti.
- Phone — kontaktné telefónne číslo.

Suppliers — informácie o dodávateľoch:
- SupplierID — unikátny identifikátor dodávateľa.
- SupplierName — názov spoločnosti dodávateľa.
- ContactName — kontaktná osoba dodávateľa.
- Address — adresa dodávateľa.
- City — mesto dodávateľa.
- PostalCode — poštové smerovacie číslo.
- Country — krajina dodávateľa.
- Phone — kontaktné telefónne číslo.

Products — obsahuje informácie o produktoch:
- ProductID — unikátny identifikátor produktu.
- ProductName — názov produktu.
- SupplierID — identifikátor dodávateľa (vzťah s tabuľkou Suppliers).
- CategoryID — identifikátor kategórie produktu (vzťah s tabuľkou Categories).
- Unit — jednotka merania produktu.
- Price — cena produktu.

Orders — údaje o objednávkach:
- OrderID — unikátny identifikátor objednávky.
- CustomerID — identifikátor zákazníka, ktorý zadal objednávku (vzťah s tabuľkou Customers).
- EmployeeID — identifikátor zamestnanca, ktorý spracoval objednávku (vzťah s tabuľkou Employees).
- OrderDate — dátum zadania objednávky.
- ShipperID — identifikátor dopravcu, ktorý doručil objednávku (vzťah s tabuľkou Shippers).

OrderDetails — podrobnosti objednávok:
- OrderDetailID — unikátny identifikátor záznamu.
- OrderID — identifikátor objednávky (vzťah s tabuľkou Orders).
- ProductID — identifikátor produktu (vzťah s tabuľkou Products).
- Quantity — množstvo objednaného produktu.

Účelom ETL procesu bolo tieto dáta pripraviť, transformovať a sprístupniť pre viacdimenzionálnu analýzu.

### 1.1. Dátová architektúra
ERD diagram
Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na entitno-relačnom diagrame (ERD):

![Entitno-relačná schéma Northwind_ERD](https://github.com/Wilkwarin/Northwind/blob/main/Northwind_ERD.png)

*Obrázok 1 Entitno-relačná schéma Northwind_ERD*

---

# 2. Návrh dimenzionálneho modelu

Navrhnutý bol hviezdicový model (star schema), kde centrálny bod predstavuje faktová tabuľka Fact_Sales, ktorá je prepojená s nasledujúcimi dimenziami:

### Popis faktovej tabuľky: Fact_Sales

Faktová tabuľka Fact_Sales obsahuje nasledujúce kľúče a metriky:

Primárny kľúč:
- SaleID — jedinečný identifikátor transakcie.

Cudzie kľúče:
- OrderID — identifikátor objednávky.
- ShipperID — identifikátor prepravcu.
- ProductID — identifikátor produktu (prepojenie s dimenziou DM_Products).
- SupplierID — identifikátor dodávateľa (prepojenie s dimenziou DM_Suppliers).
- CustomerID — identifikátor zákazníka (prepojenie s dimenziou DM_Customers).
- DateID — identifikátor dátumu objednávky (prepojenie s dimenziou DM_Date).

Metriky:
- EmployeeAgeAtOrder - vek zamestnanca v čase spracovania objednávky.
- Quantity — počet predaných jednotiek produktu.
- Suma — celková suma transakcie, vypočítaná ako Quantity * Cena (kde Cena pochádza z DM_Products).

### Popis dimenzií a ich prepojenia s faktovou tabuľkou

DM_Products:
- Obsahuje informácie o produktoch vrátane názvu (ProductName), kategórie (ProductCategory), jednotky (Unit) a ceny (Price).
- Typ dimenzie: SCD typu 1 (Slowly Changing Dimension Type 1 - keď sa údaje v dimenzii zmenia, staré údaje sú úplne nahradené novými. História zmien sa pritom neuchováva).

DM_Suppliers:
- Obsahuje informácie o dodávateľoch vrátane mesta (City) a krajiny (Country).
- Typ dimenzie: SCD typu 1 (aktuálne údaje).

DM_Customers:
- Obsahuje geografické údaje o zákazníkoch, ako sú mesto (City) a krajina (Country).
- Typ dimenzie: SCD typu 1 (aktuálne údaje).

DM_Date:
- Obsahuje informácie o dátumoch objednávok (OrderDate), vrátane roku (Year), mesiaca (Month) a dňa (Day).
- Typ dimenzie: Statická (nemenná - údaje v dimenzii sa časom nemenia).

Štruktúra hviezdicového modelu je znázornená na diagrame nižšie. Diagram ukazuje prepojenia medzi faktovou tabuľkou a dimenziami, čo zjednodušuje pochopenie a implementáciu modelu.

![Obrázok 2 Schéma hviezdy pre NorthWind](https://github.com/Wilkwarin/Northwind/blob/main/Sch%C3%A9ma%20hviezdy%20pre%20NorthWind.png)
 
*Obrázok 2 Schéma hviezdy pre NorthWind*

---

# 3. ETL proces v Snowflake

ETL proces pozostával z troch hlavných fáz: extrahovanie (Extract), transformácia (Transform) a načítanie (Load). Tento proces bol implementovaný v Snowflake s cieľom pripraviť zdrojové dáta zo staging vrstvy do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

## 3.1. Príprava databázy (Setup)

Na začiatku boli vytvorené základné komponenty Snowflake: dátový sklad (warehouse), databáza a tabuľky pre viacdimenzionálny model.

Vytvorenie dátového skladu a databázy:
```sql
USE ROLE TRAINING_ROLE;
CREATE WAREHOUSE IF NOT EXISTS COBRA_WH;
USE WAREHOUSE COBRA_WH;
CREATE DATABASE IF NOT EXISTS COBRA_NorthWind;
USE COBRA_NorthWind;
```
### Vytvorenie tabuliek dimenzií a faktov:

Faktová tabuľka Fact_Sales:
```sql
CREATE OR REPLACE TABLE Fact_Sales (
    SaleID INTEGER NOT NULL,
    OrderID INTEGER NOT NULL,
    ShipperID INTEGER NOT NULL,
    EmployeeAgeAtOrder INTEGER NOT NULL,
    Quantity INTEGER NOT NULL,
    Suma DECIMAL(16, 2),
    ProductID INTEGER NOT NULL,
    SupplierID INTEGER NOT NULL,
    CustomerID INTEGER NOT NULL,
    DateID INTEGER NOT NULL
);
```
Dimenzionálna tabuľka DM_Products:
```sql
CREATE OR REPLACE TABLE DM_Products (
    ProductID INTEGER NOT NULL,
    ProductName VARCHAR(50),
    ProductCategory VARCHAR(25),
    Unit VARCHAR(25),
    Price DECIMAL(10, 2)
);
```
Dimenzionálna tabuľka DM_Suppliers:
```sql
CREATE OR REPLACE TABLE DM_Suppliers (
    SupplierID INTEGER NOT NULL,
    City VARCHAR(50),
    Country VARCHAR(50)
);
```
Dimenzionálna tabuľka DM_Customers:
```sql
CREATE OR REPLACE TABLE DM_Customers (
    CustomerID INTEGER NOT NULL,
    City VARCHAR(50),
    Country VARCHAR(50)
);
```
Dimenzionálna tabuľka DM_Date:
```sql
CREATE OR REPLACE TABLE DM_Date (
    DateID INTEGER NOT NULL,
    OrderDate DATE,
    OrderYear INTEGER NOT NULL,
    OrderMonth INTEGER NOT NULL,
    OrderDay INTEGER NOT NULL
);
```

## 3.2. Extract (Extrahovanie dát)

Dáta zo zdrojového datasetu (formát .csv) boli najprv nahraté do Snowflake prostredníctvom interného stage úložiska s názvom COBRA_NorthWindStage. Stage v Snowflake slúži ako dočasné úložisko na import alebo export dát. Vytvorenie stage bolo zabezpečené príkazom:
```sql
CREATE OR REPLACE STAGE COBRA_NorthWindStage;
```
Na zabezpečenie správneho formátu importovaných dát bol vytvorený formát súborov typu CSV pomocou príkazu:
```sql
CREATE OR REPLACE FILE FORMAT CSV_FORMAT
  TYPE = CSV
  COMPRESSION = NONE
  SKIP_HEADER = 1
  FIELD_DELIMITER = ','
  RECORD_DELIMITER = '\n'
  FILE_EXTENSION = 'csv'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"';
```
CSV súbory boli následne manuálne nahraté do stage prostredníctvom webového rozhrania Snowflake. Stav stage bol overený príkazom:
```sql
LIST @COBRA_NorthWindStage;
```

## 3.3. Transform (Transformácia dát)

Transformácia dát zahŕňala vytvorenie dočasných tabuliek, načítanie dát z CSV súborov a ich prípravu na načítanie do tabuliek dimenzií a faktov.

### 3.3.1. Načítanie zákazníkov (Customers)

Dočasná tabuľka pre zákazníkov bola vytvorená nasledovne:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Customers (
    CustomerID INTEGER,
    CustomerName VARCHAR(50),
    ContactName VARCHAR(50),
    Address VARCHAR(50),
    City VARCHAR(20),
    PostalCode VARCHAR(10),
    Country VARCHAR(15)
);
```
Dáta z CSV súboru Customers.csv boli skopírované do dočasnej tabuľky:
```sql
COPY INTO Temp_Customers
FROM @COBRA_NorthWindStage/Customers.csv
FILE_FORMAT = CSV_FORMAT;
```
### 3.3.2. Načítanie dodávateľov (Suppliers)

Podobný postup bol použitý na načítanie dodávateľov:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Suppliers (
    SupplierID INTEGER,
    SupplierName VARCHAR(50),
    ContactName VARCHAR(50),
    Address VARCHAR(50),
    City VARCHAR(20),
    PostalCode VARCHAR(10),
    Country VARCHAR(15),
    Phone VARCHAR(15)
);
```
Načítanie dát z CSV súboru:
```sql
COPY INTO Temp_Suppliers
FROM @COBRA_NorthWindStage/Suppliers.csv
FILE_FORMAT = CSV_FORMAT;
```

### 3.3.3. Príprava produktov (Products)
Dočasná tabuľka pre produkty:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Products (
    ProductID INTEGER,
    ProductName VARCHAR(50),
    SupplierID INTEGER,
    CategoryID INTEGER,
    Unit VARCHAR(25),
    Price DECIMAL(10,2)
);
```
Načítanie dát z Products.csv:
```sql
COPY INTO Temp_Products
FROM @COBRA_NorthWindStage/Products.csv
FILE_FORMAT = CSV_FORMAT;
```
Dočasná tabuľka pre kategórie:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Categories (
    CategoryID INTEGER,
    CategoryName VARCHAR(50),
    Description VARCHAR(255)
);
```
Načítanie dát z Categories.csv:
```sql
COPY INTO Temp_Categories
FROM @COBRA_NorthWindStage/Categories.csv
FILE_FORMAT = CSV_FORMAT;
```

### 3.3.4. Príprava dátumu objednávok (Date)

Vytvorenie dočasnej tabuľky pre objednávky:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Orders (
    OrderID INTEGER,
    CustomerID INTEGER,
    EmployeeID INTEGER,
    OrderDate DATETIME,
    ShipperID INTEGER
);
```
Načítanie dát z Orders.csv:
```sql
COPY INTO Temp_Orders
FROM @COBRA_NorthWindStage/Orders.csv
FILE_FORMAT = CSV_FORMAT;
```

## 3.4. Load (Načítanie dát do faktovej tabuľky)

Do dimenzionálnej tabuľky DM_Customers boli nahrané iba relevantné údaje (CustomerID, City, Country):
```sql
INSERT INTO DM_Customers (CustomerID, City, Country)
SELECT 
    CustomerID,
    City,
    Country
FROM Temp_Customers;
```

Vloženie relevantných údajov do DM_Suppliers:
```sql
INSERT INTO DM_Suppliers (SupplierID, City, Country)
SELECT 
    SupplierID,
    City,
    Country
FROM Temp_Suppliers;
```

Vloženie produktov do DM_Products:
```sql
    INSERT INTO DM_Products (ProductID, ProductName, ProductCategory, Unit, Price)
    SELECT 
        p.ProductID,
        p.ProductName,
        c.CategoryName AS ProductCategory,
        p.Unit,
        p.Price
    FROM Temp_Products p
    JOIN Temp_Categories c ON p.CategoryID = c.CategoryID;
```

Extrahovanie unikátnych dátumov, priradenie k nim identifikátory DateID a vloženie do DM_Date:
```sql
INSERT INTO DM_Date (DateID, OrderDate, OrderYear, OrderMonth, OrderDay)
SELECT 
    ROW_NUMBER() OVER (ORDER BY OrderDate) AS DateID,
    OrderDate,
    YEAR(OrderDate) AS OrderYear,
    MONTH(OrderDate) AS OrderMonth,
    DAY(OrderDate) AS OrderDay
FROM (
    SELECT DISTINCT OrderDate
    FROM Temp_Orders
) AS DistinctOrders;
```

Faktová tabuľka Fact_Sales bola naplnená spojením dát z rôznych dočasných tabuliek (Temp_Orders, Temp_OrderDetails, Temp_Employees, Temp_Products) a dimenzionálnej tabuľky DM_Date. 
Kľúčové kroky:

Načítanie detailov objednávok:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_OrderDetails (
    OrderDetailID INTEGER,
    OrderID INTEGER,
    ProductID INTEGER,
    Quantity INTEGER
);
```
Načítanie dát z OrderDetails.csv:
```sql
COPY INTO Temp_OrderDetails
FROM @COBRA_NorthWindStage/OrderDetails.csv
FILE_FORMAT = CSV_FORMAT;
```
Dočasná tabuľka pre zamestnancov:
```sql
CREATE OR REPLACE TEMPORARY TABLE Temp_Employees (
    EmployeeID INTEGER,
    LastName VARCHAR(15),
    FirstName VARCHAR(15),
    BirthDate DATE,
    Photo VARCHAR(25),
    Notes VARCHAR(1024)
);
```
Načítanie dát z Employees.csv:
```sql
COPY INTO Temp_Employees
FROM @COBRA_NorthWindStage/Employees.csv
FILE_FORMAT = CSV_FORMAT;
```
Naplnenie faktovej tabuľky:
```sql
INSERT INTO Fact_Sales (SaleID, OrderID, ShipperID, EmployeeAgeAtOrder, Quantity, Suma, ProductID, SupplierID, CustomerID, DateID)
SELECT
    od.OrderDetailID AS SaleID,
    o.OrderID,
    o.ShipperID,
    FLOOR(DATEDIFF('day', te.BirthDate, o.OrderDate) / 365) AS EmployeeAgeAtOrder,
    od.Quantity,
    od.Quantity * tp.Price AS Suma,
    od.ProductID,
    tp.SupplierID,
    o.CustomerID,
    d.DateID
FROM Temp_Orders o
JOIN Temp_OrderDetails od ON o.OrderID = od.OrderID
JOIN Temp_Products tp ON od.ProductID = tp.ProductID
JOIN DM_Date d ON o.OrderDate = d.OrderDate
JOIN Temp_Employees te ON o.EmployeeID = te.EmployeeID;
```

Po úspešnom vytvorení dimenzií a faktovej tabuľky boli dáta nahraté do finálnej štruktúry. Na záver boli staging tabuľky odstránené, aby sa optimalizovalo využitie úložiska:
```sql
DROP TABLE IF EXISTS Temp_Customers;
DROP TABLE IF EXISTS Temp_Suppliers;
DROP TABLE IF EXISTS Temp_Employees;
DROP TABLE IF EXISTS Temp_Orders;
DROP TABLE IF EXISTS Temp_Products;
DROP TABLE IF EXISTS Temp_Categories;
DROP TABLE IF EXISTS Temp_OrderDetails;
```
ETL proces v Snowflake umožnil spracovanie pôvodných dát z .csv formátu do viacdimenzionálneho modelu typu hviezda. Tento proces zahŕňal čistenie, obohacovanie a reorganizáciu údajov. Výsledný model umožňuje analýzu predajov, správanie zákazníkov a výkonnosť zamestnancov, pričom poskytuje základ pre vizualizácie a reporty.
---
# 4. Vizualizácia dát.



