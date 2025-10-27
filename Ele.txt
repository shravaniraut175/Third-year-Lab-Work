CREATE TABLE Time_Dim (
    Time_ID INT PRIMARY KEY,
    Day VARCHAR(10),
    Month VARCHAR(10),
    Quarter VARCHAR(10),
    Year INT
);

CREATE TABLE Item_Dim (
    Item_ID INT PRIMARY KEY,
    Item_Name VARCHAR(50),
    Brand VARCHAR(50),
    Category VARCHAR(50)
);

CREATE TABLE Branch_Dim (
    Branch_ID INT PRIMARY KEY,
    Branch_Name VARCHAR(50),
    Manager VARCHAR(50)
);

CREATE TABLE Location_Dim (
    Location_ID INT PRIMARY KEY,
    City VARCHAR(50),
    State VARCHAR(50),
    Country VARCHAR(50)
);

CREATE TABLE Sales_Fact (
    Sales_ID INT PRIMARY KEY,
    Time_ID INT,
    Item_ID INT,
    Branch_ID INT,
    Location_ID INT,
    Dollars_Sold DECIMAL(10,2),
    Units_Sold INT,
    FOREIGN KEY (Time_ID) REFERENCES Time_Dim(Time_ID),
    FOREIGN KEY (Item_ID) REFERENCES Item_Dim(Item_ID),
    FOREIGN KEY (Branch_ID) REFERENCES Branch_Dim(Branch_ID),
    FOREIGN KEY (Location_ID) REFERENCES Location_Dim(Location_ID)
);


-- Time Dimension
INSERT INTO Time_Dim VALUES
(1, '01', 'January', 'Q1', 2025),
(2, '15', 'February', 'Q1', 2025),
(3, '10', 'March', 'Q1', 2025);

-- Item Dimension
INSERT INTO Item_Dim VALUES
(1, 'Smartphone', 'Samsung', 'Electronics'),
(2, 'Laptop', 'HP', 'Computers'),
(3, 'TV', 'Sony', 'Electronics');

-- Branch Dimension
INSERT INTO Branch_Dim VALUES
(1, 'Andheri Branch', 'Ravi Kumar'),
(2, 'Bandra Branch', 'Neha Sharma');

-- Location Dimension
INSERT INTO Location_Dim VALUES
(1, 'Mumbai', 'Maharashtra', 'India'),
(2, 'Pune', 'Maharashtra', 'India');

-- Fact Table
INSERT INTO Sales_Fact VALUES
(1, 1, 1, 1, 1, 50000.00, 10),
(2, 2, 2, 1, 2, 75000.00, 5),
(3, 3, 3, 2, 1, 120000.00, 8);


SELECT * FROM Time_Dim;
SELECT * FROM Item_Dim;
SELECT * FROM Branch_Dim;
SELECT * FROM Location_Dim;
SELECT * FROM Sales_Fact;

 d) OLAP Operations
1 ROLL-UP
→ Aggregates data by climbing up the hierarchy (e.g., from Month to Quarter to Year)
Example: Show total dollars sold per year


SELECT T.Year, SUM(S.Dollars_Sold) AS Total_Sales
FROM Sales_Fact S
JOIN Time_Dim T ON S.Time_Key = T.Time_Key
GROUP BY T.Year;
Roll up further: Show total sales per quarter
SELECT T.Year, T.Quarter, SUM(S.Dollars_Sold) AS Total_Sales
FROM Sales_Fact S
JOIN Time_Dim T ON S.Time_Key = T.Time_Key
GROUP BY T.Year, T.Quarter;

2 DRILL-DOWN
→ Moves from summary level to more detailed data (e.g., from Year → Month → Day)
Example: Show sales per month


SELECT T.Year, T.Month, SUM(S.Dollars_Sold) AS Total_Sales
FROM Sales_Fact S
JOIN Time_Dim T ON S.Time_Key = T.Time_Key
GROUP BY T.Year, T.Month;
Further Drill Down: Show sales per day
SELECT T.Year, T.Month, T.Day, SUM(S.Dollars_Sold) AS Total_Sales
FROM Sales_Fact S
JOIN Time_Dim T ON S.Time_Key = T.Time_Key
GROUP BY T.Year, T.Month, T.Day;

1️⃣ Slice Operation
Slice means selecting a single dimension value to view a subset of the cube.
Example → View all sales for January 2025 only:

SELECT 
    i.Item_Name, 
    b.Branch_Name, 
    l.City, 
    s.Dollars_Sold, 
    s.Units_SoldFROM Sales_Fact sJOIN Time_Dim t ON s.Time_ID = t.Time_IDJOIN Item_Dim i ON s.Item_ID = i.Item_IDJOIN Branch_Dim b ON s.Branch_ID = b.Branch_IDJOIN Location_Dim l ON s.Location_ID = l.Location_IDWHERE t.Month = 'January' AND t.Year = 2025;

2️⃣ Dice Operation
Dice selects data from multiple dimensions using conditions on more than one dimension.
Example → View sales for Electronics category in Mumbai during Q1 2025:


SELECT 
    t.Month, 
    i.Item_Name, 
    b.Branch_Name, 
    l.City, 
    s.Dollars_Sold, 
    s.Units_SoldFROM Sales_Fact sJOIN Time_Dim t ON s.Time_ID = t.Time_IDJOIN Item_Dim i ON s.Item_ID = i.Item_IDJOIN Branch_Dim b ON s.Branch_ID = b.Branch_IDJOIN Location_Dim l ON s.Location_ID = l.Location_IDWHERE i.Category = 'Electronics'
  AND l.City = 'Mumbai'
  AND t.Quarter = 'Q1'
  AND t.Year = 2025;