# 📊 Koha SQL Reports Documentation

## 📌 Overview
This repository contains SQL queries for generating various reports from the **Koha Library Management System**. These reports help in tracking library books, borrowers, overdue books, and more. 

---

## 📝 **SQL Reports**

### **1️⃣ Total Number of Books in the Library**
**Report Name:** `Total Books in the Library`
```sql
SELECT COUNT(*) AS Total_Books FROM items;
```
✅ **Description:** Counts all books available in the library.

---

### **2️⃣ Total Number of Books Issued**
**Report Name:** `Total Books Issued`
```sql
SELECT COUNT(*) AS Total_Issued_Books FROM issues;
```
✅ **Description:** Counts all books currently borrowed by users.

---

### **3️⃣ Total Number of Active Borrowers**
**Report Name:** `Total Active Borrowers`
```sql
SELECT COUNT(DISTINCT borrowernumber) AS Active_Borrowers FROM issues;
```
✅ **Description:** Retrieves the number of unique borrowers with issued books.

---

### **4️⃣ List of Students Who Borrowed Books**
**Report Name:** `List of Borrowers and Books Issued`
```sql
SELECT 
    b.cardnumber AS Student_ID,
    CONCAT(b.surname, ' ', b.firstname) AS Student_Name,
    COUNT(i.itemnumber) AS Books_Issued
FROM borrowers b
JOIN issues i ON b.borrowernumber = i.borrowernumber
GROUP BY b.cardnumber, Student_Name
ORDER BY Books_Issued DESC;
```
✅ **Description:** Retrieves student details and the count of books borrowed.

---

### **5️⃣ Books Issued per Subject**
**Report Name:** `Books Issued by Subject`
```sql
SELECT 
    ExtractValue(bmd.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Subject,
    COUNT(i.itemnumber) AS Books_Issued
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio_metadata bmd ON it.biblionumber = bmd.biblionumber
GROUP BY Subject
ORDER BY Books_Issued DESC;
```
✅ **Description:** Retrieves books issued, grouped by subject.

---

### **6️⃣ Top 25 Most Issued Books**
**Report Name:** `Most Popular Books (Top 25)`
```sql
SELECT 
    bib.title AS Book_Title,
    COUNT(i.itemnumber) AS Times_Issued
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
GROUP BY bib.title
ORDER BY Times_Issued DESC
LIMIT 25;
```
✅ **Description:** Lists the top 25 most borrowed books.

---

### **7️⃣ Books Acquired Yearly**
**Report Name:** `Books Added to Library Per Year`
```sql
SELECT 
    YEAR(it.dateaccessioned) AS Year_Acquired, 
    COUNT(it.itemnumber) AS Books_Acquired
FROM items it
WHERE it.dateaccessioned IS NOT NULL
GROUP BY YEAR(it.dateaccessioned)
ORDER BY Year_Acquired DESC;
```
✅ **Description:** Displays books acquired per year.

---

### **8️⃣ List of Overdue Books**
**Report Name:** `Overdue Books and Borrowers`
```sql
SELECT 
    b.cardnumber AS Student_ID,
    CONCAT(b.surname, ' ', b.firstname) AS Student_Name,
    bib.title AS Book_Title,
    i.date_due AS Due_Date
FROM issues i
JOIN borrowers b ON i.borrowernumber = b.borrowernumber
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
WHERE i.date_due < CURDATE()
ORDER BY i.date_due ASC;
```
✅ **Description:** Returns overdue books with borrower details.

---

### **9️⃣ Books Issued in Arabic**
**Report Name:** `Most Borrowed Arabic Books`
```sql
SELECT 
    bib.title AS Book_Title,
    COUNT(i.itemnumber) AS Times_Issued
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
WHERE ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') LIKE '%ara%'
GROUP BY bib.title
ORDER BY Times_Issued DESC
LIMIT 25;
```
✅ **Description:** Filters and lists Arabic books based on MARC data.

---

### **🔟 List of Lost Books**
**Report Name:** `Lost Books Report`
```sql
SELECT 
    bib.title AS Book_Title,
    bib.author AS Author,
    it.barcode AS Barcode,
    it.itemlost AS Lost_Status
FROM items it
JOIN biblio bib ON it.biblionumber = bib.biblionumber
WHERE it.itemlost > 0;
```

