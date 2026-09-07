\# Hospital Pharmacy Inventory \& Stockout Engine



\## Executive Summary

This project models a relational database system for a hospital pharmacy network to track drug stock levels, predict stockout vulnerabilities, audit financial loss from near-expiry inventory, and evaluate supplier fulfillment delays. By combining core SQL joins, conditional logic (`CASE WHEN`), and aggregations, this engine translates raw operational data into actionable clinical inventory decisions.



\---



\## Database Architecture \& Schema

The relational database consists of four normalized tables (`medications`, `inventory`, `suppliers`, and `purchase\_orders`) linked via Foreign Key constraints.



\[ medications ]

&#x20;   /           \\

&#x20;  /             \\

\[ inventory ]     \[ purchase\_orders ]

|

\[ suppliers ]





\* \*\*`medications`\*\*: Master drug catalog storing medication names, clinical categories, and unit costs.

\* \*\*`inventory`\*\*: Real-time stock counts, minimum reorder thresholds, and expiration dates per item.

\* \*\*`suppliers`\*\*: Vendor registry tracking average fulfillment lead times.

\* \*\*`purchase\_orders`\*\*: Active order tracking system monitoring order status (`Pending`, `Delivered`, `Cancelled`).



\---



\## Key Business Insights \& Analytical Queries



\### 1. Inventory Priority \& Stockout Classification

Categorizes all inventory stock levels into actionable risk tiers (`CRITICAL LOW`, `REORDER NEEDED`, `HEALTHY`) based on baseline reorder thresholds.



```sql

SELECT 

&#x20;   m.drug\_name,

&#x20;   m.category,

&#x20;   i.stock\_on\_hand,

&#x20;   i.reorder\_point,

&#x20;   CASE 

&#x20;       WHEN i.stock\_on\_hand = 0 THEN 'OUT OF STOCK'

&#x20;       WHEN i.stock\_on\_hand <= (i.reorder\_point \* 0.25) THEN 'CRITICAL LOW'

&#x20;       WHEN i.stock\_on\_hand <= i.reorder\_point THEN 'REORDER NEEDED'

&#x20;       ELSE 'HEALTHY'

&#x20;   END AS stock\_status

FROM inventory i

JOIN medications m ON i.med\_id = m.med\_id

ORDER BY i.stock\_on\_hand ASC;

Key Finding: Atorvastatin 20mg is currently at a CRITICAL LOW state (5 units remaining vs. 30 unit reorder threshold), representing an immediate supply risk for the cardiovascular department.



2\. Expiration Risk \& Financial Loss Audit

Identifies pharmaceuticals expiring within 30 days and calculates total financial capital at risk.



SELECT 

&#x20;   m.drug\_name,

&#x20;   i.stock\_on\_hand,

&#x20;   m.unit\_cost,

&#x20;   (i.stock\_on\_hand \* m.unit\_cost) AS total\_value\_at\_risk,

&#x20;   i.expiration\_date

FROM inventory i

JOIN medications m ON i.med\_id = m.med\_id

WHERE i.expiration\_date <= '2026-10-01'

ORDER BY i.expiration\_date ASC;



Key Finding: Identified $1,025.00 in combined inventory value (including Lisinopril 10mg and Metformin 1000mg) expiring within 30 days.



Recommendation: Implement First-In, First-Out (FIFO) dispensing protocols immediately to reduce waste.



3\. Vendor Fulfillment Lead Time \& Supply Delay Analysis

Joins critical inventory items with pending purchase orders and vendor lead times to calculate expected delivery delays.



SQL

SELECT 

&#x20;   m.drug\_name,

&#x20;   i.stock\_on\_hand,

&#x20;   po.status AS order\_status,

&#x20;   s.supplier\_name,

&#x20;   s.lead\_time\_days

FROM inventory i

JOIN medications m ON i.med\_id = m.med\_id

JOIN purchase\_orders po ON m.med\_id = po.med\_id

JOIN suppliers s ON po.supplier\_id = s.supplier\_id

WHERE i.stock\_on\_hand <= i.reorder\_point 

&#x20; AND po.status = 'Pending';



Key Finding: Pending reorder for Atorvastatin 20mg with AmerisourceBergen carries a 5-day lead time, creating a projected 3-day stockout window before replenishment arrives.



Technical Stack

Language: SQL (ANSI SQL compliant)



Concepts Applied: Multi-table JOIN, CASE WHEN conditional statements, GROUP BY aggregations, Foreign Key constraints, Data Filtering (WHERE \& HAVING).



Version Control: Git \& GitHub Desktop.



