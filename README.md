# Koha SQL Queries Repository

This repository contains a collection of SQL queries designed for use with the Koha Integrated Library System (ILS). These queries are intended to help librarians and system administrators generate various reports and manage their library data efficiently. Below is a detailed list of the SQL queries available in this repository.

## Table of Contents

1. [Accession Register](#accession-register)
2. [AR Full](#ar-full)
3. [Accession Register with Keyword/Subject](#accession-register-with-keywordsubject)
4. [Accession Register (Joined Title and Subtitle, Authors and Editors)](#accession-register-joined-title-and-subtitle-authors-and-editors)
5. [Accession Register (Joined Title and Subtitle, Authors and Editors Separated Column)](#accession-register-joined-title-and-subtitle-authors-and-editors-separated-column)
6. [Report by Item Type](#report-by-item-type)
7. [Report by Collection](#report-by-collection)
8. [Report by Location](#report-by-location)
9. [Additional Authorised Values and Append with Framework and SQL Query](#additional-authorised-values-and-append-with-framework-and-sql-query)
10. [Accession Register with Additional Fields](#accession-register-with-additional-fields)
11. [New Arrivals](#new-arrivals)
12. [Date-wise Catalogued Books](#date-wise-catalogued-books)
13. [Report for Creating Spine Labels](#report-for-creating-spine-labels)
14. [Unique Titles](#unique-titles)
15. [Accession Register Report by Branch](#accession-register-report-by-branch)
16. [Item Added Date-wise](#item-added-date-wise)
17. [Custom Barcode List](#custom-barcode-list)
18. [Circulation](#circulation)
19. [Patron](#patron)
20. [Total Bibs & Items by Location](#total-bibs--items-by-location)
21. [SQL Query for Exporting Complete Data from a Database via PHPMyAdmin](#sql-query-for-exporting-complete-data-from-a-database-via-phpmyadmin)

## Accession Register

```sql
SELECT items.barcode, items.dateaccessioned, items.itemcallnumber, biblioitems.isbn, biblio.author, biblio.title, biblioitems.pages, biblioitems.publishercode, biblioitems.place, biblio.copyrightdate
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
ORDER BY items.barcode ASC;
```

## AR Full

```sql
SELECT biblioitems.isbn, biblio.author, biblio.title, biblioitems.editionstatement, biblioitems.place, biblioitems.publishercode, biblio.copyrightdate, biblioitems.pages, items.itype, items.homebranch, items.holdingbranch, items.ccode, items.location, items.price, items.itemcallnumber, items.barcode, items.itype 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber);
```

## Accession Register with Keyword/Subject

```sql
SELECT items.barcode, items.dateaccessioned, items.itemcallnumber, biblio.author, biblio.title, ExtractValue(metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Keyword, biblioitems.pages, biblioitems.publishercode, biblioitems.place, biblio.copyrightdate
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
JOIN biblio_metadata ON (biblioitems.biblionumber = biblio_metadata.biblionumber)
ORDER BY LPAD(items.barcode, 40, ' ') ASC;
```

## Accession Register (Joined Title and Subtitle, Authors and Editors)

```sql
SELECT biblioitems.isbn, ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS DDC, ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS itemnumber, CONCAT_WS('', biblio.author, '; ', ExtractValue(metadata, '//datafield[@tag="700"]/subfield[@code="a"]')) AS Author, CONCAT(biblio.title, ' ', ExtractValue(metadata, '//datafield[@tag="245"]/subfield[@code="b"]')) AS Title, biblioitems.editionstatement, biblioitems.place, biblioitems.publishercode, biblio.copyrightdate, biblioitems.pages, ExtractValue(metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS keywords, items.itype, items.homebranch, items.holdingbranch, items.ccode, items.location, items.price, items.itemcallnumber, items.barcode, items.itype 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
LEFT JOIN biblio_metadata ON (biblio_metadata.biblionumber = biblio.biblionumber)
WHERE items.homebranch = <<Branch|branches>> 
ORDER BY items.barcode ASC;
```

## Accession Register (Joined Title and Subtitle, Authors and Editors Separated Column)

```sql
SELECT biblioitems.isbn, ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS DDC, ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS itemnumber, biblio.author, CONCAT(biblio.title, ' ', ExtractValue(metadata, '//datafield[@tag="245"]/subfield[@code="b"]')) AS Title, biblioitems.editionstatement, biblioitems.place, biblioitems.publishercode, biblio.copyrightdate, biblioitems.pages, ExtractValue(metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS keywords, ExtractValue(metadata, '//datafield[@tag="700"]/subfield[@code="a"]') AS Author2, items.itype, items.homebranch, items.holdingbranch, items.ccode, items.location, items.price, items.itemcallnumber, items.barcode, items.itype 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
LEFT JOIN biblio_metadata ON (biblio_metadata.biblionumber = biblio.biblionumber)
WHERE items.homebranch = <<Branch|branches>> 
ORDER BY items.barcode ASC;
```

## Report by Item Type

```sql
SELECT COALESCE(homebranch, '*GRAND TOTAL*') AS homebranch, IFNULL(itype, "") AS itype, count(itype) AS count
FROM items
WHERE dateaccessioned < <<Added before (yyyy-mm-dd)|date>>
GROUP BY homebranch, itype
WITH ROLLUP;
```

## Report by Collection

```sql
SELECT items.barcode, items.dateaccessioned, items.itemcallnumber, biblioitems.isbn, biblio.author, biblio.title, biblioitems.pages, biblioitems.publishercode, biblioitems.place, biblio.copyrightdate 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber) 
WHERE items.homebranch = <<Choose library|branches>> 
AND items.ccode LIKE <<Choose Section/Collection |CCODE>>;
```

## Report by Location

```sql
SELECT items.barcode, items.dateaccessioned, items.itemcallnumber, biblioitems.isbn, biblio.author, biblio.title, biblioitems.pages, biblioitems.publishercode, biblioitems.place, biblio.copyrightdate 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber) 
WHERE items.homebranch = <<Choose library|branches>> 
AND items.location LIKE <<Choose location|LOC>>;
```

## Additional Authorised Values and Append with Framework and SQL Query

### Tag 952$z

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate, i.itemnotes 
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
WHERE i.homebranch = <<Choose library|branches>> 
AND i.itemnotes LIKE <<Book receive type |BOOKSRECEIVEDAS>>;
```

### Tag 952$5

```sql
SELECT i.barcode, i.dateaccessioned, i.itemcallnumber, bi.isbn, b.author, b.title, bi.pages, bi.publishercode, bi.place, b.copyrightdate, COALESCE(av.lib, 'Unknown') AS subject
FROM items i
LEFT JOIN biblioitems bi ON i.biblioitemnumber = bi.biblioitemnumber
LEFT JOIN biblio b ON bi.biblionumber = b.biblionumber
LEFT JOIN authorised_values av ON i.restricted = av.authorised_value AND av.category = 'SUBJECTS'
WHERE i.homebranch = <<Choose library|branches>> 
AND i.restricted LIKE <<Choose Subject |SUBJECTS>>;
```

## Accession Register with Additional Fields

```sql
SELECT items.barcode, items.dateaccessioned, items.itemcallnumber, biblioitems.isbn, biblio.author, biblio.title, biblioitems.pages, biblioitems.publishercode, biblioitems.place, biblio.copyrightdate, ccode_auth.lib AS department, loc_auth.lib AS location, subj_auth.lib AS subject, notes_auth.lib AS recieved_as
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
LEFT JOIN authorised_values AS ccode_auth ON (items.ccode = ccode_auth.authorised_value AND ccode_auth.category = 'CCODE')
LEFT JOIN authorised_values AS loc_auth ON (items.location = loc_auth.authorised_value AND loc_auth.category = 'LOC')
LEFT JOIN authorised_values AS subj_auth ON (items.enumchron = subj_auth.authorised_value AND subj_auth.category = 'SUBJECTS')
LEFT JOIN authorised_values AS notes_auth ON (items.itemnotes = notes_auth.authorised_value AND notes_auth.category = 'BOOKRECIEVEDAS')
ORDER BY items.barcode ASC;
```

## New Arrivals

```sql
SELECT b.biblionumber, SUBSTRING_INDEX(m.isbn, ' ', 1) AS isbn, b.title
FROM items i
LEFT JOIN biblioitems m USING (biblioitemnumber)
LEFT JOIN biblio b ON (i.biblionumber = b.biblionumber)
WHERE DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= i.dateaccessioned 
AND m.isbn IS NOT NULL 
AND m.isbn != ''
GROUP BY biblionumber
HAVING isbn != ""
ORDER BY rand()
LIMIT 30;
```

## Date-wise Catalogued Books

```sql
SELECT items.dateaccessioned, items.barcode, items.itemcallnumber, biblio.author, biblio.title, biblioitems.publishercode 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber) 
WHERE items.dateaccessioned BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>
ORDER BY items.barcode DESC;
```

## Report for Creating Spine Labels

```sql
SELECT ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="a"]') AS ClassNo, ExtractValue(metadata, '//datafield[@tag="082"]/subfield[@code="b"]') AS BookNo, items.barcode
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
JOIN biblio_metadata ON (biblioitems.biblionumber = biblio_metadata.biblionumber)
WHERE items.dateaccessioned;
```

## Unique Titles

```sql
SELECT homebranch, count(DISTINCT biblionumber) AS bibs, count(itemnumber) AS items
FROM items
GROUP BY homebranch
ORDER BY homebranch ASC;
```

## Accession Register Report by Branch

```sql
SELECT items.barcode, items.itemcallnumber, items.itype, items.ccode, items.location, biblioitems.isbn, biblio.author, biblio.title, biblio.subtitle, biblioitems.editionstatement, biblioitems.place, biblioitems.publishercode, biblio.copyrightdate, biblioitems.pages, items.price, items.enumchron, items.dateaccessioned 
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber) 
WHERE items.homebranch = <<Choose library|branches>>;
```

## Item Added Date-wise

```sql
SELECT items.dateaccessioned, biblio.title, items.barcode, items.itemcallnumber
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
WHERE DATE(items.dateaccessioned) BETWEEN <<BETWEEN (yyyy-mm-dd)|date>> AND <<AND (yyyy-mm-dd)|date>>
AND items.homebranch = <<Home branch|branches>>
ORDER BY items.itemcallnumber ASC;
```

## Custom Barcode List

```sql
SELECT items.dateaccessioned, items.barcode, biblio.title, biblio.subtitle, biblio.author, biblioitems.editionstatement, biblioitems.publishercode, biblio.copyrightdate, items.price
FROM items
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber)
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber)
WHERE items.barcode IN (26450, 26451, 26452, 26453, 26454, 26455, 26456, 26457, 26458, 26459, 26460, 26461, 26462, 26463, 26464, 26465, 27613, 27614)
ORDER BY items.barcode;
```

## Circulation

### Total Checked Out (Issued) Book

```sql
SELECT i.barcode, c.date_due, p.surname, p.firstname, p.cardnumber, p.phone, p.email, b.title, b.author, i.itemcallnumber, i.location
FROM issues c
LEFT JOIN items i ON (c.itemnumber = i.itemnumber)
LEFT JOIN borrowers p ON (c.borrowernumber = p.borrowernumber)
LEFT JOIN biblio b ON (i.biblionumber = b.biblionumber)
ORDER BY c.date_due ASC;
```

### Date-wise List of Checked Out (Issued) Books

```sql
SELECT DATE_FORMAT(c.issuedate, "%d %b %Y %h:%i %p") AS Issue_Date, DATE_FORMAT(c.date_due, "%d %b %Y") AS Due_Date, i.barcode AS Barcode, b.title AS Title, b.author AS Author, p.cardnumber AS Card_No, p.firstname AS First_Name, p.surname AS Last_Name
FROM issues c
LEFT JOIN items i ON (c.itemnumber = i.itemnumber)
LEFT JOIN borrowers p ON (c.borrowernumber = p.borrowernumber)
LEFT JOIN biblio b ON (i.biblionumber = b.biblionumber)
WHERE c.issuedate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY c.issuedate DESC;
```

### Date-wise List of Checked In (Returned) Books

```sql
SELECT old_issues.returndate, items.barcode, biblio.title, biblio.author, borrowers.firstname, borrowers.surname, borrowers.cardnumber, borrowers.categorycode
FROM old_issues  
LEFT JOIN borrowers ON borrowers.borrowernumber = old_issues.borrowernumber
LEFT JOIN items ON old_issues.itemnumber = items.itemnumber 
LEFT JOIN biblio ON items.biblionumber = biblio.biblionumber 
WHERE old_issues.returndate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY old_issues.returndate DESC;
```

### Date-wise List of Checked Out (Issued) Books Branchwise

```sql
SELECT issues.issuedate, items.barcode, items.itemcallnumber, items.homebranch, borrowers.title, borrowers.firstname, borrowers.surname, biblio.title, biblio.author
FROM issues
LEFT JOIN borrowers ON borrowers.borrowernumber = issues.borrowernumber
LEFT JOIN items ON issues.itemnumber = items.itemnumber
LEFT JOIN biblio ON items.biblionumber = biblio.biblionumber
WHERE items.homebranch = <<Branch|branches>>  
AND date(issuedate) BETWEEN <<Checked out BETWEEN (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>
ORDER BY issues.issuedate DESC;
```

### Date-wise List of Checked In (Return) Books Branchwise

```sql
SELECT old_issues.returndate, items.barcode, items.itemcallnumber, items.homebranch, borrowers.title, borrowers.firstname, borrowers.surname, biblio.title, biblio.author
FROM old_issues  
LEFT JOIN borrowers ON borrowers.borrowernumber = old_issues.borrowernumber
LEFT JOIN items ON old_issues.itemnumber = items.itemnumber
LEFT JOIN biblio ON items.biblionumber = biblio.biblionumber
WHERE items.homebranch = <<Branch|branches>>  
AND old_issues.returndate BETWEEN <<Between Date (yyyy-mm-dd)|date>> AND <<and (yyyy-mm-dd)|date>>  
ORDER BY old_issues.returndate DESC;
```

### All Circulation Transactions on Date Range with Patron & Item Details

```sql
SELECT datetime AS "Date", cardnumber AS "Card number", surname AS "Last name", 
CASE type 
WHEN 'issue' THEN "Check out" 
WHEN 'localuse' THEN "In house use" 
WHEN 'return' THEN "Check in" 
WHEN 'renew' THEN "Renew" 
WHEN 'writeoff' THEN "Amnesty" 
WHEN 'payment' THEN "Payment" 
ELSE "Other" END AS "Transaction", 
CASE value 
WHEN '0' THEN "-" 
ELSE value END AS "Amount", 
barcode AS "Barcode", 
biblio.title AS "Title", 
author AS "Author", 
items.homebranch, 
items.holdingbranch 
FROM statistics 
JOIN borrowers ON statistics.borrowernumber = borrowers.borrowernumber 
LEFT JOIN items ON statistics.itemnumber = items.itemnumber 
LEFT JOIN biblio ON items.biblionumber = biblio.biblionumber 
WHERE DATE(statistics.datetime) BETWEEN <<From Date|date>> AND <<To Date|date>>;
```

### Overdue List

```sql
SELECT borrowers.surname, borrowers.firstname, issues.date_due, (TO_DAYS(curdate()) - TO_DAYS(date_due)) AS 'days overdue', items.itemcallnumber, items.barcode, biblio.title, biblio.author  
FROM borrowers 
LEFT JOIN issues ON (borrowers.borrowernumber = issues.borrowernumber)  
LEFT JOIN items ON (issues.itemnumber = items.itemnumber)  
LEFT JOIN biblio ON (items.biblionumber = biblio.biblionumber)  
WHERE (TO_DAYS(curdate()) - TO_DAYS(date_due)) > '30'   
ORDER BY borrowers.surname ASC, issues.date_due ASC;
```

### Patrons with Books Due Tomorrow

```sql
SELECT p.cardnumber, p.surname, p.branchcode, p.firstname, co.date_due, i.barcode, b.title, b.author
FROM borrowers p
LEFT JOIN issues co ON (co.borrowernumber = p.borrowernumber)
LEFT JOIN items i ON (co.itemnumber = i.itemnumber)
LEFT JOIN biblio b ON (b.biblionumber = i.biblionumber)
WHERE DATE(co.date_due) = DATE_ADD(curdate(), INTERVAL 1 DAY)
AND i.homebranch = <<Branch|branches>>
ORDER BY p.surname ASC;
```

### Patron with Fine

```sql
SELECT (SELECT CONCAT('<a href=\"/cgi-bin/koha/members/boraccount.pl?borrowernumber=', b.borrowernumber, '\">', b.surname, ', ', b.firstname, '</a>') 
FROM borrowers b WHERE b.borrowernumber = a.borrowernumber) AS Patron, 
format(sum(amountoutstanding), 2) AS 'Outstanding', 
(SELECT count(i.itemnumber) FROM issues i WHERE b.borrowernumber = i.borrowernumber) AS 'Checkouts' 
FROM accountlines a, borrowers b 
WHERE (SELECT sum(amountoutstanding) FROM accountlines a2 WHERE a2.borrowernumber = a.borrowernumber) > '0.00' 
AND a.borrowernumber = b.borrowernumber 
GROUP BY a.borrowernumber 
ORDER BY b.surname, b.firstname, Outstanding ASC;
```

## Patron

### Patron List by Category

```sql
SELECT borrowers.cardnumber, borrowers.surname, borrowers.address, borrowers.sex, borrowers.mobile, borrowers.email, borrowers.sort1, borrowers.sort2, borrowers.dateenrolled, borrowers.dateexpiry 
FROM borrowers 
WHERE branchcode = <<Enter patrons library|branches>> 
AND categorycode LIKE <<Enter Category borrowers|categorycode>>;
```

### Patron without Image

```sql
SELECT cardnumber, borrowernumber, surname, firstname 
FROM borrowers 
WHERE borrowernumber 
NOT IN (SELECT borrowernumber FROM patronimage);
```

## Total Bibs & Items by Location

```sql
SELECT homebranch, location, count(DISTINCT biblionumber) AS bibs, count(itemnumber) AS items
FROM items
WHERE homebranch = <<Choose library|branches>>  
AND location = <<Choose location|LOC>>
GROUP BY homebranch, location
ORDER BY homebranch ASC, location ASC;
```

## SQL Query for Exporting Complete Data from a Database via PHPMyAdmin

1. Create a database with `koha_test` (or any name you prefer).
2. Restore the old database.
3. Go to PHPMyAdmin or any MySQL/MariaDB client application, select `koha_test`, and execute the following query:

```sql
SELECT biblioitems.biblioitemnumber, biblioitems.isbn, biblio.author, biblio.title, biblioitems.editionstatement, biblioitems.place, biblioitems.publishercode, biblio.copyrightdate, biblioitems.pages, items.itype, items.ccode, items.homebranch, items.holdingbranch, items.location, items.dateaccessioned, items.price, items.itemcallnumber, items.barcode, items.itype 
FROM items 
LEFT JOIN biblioitems ON (items.biblioitemnumber = biblioitems.biblioitemnumber) 
LEFT JOIN biblio ON (biblioitems.biblionumber = biblio.biblionumber);
```

4. Export the result as a CSV file.

---

This repository is designed to assist Koha users in managing their library data efficiently. Feel free to contribute or suggest improvements!
