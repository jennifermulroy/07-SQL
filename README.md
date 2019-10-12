# 07-SQL

## Data Analysis

#### How can you isolate (or group) the transactions of each cardholder?

SELECT card_number, COUNT(card_number) as "transactions per card_number"
FROM transactions
GROUP BY card_number; 

#### Consider the time period 7:00 a.m. to 9:00 a.m. What are the top 100 highest transactions during this time period? Do you see any fraudulent or anomalous transactions? If you answered yes to the previous question, explain why you think there might be fraudulent transactions during this time frame.

SELECT date_time, Amount as "largest transactions from 7-9am"
FROM transactions
WHERE CAST(date_time as time) >= '07:00:00' 
   and CAST(date_time as time) <= '09:00:00'
ORDER by amount DESC
LIMIT 100;

SELECT date_time, amount as " average transaction from 7-9am"
FROM transactions
WHERE CAST(date_time as time) >= '07:00:00' 
   and CAST(date_time as time) <= '09:00:00'
ORDER by amount DESC
LIMIT 100;

The average transaction size is $45.19, but the first 8 largest transactions during 7-9am average $1,226.25. These transactions appear to significant outliers in the data and therefore, are potentally fraudulent transactions. 

#### Some fraudsters hack a credit card by making several small payments (generally less than $2.00), which are typically ignored by cardholders. Count the transactions that are less than $2.00 per cardholder. Is there any evidence to suggest that a credit card has been hacked? Explain your rationale.

I created a view of all transaction amounts less than $2.00 grouped by credit card.

CREATE VIEW two_dollar as 
SELECT card_number, COUNT(card_number) as "transactions per card_number less than $2" 
FROM transactions
WHERE amount < 2.00
GROUP BY card_number; 

I searched for the average number of transactions per card and it was 6.6

SELECT ROUND(AVG("transactions per card_number less than $2"),2) as "average number of transactions less than $2"
FROM two_dollar;

I searched for the maximum number of transactions for a given credit card and arrived at 13, that is approximately double the average and should be flagged and further investigated for fraudulent activity.  

SELECT MAX("transactions per card_number less than $2") as "largest number of transactions per card"
FROM two_dollar;

I then searched for the corresponding card number that matched the 13 transactions.  

SELECT card_number, ("transactions per card_number less than $2") as "largest number of transactions per card"
FROM two_dollar
WHERE "transactions per card_number less than $2" =13;

--What are the top 5 merchants prone to being hacked using small transactions?

CREATE VIEW join_trx_merchant as
SELECT transactions.merchant_id, merchant.merchant_category_id, merchant.merchant_name, transactions.amount
FROM transactions
INNER JOIN merchant ON transactions.merchant_id = merchant.merchant_id;

SELECT merchant_name, COUNT(merchant_name) as "number of transactions under $2" 
FROM join_trx_merchant
WHERE amount <2.00
GROUP BY merchant_name
ORDER BY "number of transactions under $2" DESC
LIMIT 5;
