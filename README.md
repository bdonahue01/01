WGU D326
Brianna Donahue
Student ID # 011873600
09/15/2024

D326 Business Report

A. Summarize one real-world written business report that can be created from the DVD Dataset from the “Labs on Demand Assessment Environment and DVD Database” attachment:
The business problem I chose to analyze was “Which store makes the most profit?” Analysis of this question can show which stores are renting out the most DVDs and allows us to allocate resources like funds and employees to those locations so they continue to thrive. We can use it for the opposite reason to see which locations need help so we can adjust marketing and help those locations do better in the future. 

Identify the specific fields that will be included in the detailed table and the summary table of the report.

The specific fields that will be included in the detailed table are the following:
store_id (INT), rental_id (INT), payment_id (INT), payment_amount (NUMERIC), formatted_payment_amount (TEXT),  rental_date (DATE)
The specific fields that will be included in the summary table of the report are the following:
store_id (INT), total_profit (NUMERIC)

Describe the types of data fields used for the report. 

The types of data fields being used for this report are INT, TEXT, DATE and NUMERIC.

Identify at least two specific tables from the given dataset that will provide the data necessary for the detailed table section and the summary table section of the report.

The specific tables from the given dataset that will provide the data necessary for the summary table will be aggregated from the detailed table. The specific tables from the given dataset that will provide the data necessary for the detailed table are store, payment and rental.

Identify at least one field in the detailed table section that will require a custom transformation with a user-defined function and explain why it should be transformed (e.g., you might translate a field with a value of N to No and Y to Yes).

One field in the detailed table section that will require custom transformation with a user defined function is the payment_amount field. It is being transformed to correctly sum the profit and aggregate total payments by store to calculate total profits as well as show it in a (e.g., $1.99) format for easy reading and to ensure that readers know which form of currency is being used for the numerics in the table. 

Explain the different business uses of the detailed table section and the summary table section of the report. 

The business use of the summary table is to get a quick high level overview of the data. It can be a quick way of seeing what the total sales are for each store location.
The business uses of the detailed table are to gain an even better insight into the data. This table includes insight into each individual payment transaction and its associated store. This allows us to see what was rented, the cost of the individual rental and date of the transaction. 

Explain how frequently your report should be refreshed to remain relevant to stakeholders.

This report should be updated on a monthly basis so that store locations can allocate its resources correctly (example: more workers at busier stores) and so that stockholders and higher ups in the company can see the profitability of the stores (more specifically which ones are remaining highly profitable). After enough data has been collected over a long enough time period this report could be used to predict annual trends and make long-term predictions on store profitability. 

*** BEGINNING OF CODE ***

B. Provide original code for function(s) in text format that perform the transformation(s) you identified in part A4.

CREATE OR REPLACE FUNCTION format_currency(amount NUMERIC)
RETURNS TEXT AS $$
BEGIN
RETURN '$' || TO_CHAR(amount, 'FM999,999,999.00');
END;
$$ LANGUAGE plpgsql;

C. Provide original SQL code in a text format that creates the detailed and summary tables to hold your report table sections.

DROP TABLE IF EXISTS detailed_table;
DROP TABLE IF EXISTS summary_table;

CREATE TABLE detailed_table (
store_id INT,
rental_id INT,
payment_id INT,
payment_amount NUMERIC,
formatted_payment_amount TEXT,
rental_date DATE,
PRIMARY KEY (store_id, rental_id, payment_id, payment_amount, formatted_payment_amount, rental_date)
);

CREATE TABLE summary_table (
store_id INT,
total_profit NUMERIC,
	PRIMARY KEY (store_id, total_profit)
);

SELECT * FROM detailed_table;
SELECT * FROM summary_table;

D. Provide an original SQL query in a text format that will extract the raw data needed for the detailed section of your report from the source database.

INSERT INTO detaield_table
SELECT 
s.store_id,
r.rental_id AS rentral_id,
p.payment_id AS payment_id,
p.amount AS payment_amount,
format_currency(SUM(p.amount)) AS formatted_payment_amount,
r.rental_date AS rental_date
FROM
payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN inventory i ON r.inventroy_id=i.inventory_id
JOIN store s ON i.store_id = s.store_id
GROUP BY
	s.store_id,
	r.rental_id,
	p.payment_id,
	p.amount,
	r.rental_date;

SELECT * FROM summary_table;
SELECT * FROM detailed_table;

E. Provide original SQL code in a text format that creates a trigger on the detailed table of the report that will continually update the summary table as data is added to the detailed table.

CREATE OR REPLACE FUNCTION insert_trigger_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS
$$
BEGIN
DELETE FROM rental_summary;
INSERT INTO rental_summary (category_id, amount)
    SELECT category_id, SUM(amount)
    FROM rental_detailed
    GROUP BY category_id;
RETURN NEW;
END;
$$;

CREATE TRIGGER update_summary
AFTER INSERT OR UPDATE ON rental_detailed
FOR EACH ROW
EXECUTE FUNCTION insert_trigger_function();

F. Provide an original stored procedure in a text format that can be used to refresh the data in both the detailed table and summary table. The procedure should clear the contents of the detailed table and summary table and perform the raw data extraction from part D.


CREATE OR REPLACE PROCEDURE refresh_tables()
LANGUAGE plpgsql
AS 
$$
BEGIN
	DELETE FROM detailed_table;
INSERT INTO detailed_table (store_id, rental_id, payment_id, payment_amount, formatted_payment_amount, rental_date)
SELECT 
s.store_id,
r.rental_id,
p.payment_id,
p.amount AS payment_amount,
format_currency(p.amount) AS formatted_payment_amount,
r.rental_date
FROM 
payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN inventory i ON r.inventroy_id=i.inventory_id
JOIN store s ON i.store_id = s.store_id;

DELETE FROM summary_table;
INSERT INTO summary_table (store_id, total_profit)
SELECT 
store_id,
           SUM(payment_amount) AS total_profit
FROM detailed_table
    GROUP BY detailed_table.store_id;
END;
$$;

CALL refresh_tables()


*** END OF CODE ***

1.  Identify a relevant job scheduling tool that can be used to automate the stored procedure.


I would instal cron jobs since this was done in PostgresSQL. I would have it execute the rerfresh_tables procedure daily at 11:59 pm every night so that the reports are up to date daily and fresh for the next work day.


G. Provide a Panopto video recording that includes the presenter and a vocalized demonstration of the functionality of the code used for the analysis.

https://wgu.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=2b4e6f87-de4b-4966-811b-b1ec01747a48#

(Sorry for the lack of enthusiasm, my vm kept stalling and I had to re-record this four times)

H. Acknowledge all utilized sources, including any sources of third-party code, using in-text citations and references. If no sources are used, clearly declare that no sources were used to support your submission.

I did not use any third-party sources for this business report.
