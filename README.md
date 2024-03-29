# 07-SQL

##### Unit 7 Homework Assignment: Looking for Suspicious Transactions 

## Data Analysis

![ERD](Images/ERD.png)

#### How can you isolate (or group) the transactions of each cardholder?

SELECT card_number, COUNT(card_number) as "transactions per card_number"
FROM transactions
GROUP BY card_number; 

![trxns](Images/trxnspercard.png)

#### Consider the time period 7:00 a.m. to 9:00 a.m. What are the top 100 highest transactions during this time period? Do you see any fraudulent or anomalous transactions? If you answered yes to the previous question, explain why you think there might be fraudulent transactions during this time frame.

SELECT date_time, amount as "largest transactions from 7-9am"
FROM transactions
WHERE CAST(date_time as time) >= '07:00:00' 
   and CAST(date_time as time) <= '09:00:00'
ORDER BY amount DESC
LIMIT 100;

![morning](Images/seven_nine.png)

The average transaction size is $45.19, but the first 8 largest transactions during 7-9am average $1,226.25. These 8 large transactions appear to significant outliers in the data and therefore, are potentally fraudulent transactions. 

SELECT AVG(Amount) as "average transaction from 7-9am"
FROM transactions
WHERE CAST(date_time as time) >= '07:00:00' 
   and CAST(date_time as time) <= '09:00:00';
   
![average](Images/seven_nine_avg.png)

#### Some fraudsters hack a credit card by making several small payments (generally less than $2.00), which are typically ignored by cardholders. Count the transactions that are less than $2.00 per cardholder. Is there any evidence to suggest that a credit card has been hacked? Explain your rationale.

I created a view of all transaction amounts less than $2.00 grouped by credit card.

CREATE VIEW two_dollar as 
SELECT card_number, COUNT(card_number) as "transactions per card_number less than $2" 
FROM transactions
WHERE amount < 2.00
GROUP BY card_number; 

![two](Images/less_than_2.png)

I searched for the average number of transactions per card and it was 6.6

SELECT ROUND(AVG("transactions per card_number less than $2"),2) as "average number of transactions less than $2"
FROM two_dollar;

![avg_two](Images/average_less_than_2.png)

I searched for the maximum number of transactions for a given credit card and arrived at 13, that is approximately double the average and should be flagged and further investigated for fraudulent activity.  

SELECT MAX("transactions per card_number less than $2") as "largest number of transactions per card"
FROM two_dollar;

![max_two](Images/max_less_than_2.png)

I then searched for the corresponding card number that matched the 13 transactions.  

SELECT card_number, ("transactions per card_number less than $2") as "largest number of transactions per card"
FROM two_dollar
WHERE "transactions per card_number less than $2" =13;

![credit_card_2](Images/max_credit_card_less_than_2.png)

#### What are the top 5 merchants prone to being hacked using small transactions?

I searched for the number of transactions under $2.00 by merchant name and compiled the top 5 merchants with the largest number of small transactions. 

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

![top5merchants](Images/top5merchants.png)

#### Verify if there are any fraudulent transactions in the history of two of the most important customers of the firm. For privacy reasons, you only know that their cardholders' IDs are 18 and 2. What difference do you observe between the consumption patterns? Does the difference suggest a fraudulent transaction? Explain your rationale.

In analyzing a year of transaction data for cardholders' IDs 18 and 2, there is a significant difference in their comsumption patterns. Cardholder 2 consistently spends within a range of $2-20 dollars. Cardholder ID 18 also spends in a similar range but has a monthly one time spend of $1,000+ which could indicate potential fraud. 

![2](Images/card_holder_2.png)
![18](Images/card_holder_18.png)

#### The CEO of the biggest customer of the firm suspects that someone has used her corporate credit card without authorization in the first quarter of 2018 to pay quite expensive restaurant bills. You are asked to find any anomalous transactions during that period. Do you notice any anomalies? Describe your observations and conclusions.

The box plot indicates outliers in the transaction data each month for the first half of 2018. These data points are signficantly above the range and average consumption spend. These tranasactions are inconsistent with the consumption trends and should be further investigated. 

![25](Images/card_holder_25.png)
