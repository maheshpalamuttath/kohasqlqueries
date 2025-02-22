# Koha SQL Queries Repository

This repository contains a collection of SQL queries designed for use with the Koha Integrated Library System (ILS). These queries are intended to help librarians and system administrators generate various reports and manage their library data efficiently. Below is a detailed list of the SQL queries available in this repository, organized into categories such as Catalog, Circulation, Patron, and more.

---

## Table of Contents

1. [Catalog](#catalog)
   - [Accession Register](#accession-register)
   - [AR Full](#ar-full)
   - [Accession Register with Keyword/Subject](#accession-register-with-keywordsubject)
   - [Accession Register (Joined Title and Subtitle, Authors and Editors)](#accession-register-joined-title-and-subtitle-authors-and-editors)
   - [Accession Register (Joined Title and Subtitle, Authors and Editors Separated Column)](#accession-register-joined-title-and-subtitle-authors-and-editors-separated-column)
   - [Report by Item Type](#report-by-item-type)
   - [Report by Collection](#report-by-collection)
   - [Report by Location](#report-by-location)
   - [Accession Register with Additional Fields](#accession-register-with-additional-fields)
   - [New Arrivals](#new-arrivals)
   - [Date-wise Catalogued Books](#date-wise-catalogued-books)
   - [Report for Creating Spine Labels](#report-for-creating-spine-labels)
   - [Unique Titles](#unique-titles)
   - [Accession Register Report by Branch](#accession-register-report-by-branch)
   - [Item Added Date-wise](#item-added-date-wise)
   - [Custom Barcode List](#custom-barcode-list)

2. [Circulation](#circulation)
   - [Total Checked Out (Issued) Books](#total-checked-out-issued-books)
   - [Date-wise List of Checked Out (Issued) Books](#date-wise-list-of-checked-out-issued-books)
   - [Date-wise List of Checked In (Returned) Books](#date-wise-list-of-checked-in-returned-books)
   - [Date-wise List of Checked Out (Issued) Books Branchwise](#date-wise-list-of-checked-out-issued-books-branchwise)
   - [Date-wise List of Checked In (Return) Books Branchwise](#date-wise-list-of-checked-in-return-books-branchwise)
   - [All Circulation Transactions on Date Range with Patron & Item Details](#all-circulation-transactions-on-date-range-with-patron--item-details)
   - [Overdue List](#overdue-list)
   - [Patrons with Books Due Tomorrow](#patrons-with-books-due-tomorrow)
   - [Patron with Fine](#patron-with-fine)

3. [Patron](#patron)
   - [Patron List by Category](#patron-list-by-category)
   - [Patron without Image](#patron-without-image)

4. [Miscellaneous](#miscellaneous)
   - [Total Bibs & Items by Location](#total-bibs--items-by-location)
   - [SQL Query for Exporting Complete Data from a Database via PHPMyAdmin](#sql-query-for-exporting-complete-data-from-a-database-via-phpmyadmin)

---

## Catalog

### Accession Register

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
ORDER BY i.barcode ASC;
```

### AR Full

```sql
SELECT bi.isbn, b.author, b.title, bi.editionstatement, bi.place, bi.publishercode, b.copyrightdate, bi.pages, i.itype, i.homebranch, i.holdingbranch, i.ccode, i.location, i.price, i.itemcallnumber, i.barcode, i.itype 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber;
```

### Accession Register with Keyword/Subject

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, b.author, b.title, ExtractValue(bm.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Keyword, bi.pages, bi.publishercode, bi.place, b.copyrightdate
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
JOIN biblio_metadata bm ON bi.biblionumber = bm.biblionumber
ORDER BY LPAD(i.barcode, 40, ' ') ASC;
```

### Accession Register (Joined Title and Subtitle, Authors and Editors)

```sql
SELECT bi.isbn, ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS DDC, ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS itemnumber, CONCAT_WS('', b.author, '; ', ExtractValue(bm.metadata, '//datafield[@tag="700"]/subfield[@code="a"]')) AS Author, CONCAT(b.title, ' ', ExtractValue(bm.metadata, '//datafield[@tag="245"]/subfield[@code="b"]')) AS Title, bi.editionstatement, bi.place, bi.publishercode, b.copyrightdate, bi.pages, ExtractValue(bm.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS keywords, i.itype, i.homebranch, i.holdingbranch, i.ccode, i.location, i.price, i.itemcallnumber, i.barcode, i.itype 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
LEFT JOIN biblio_metadata bm ON bm.biblionumber = b.biblionumber
WHERE i.homebranch = <<Branch|branches>> 
ORDER BY i.barcode ASC;
```

### Accession Register (Joined Title and Subtitle, Authors and Editors Separated Column)

```sql
SELECT bi.isbn, ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS DDC, ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS itemnumber, b.author, CONCAT(b.title, ' ', ExtractValue(bm.metadata, '//datafield[@tag="245"]/subfield[@code="b"]')) AS Title, bi.editionstatement, bi.place, bi.publishercode, b.copyrightdate, bi.pages, ExtractValue(bm.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS keywords, ExtractValue(bm.metadata, '//datafield[@tag="700"]/subfield[@code="a"]') AS Author2, i.itype, i.homebranch, i.holdingbranch, i.ccode, i.location, i.price, i.itemcallnumber, i.barcode, i.itype 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
LEFT JOIN biblio_metadata bm ON bm.biblionumber = b.biblionumber
WHERE i.homebranch = <<Branch|branches>> 
ORDER BY i.barcode ASC;
```

### Report by Item Type

```sql
SELECT COALESCE(i.homebranch, '*GRAND TOTAL*') AS homebranch, IFNULL(i.itype, "") AS itype, COUNT(i.itype) AS count
FROM items i
WHERE i.dateaccessioned < <<Added before (yyyy-mm-dd)|date>>
GROUP BY i.homebranch, i.itype
WITH ROLLUP;
```

### Report by Collection

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber 
WHERE i.homebranch = <<Choose library|branches>> 
AND i.ccode LIKE <<Choose Section/Collection |CCODE>>;
```

### Report by Location

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber 
WHERE i.homebranch = <<Choose library|branches>> 
AND i.location LIKE <<Choose location|LOC>>;
```

### Accession Register with Additional Fields

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate, ccode_auth.lib AS department, loc_auth.lib AS location, subj_auth.lib AS subject, notes_auth.lib AS recieved_as
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
LEFT JOIN authorised_values AS ccode_auth ON i.ccode = ccode_auth.authorised_value AND ccode_auth.category = 'CCODE'
LEFT JOIN authorised_values AS loc_auth ON i.location = loc_auth.authorised_value AND loc_auth.category = 'LOC'
LEFT JOIN authorised_values AS subj_auth ON i.enumchron = subj_auth.authorised_value AND subj_auth.category = 'SUBJECTS'
LEFT JOIN authorised_values AS notes_auth ON i.itemnotes = notes_auth.authorised_value AND notes_auth.category = 'BOOKRECIEVEDAS'
ORDER BY i.barcode ASC;
```

### New Arrivals

```sql
SELECT b.biblionumber, SUBSTRING_INDEX(m.isbn, ' ', 1) AS isbn, b.title
FROM items i
LEFT JOIN biblioitems m USING (biblioitemnumber)
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
WHERE DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= i.dateaccessioned 
AND m.isbn IS NOT NULL 
AND m.isbn != ''
GROUP BY b.biblionumber
HAVING isbn != ""
ORDER BY RAND()
LIMIT 30;
```

### Date-wise Catalogued Books

```sql
SELECT i.dateaccessioned, i.barcode, i.itemcallnumber, b.author, b.title, bi.publishercode 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber 
WHERE i.dateaccessioned BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>
ORDER BY i.barcode DESC;
```

### Report for Creating Spine Labels

```sql
SELECT ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS ClassNo, ExtractValue(bm.metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS BookNo, i.barcode
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
JOIN biblio_metadata bm ON bi.biblionumber = bm.biblionumber
WHERE i.dateaccessioned;
```

### Unique Titles

```sql
SELECT i.homebranch, COUNT(DISTINCT i.biblionumber) AS bibs, COUNT(i.itemnumber) AS items
FROM items i
GROUP BY i.homebranch
ORDER BY i.homebranch ASC;
```

### Accession Register Report by Branch

```sql
SELECT i.barcode, i.itemcallnumber, i.itype, i.ccode, i.location, bi.isbn, b.author, b.title, b.subtitle, bi.editionstatement, bi.place, bi.publishercode, b.copyrightdate, bi.pages, i.price, i.enumchron, i.dateaccessioned 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber 
WHERE i.homebranch = <<Choose library|branches>>;
```

### Item Added Date-wise

```sql
SELECT i.dateaccessioned, b.title, i.barcode, i.itemcallnumber
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
WHERE DATE(i.dateaccessioned) BETWEEN <<BETWEEN (yyyy-mm-dd)|date>> AND <<AND (yyyy-mm-dd)|date>>
AND i.homebranch = <<Home branch|branches>>
ORDER BY i.itemcallnumber ASC;
```

### Custom Barcode List

```sql
SELECT i.dateaccessioned, i.barcode, b.title, b.subtitle, b.author, bi.editionstatement, bi.publishercode, b.copyrightdate, i.price
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
WHERE i.barcode IN (26450, 26451, 26452, 26453, 26454, 26455, 26456, 26457, 26458, 26459, 26460, 26461, 26462, 26463, 26464, 26465, 27613, 27614)
ORDER BY i.barcode;
```

---

## Circulation

### Total Checked Out (Issued) Books

```sql
SELECT i.barcode, c.date_due, p.surname, p.firstname, p.cardnumber, p.phone, p.email, b.title, b.author, i.itemcallnumber, i.location
FROM issues c
LEFT JOIN items i ON c.itemnumber = i.itemnumber
LEFT JOIN borrowers p ON c.borrowernumber = p.borrowernumber
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
ORDER BY c.date_due ASC;
```

### Date-wise List of Checked Out (Issued) Books

```sql
SELECT DATE_FORMAT(c.issuedate, "%d %b %Y %h:%i %p") AS Issue_Date, DATE_FORMAT(c.date_due, "%d %b %Y") AS Due_Date, i.barcode AS Barcode, b.title AS Title, b.author AS Author, p.cardnumber AS Card_No, p.firstname AS First_Name, p.surname AS Last_Name
FROM issues c
LEFT JOIN items i ON c.itemnumber = i.itemnumber
LEFT JOIN borrowers p ON c.borrowernumber = p.borrowernumber
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
WHERE c.issuedate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY c.issuedate DESC;
```

### Date-wise List of Checked In (Returned) Books

```sql
SELECT oi.returndate, i.barcode, b.title, b.author, p.firstname, p.surname, p.cardnumber, p.categorycode
FROM old_issues oi  
LEFT JOIN borrowers p ON oi.borrowernumber = p.borrowernumber
LEFT JOIN items i ON oi.itemnumber = i.itemnumber 
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber 
WHERE oi.returndate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY oi.returndate DESC;
```

### Date-wise List of Checked Out (Issued) Books Branchwise

```sql
SELECT c.issuedate, i.barcode, i.itemcallnumber, i.homebranch, p.title, p.firstname, p.surname, b.title, b.author
FROM issues c
LEFT JOIN borrowers p ON c.borrowernumber = p.borrowernumber
LEFT JOIN items i ON c.itemnumber = i.itemnumber
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
WHERE i.homebranch = <<Branch|branches>>  
AND DATE(c.issuedate) BETWEEN <<Checked out BETWEEN (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>
ORDER BY c.issuedate DESC;
```

### Date-wise List of Checked In (Return) Books Branchwise

```sql
SELECT oi.returndate, i.barcode, i.itemcallnumber, i.homebranch, p.title, p.firstname, p.surname, b.title, b.author
FROM old_issues oi  
LEFT JOIN borrowers p ON oi.borrowernumber = p.borrowernumber
LEFT JOIN items i ON oi.itemnumber = i.itemnumber
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
WHERE i.homebranch = <<Branch|branches>>  
AND oi.returndate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY oi.returndate DESC;
```

### All Circulation Transactions on Date Range with Patron & Item Details

```sql
SELECT s.datetime AS "Date", p.cardnumber AS "Card number", p.surname AS "Last name", 
CASE s.type 
WHEN 'issue' THEN "Check out" 
WHEN 'localuse' THEN "In house use" 
WHEN 'return' THEN "Check in" 
WHEN 'renew' THEN "Renew" 
WHEN 'writeoff' THEN "Amnesty" 
WHEN 'payment' THEN "Payment" 
ELSE "Other" END AS "Transaction", 
CASE s.value 
WHEN '0' THEN "-" 
ELSE s.value END AS "Amount", 
i.barcode AS "Barcode", 
b.title AS "Title", 
b.author AS "Author", 
i.homebranch, 
i.holdingbranch 
FROM statistics s
JOIN borrowers p ON s.borrowernumber = p.borrowernumber 
LEFT JOIN items i ON s.itemnumber = i.itemnumber 
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber 
WHERE DATE(s.datetime) BETWEEN <<From Date|date>> AND <<To Date|date>>;
```

### Overdue List

```sql
SELECT p.surname, p.firstname, c.date_due, (TO_DAYS(CURDATE()) - TO_DAYS(c.date_due)) AS 'days overdue', i.itemcallnumber, i.barcode, b.title, b.author  
FROM borrowers p 
LEFT JOIN issues c ON p.borrowernumber = c.borrowernumber  
LEFT JOIN items i ON c.itemnumber = i.itemnumber  
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber  
WHERE (TO_DAYS(CURDATE()) - TO_DAYS(c.date_due)) > '30'   
ORDER BY p.surname ASC, c.date_due ASC;
```

### Patrons with Books Due Tomorrow

```sql
SELECT p.cardnumber, p.surname, p.branchcode, p.firstname, c.date_due, i.barcode, b.title, b.author
FROM borrowers p
LEFT JOIN issues c ON c.borrowernumber = p.borrowernumber
LEFT JOIN items i ON c.itemnumber = i.itemnumber
LEFT JOIN biblio b ON i.biblionumber = b.biblionumber
WHERE DATE(c.date_due) = DATE_ADD(CURDATE(), INTERVAL 1 DAY)
AND i.homebranch = <<Branch|branches>>
ORDER BY p.surname ASC;
```

### Patron with Fine

```sql
SELECT (SELECT CONCAT('<a href=\"/cgi-bin/koha/members/boraccount.pl?borrowernumber=', b.borrowernumber, '\">', b.surname, ', ', b.firstname, '</a>') 
FROM borrowers b WHERE b.borrowernumber = a.borrowernumber) AS Patron, 
FORMAT(SUM(a.amountoutstanding), 2) AS 'Outstanding', 
(SELECT COUNT(i.itemnumber) FROM issues i WHERE b.borrowernumber = i.borrowernumber) AS 'Checkouts' 
FROM accountlines a, borrowers b 
WHERE (SELECT SUM(a2.amountoutstanding) FROM accountlines a2 WHERE a2.borrowernumber = a.borrowernumber) > '0.00' 
AND a.borrowernumber = b.borrowernumber 
GROUP BY a.borrowernumber 
ORDER BY b.surname, b.firstname, Outstanding ASC;
```

---

## Patron

### Patron List by Category

```sql
SELECT p.cardnumber, p.surname, p.address, p.sex, p.mobile, p.email, p.sort1, p.sort2, p.dateenrolled, p.dateexpiry 
FROM borrowers p 
WHERE p.branchcode = <<Enter patrons library|branches>> 
AND p.categorycode LIKE <<Enter Category borrowers|categorycode>>;
```

### Patron without Image

```sql
SELECT p.cardnumber, p.borrowernumber, p.surname, p.firstname 
FROM borrowers p 
WHERE p.borrowernumber 
NOT IN (SELECT pi.borrowernumber FROM patronimage pi);
```

---

## Miscellaneous

### Total Bibs & Items by Location

```sql
SELECT i.homebranch, i.location, COUNT(DISTINCT i.biblionumber) AS bibs, COUNT(i.itemnumber) AS items
FROM items i
WHERE i.homebranch = <<Choose library|branches>>  
AND i.location = <<Choose location|LOC>>
GROUP BY i.homebranch, i.location
ORDER BY i.homebranch ASC, i.location ASC;
```

### SQL Query for Exporting Complete Data from a Database via PHPMyAdmin

1. Create a database with `koha_test` (or any name you prefer).
2. Restore the old database.
3. Go to PHPMyAdmin or any MySQL/MariaDB client application, select `koha_test`, and execute the following query:

```sql
SELECT bi.biblioitemnumber, bi.isbn, b.author, b.title, bi.editionstatement, bi.place, bi.publishercode, b.copyrightdate, bi.pages, i.itype, i.ccode, i.homebranch, i.holdingbranch, i.location, i.dateaccessioned, i.price, i.itemcallnumber, i.barcode, i.itype 
FROM items i 
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber 
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber;
```

4. Export the result as a CSV file.

---

This repository is designed to assist Koha users in managing their library data efficiently. Feel free to contribute or suggest improvements!
