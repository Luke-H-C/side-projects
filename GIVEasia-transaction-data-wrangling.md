# About GIVEasia
[GIVEasia](https://give.asia/) is a crowdfunding platform for people to raise money online. It offers services similar to [GoFundMe](https://www.gofundme.com/). Instead of charging transaction fees, it relies on optional "tips" from users to cover operational expenses. For example, use can donate $100 to a campaign and pay an optional $10 tip to GIVEasia. 


# The Problem
Since GIVEasia was founded, thousands of transactions have been recorded in the database. When a user makes a donation on GIVEasia, a new row of the transaction record is created. Typically I have to manually go to the backend, hit a button and export the monthly transaction CSV file in the following format:

Column| Field (String) | Example 
------|-------|--------
Date | Timestamp of donation | 1 Jun 2017 06:33:40 +0800
Name | Username | LukeHC or Anonymous
E-mail | email | Test@gmail.com
Charity |	organization who receives the money | RED CROSS TAIWAN
Movement | which fundraising campaign | Help Typhoon Victims
Amount | donation amount | SGD 20.00
Net Amount | donation amount without payment processing fee	| SGD 19.60

Things that need to be fixed:

1. Some fields are messed up with redundant text. For example "SGD" (which stands for Singapore Dollar)
2. Data types
3. Special or test fundraising campaigns are also included 

# First solution with EXCEL 

Excel was the only data analysis tool I knew so it's obviously my only choice then. I needed to perform the following steps just to get basic descriptive statistic.

1. Manually merge all monthly transactiontrupdateansaction files
2. Remove redudent text
3. Summarize data with complex array functions

Performance soon became a major headache and I would literally spend the entire afternoon just to update the file. I tried to learn VBA to automate the process but it didn't take long for me to realize that I need a different approach.

PLACEHOLDER : add sample Excel syntax

# Searching for a better solution - SQL

I took the free courses on Khan Academy - [Hour of SQL](https://www.khanacademy.org/computing/hour-of-code/hour-of-sql). I also setup a local MySQL server using MySQL Workbench. This was probably the first time I was serious about learning some kind of programming language.

#  Correcting formats

The first agenda to me is to produce a "cleaned" version of transaction data. I import the raw data to MySQL Workbench and create a VIEW to achieve this. I also change the data type if required.

```
# Only extract necessary string from timestamp and change datatype

STR_TO_DATE(TRIM(BOTH ' ' FROM LEFT(`row_donation_data`.`Date`, 11)),
                '%d %b %Y') AS `Donation_Date`,
                
#Remove remove trailing spaces from these fields

        TRIM(BOTH ' ' FROM `row_donation_data`.`Name`) AS `Donor_Name`,
        LOWER(TRIM(BOTH ' ' FROM `row_donation_data`.`Email`)) AS `Email`,
        TRIM(BOTH ' ' FROM `row_donation_data`.`Charity`) AS `Charity`,
        TRIM(BOTH ' ' FROM `row_donation_data`.`Movement`) AS `Movement`,
```
Here comes another tricky thing, the system generates two different versions of csv files. I wasn't aware of this until I discovered discrepancy between my analysis. I need to build a simple "if...then" rule to process the ```Net_Amount``` field.

```
(CASE
            WHEN
                (LOCATE('SGD', `row_donation_data`.`Net_Amount`) = 1)
            THEN
                CAST(REPLACE(SUBSTR(`row_donation_data`.`Net_Amount`,
                            4,
                            (CHAR_LENGTH(`row_donation_data`.`Net_Amount`) - 4)),
                        ',',
                        '')
                    AS DECIMAL (7 , 2 ))
            ELSE CAST(REPLACE(SUBSTR(`row_donation_data`.`Net_Amount`,
                        2,
                        (CHAR_LENGTH(`row_donation_data`.`Net_Amount`) - 8)),
                    ',',
                    '')
                AS DECIMAL (7 , 2 ))
```
A clean dataset is now created. I can move on to summarize the data.

# Summarize data
PLACEHOLDER; add business considerations 

From the business perspective, I'd like to see things like how much a user has contributed.

The list contains all the unique donors from August-2012 to June-2014
I customized some fields in the list so we can do some segmentations accordingly.

1) Full Name - users name. Please note that some users are “guest/anonymous”
2) First Donation - the date of first donation was made on GIVEasia. Use this to identify GIVE early users
3) Last Donation - the date of last donation was made on GIVEasia. Use this to figure out users recent activities
4) Number of Donations - How many times the user has made a donation. Please note that “giving GIVEasia tip during checkout process” is considered as one donation. Use this to identify active users  
5) Sum of Donation - total amount of donation of that user has made(plus tip to GIVEasia).  Use this to find ...rich users
6) Average Donation - Sum of Donation/Number of Donations. Use this to figure out user behaviors


```
SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
SELECT 
    `T_Donation_to_Charities`.`Email`,
    `Name`,
    `First Donation to Charity`,
    `Last_Charity`,
    `Number of Donations to Charity`,
    `SUM_Charity`,
    `AVG_Charity`,
    IFNULL(`Donation to GIVE`, 0) AS 'Donation to GIVE'
FROM
    (SELECT 
        cleaned_data.Email,
            Donor_Name AS 'Name',
            MIN(Donation_Date) AS 'First Donation to Charity',
            MAX(Donation_Date) AS 'Last_Charity',
            COUNT(cleaned_data.Email) AS 'Number of Donations to Charity',
            SUM(Net_Amount) AS 'SUM_Charity',
            FORMAT(AVG(Net_Amount), 2) AS 'AVG_Charity'
    FROM
        cleaned_data
    WHERE
        cleaned_data.Email <> ' '
            AND charity <> 'GIVEasia Website'
            AND charity <> 'GIVE.asia'
    GROUP BY cleaned_data.Email) AS T_Donation_to_Charities
        LEFT JOIN
    (SELECT 
        cleaned_data.Email, 
			SUM(Net_Amount) AS 'Donation to GIVE'
    FROM
        cleaned_data
    WHERE
        cleaned_data.Email <> ' '
            AND (charity = 'GIVE.asia'
            OR charity = 'GIVEasia Website')
    GROUP BY cleaned_data.Email) AS T_Donation_to_GIVE ON T_Donation_to_Charities.Email = T_Donation_to_GIVE.Email
```


# Conclusion
What the current state of the project and want can be built further 

PLACEHOLDER: MUST DO SPELLING AND GRAMMAR CHECK