### **📚 Books Acquired Yearly**
**Report Name:** `Books Acquired Per Year`
```sql
SELECT 
    YEAR(bi.timestamp) AS Year_Acquired,
    b.title AS Book_Title,
    COUNT(b.biblionumber) AS Total_Acquired
FROM biblioitems bi
JOIN biblio b ON bi.biblionumber = b.biblionumber
WHERE bi.timestamp IS NOT NULL
GROUP BY Year_Acquired, Book_Title
ORDER BY Year_Acquired DESC, Total_Acquired DESC;



✅ **Description:** Retrieves books marked as lost in the system.

---
### 10. Student Book Borrowing Frequency Report  
📊 **Report Name:** **Student Book Borrowing Frequency Report**  

###  Description  
This report provides a **detailed breakdown** of books borrowed by each student, including:  
- **Student details** (ID, Name, User ID).  
- **Total books borrowed**.  
- **Book titles** and **subjects**.  
- **How many times each book was borrowed by the same student**.  

###  SQL Query  
```sql
SELECT 
    b.borrowernumber AS Student_ID,
    b.cardnumber AS User_ID,
    CONCAT(b.surname, ' ', b.firstname) AS Student_Name,
    COUNT(i.itemnumber) AS Total_Books_Issued,
    bib.title AS Book_Title,
    ExtractValue(bmd.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Subject,
    COUNT(i.itemnumber) AS Times_Issued
FROM borrowers b
LEFT JOIN issues i ON b.borrowernumber = i.borrowernumber
LEFT JOIN items it ON i.itemnumber = it.itemnumber
LEFT JOIN biblio bib ON it.biblionumber = bib.biblionumber
LEFT JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
GROUP BY b.borrowernumber, b.cardnumber, Student_Name, bib.title, Subject
ORDER BY Total_Books_Issued DESC, Times_Issued DESC;
```

### 11. SQL CODE FOR TOP 25 ARABIC BOOKS ISSUED

```
SELECT 
    bib.title AS Book_Title, 
    COUNT(i.itemnumber) AS Issued_Count,
    ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') AS Language
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
WHERE ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') = 'Arabic'
GROUP BY bib.title
ORDER BY Issued_Count DESC
LIMIT 25;

```

### 12. TOP 25 ENGLISH BOOKS ISSUED

```
SELECT 
    bib.title AS Book_Title, 
    COUNT(i.itemnumber) AS Issued_Count,
    ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') AS Language
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
WHERE ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') = 'English'
GROUP BY bib.title
ORDER BY Issued_Count DESC
LIMIT 25;


```

### 13. Top 25 Lisan-ud-Dawat Books (Most Issued)
```SELECT 
    bib.title AS Book_Title, 
    COUNT(i.itemnumber) AS Issued_Count,
    ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') AS Language
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
WHERE ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') = 'Lisan-ud-Dawat'
GROUP BY bib.title
ORDER BY Issued_Count DESC
LIMIT 25;
```

### 14. Top 25 Books in Other Languages
```
SELECT 
    bib.title AS Book_Title, 
    COUNT(i.itemnumber) AS Issued_Count,
    ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') AS Language
FROM issues i
JOIN items it ON i.itemnumber = it.itemnumber
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
WHERE ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') NOT IN ('Arabic', 'English', 'Lisan-ud-Dawat')
GROUP BY bib.title
ORDER BY Issued_Count DESC
LIMIT 25;
```
### 15. Report Name: Books Not Issued Ever

```
SELECT 
    bib.biblionumber AS Biblio_ID,
    bib.title AS Book_Title,
    bib.author AS Author,
    it.barcode AS Barcode,
    it.itemcallnumber AS Call_Number,
    it.replacementprice AS Price,
    it.itype AS Item_Type,  -- Replaced 'itemtype' with 'itype'
    it.location AS Location,
    ExtractValue(bmd.metadata, '//datafield[@tag="041"]/subfield[@code="a"]') AS Language,
    ExtractValue(bmd.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Subjects,
    it.dateaccessioned AS Date_Accessioned
FROM items it
JOIN biblio bib ON it.biblionumber = bib.biblionumber
JOIN biblio_metadata bmd ON bib.biblionumber = bmd.biblionumber
LEFT JOIN issues i ON it.itemnumber = i.itemnumber
WHERE i.itemnumber IS NULL
ORDER BY it.dateaccessioned DESC;

```





## 📌 **How to Use These Reports**
1. **Run the queries in Koha’s Reports module** to fetch live data.
2. **Export to CSV** for analysis or dashboard integration.
3. **Use SQL queries for dashboard visualization** in Power BI, Grafana, or Tableau.



## 💡 **More...**
These reports can be improved with a more customized data.
