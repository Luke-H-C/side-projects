#under construction...

# Summary
This is about simplifying data cleaning process when I was working at GIVEasia. The original method was using a set of complex Excel functions to clean the data. I replaced the Excel solution with a new piece of SQL script I built from scratch.

# About GIVEasia
[GIVEasia](https://give.asia/) is a crowdfunding platform for people to raise money online. It offers services similar to [GoFundMe](https://www.gofundme.com/). Instead of charging transaction fees, it relies on optional "tips" from users to cover operational expenses. For example, the user can donate $100 to a campaign and pay an optional $10 tip to GIVEasia.

# The Problem
Since GIVEasia was founded, thousands of transactions have been recorded. When a user makes a donation on GIVEasia, a new row of the transaction record is created. Typically I have to manually go to the backend, hit a button to export the monthly transaction CSV file in the following format:

Column| Field (String) | Example 
------|-------|--------
Date | Timestamp of donation | 1 Jun 2017 06:33:40 +0800
Name | Username | LukeHC or Anonymous
E-mail | Email | Test@gmail.com
Charity | Organization who receives the money | RED CROSS TAIWAN
Movement | Fundraising campaign | Help Typhoon Victims
Amount | Donation amount | SGD 20.00
Net Amount | Donation amount less payment processing fees | SGD 19.60

Things that needed to be fixed:

1. Some fields are messed up with redundant text. For example: "SGD" (which stands for Singapore Dollar)
2. Data types
3. Special or test fundraising campaigns are also included 

# First solution with EXCEL 

Excel was the only data analysis tool I knew so it was obviously the only choice. I had to perform the following steps just to get the basic descriptive statistics.

1. Manually merging all monthly transaction files
2. Removing redundant text
3. Summarizing data with complex array functions

I built nested functions like TRIM, LEFT, RIGHT, MID, FIND, Array functions and a bunch of VLOOKUP. To make things easier I created multiple sheets to clean data in different stages. Performance soon became a major headache and I would literally spend the entire afternoon just to update the file with additional monthly data. I tried to learn Excel VBA to automate the process but it didn't take long for me to realize that I needed a different approach.

# Searching for a better solution - SQL

I took free courses on Khan Academy - [Hour of SQL](https://www.khanacademy.org/computing/hour-of-code/hour-of-sql). I also setup a local MySQL server using MySQL Workbench. This was probably the first time I was serious about learning some kind of programming language.(Well, strictly speaking, SQL is not actually a programming language)

#  Correcting formats

The first agenda to me was to produce a "cleaned" version of transaction data. I imported the raw data to MySQL Workbench and created a VIEW to achieve this. I also changed the data type if required.

```
# Only extract necessary string from timestamp field and change the data type

STR_TO_DATE(TRIM(BOTH ' ' FROM LEFT(`


`.`Date`, 11)),
                '%d %b %Y') AS `Donation_Date`,
                
#Remove trailing spaces from these fields

        TRIM(BOTH ' ' FROM `raw_donation_data`.`Name`) AS `Donor_Name`,
        LOWER(TRIM(BOTH ' ' FROM `raw_donation_data`.`Email`)) AS `Email`,
        TRIM(BOTH ' ' FROM `raw_donation_data`.`Charity`) AS `Charity`,
        TRIM(BOTH ' ' FROM `raw_donation_data`.`Movement`) AS `Movement`,
```

Here comes another tricky thing, the system generates two different versions of CSV files. I wasn't aware of this until I discovered the discrepancy between my analysis later on. I needed to build an "if...then" rule to process the ```Net_Amount``` field.

```
(CASE
            WHEN
                (LOCATE('SGD', `raw_donation_data`.`Net_Amount`) = 1)
            THEN
                CAST(REPLACE(SUBSTR(`raw_donation_data`.`Net_Amount`,
                            4,
                            (CHAR_LENGTH(`raw_donation_data`.`Net_Amount`) - 4)),
                        ',',
                        '')
                    AS DECIMAL (7 , 2 ))
            ELSE CAST(REPLACE(SUBSTR(`raw_donation_data`.`Net_Amount`,
                        2,\
                        (CHAR_LENGTH(`raw_donation_data`.`Net_Amount`) - 8)),
                    ',',
                    '')
                AS DECIMAL (7 , 2 ))
```
A clean dataset is now created. I can move on to summarize the data.

# Summarize data

From a business perspective, I'd like to segment users to personalize our offerings.

Column Name| Defination | Remarks 
------|-------|--------
Email|Unique user ID
Name|username|some users are “guest/anonymous”
First Donation to Charity|the date of first donation was made on GIVEasia.|find early users
Last_Charity|the date of last donation was made on GIVEasia.|users recent activities
Number of Donations to Charity|# times the user has made a donation.| “Giving GIVEasia tip during checkout process” is considered as one donation. Use this to identify active users 
SUM_Charity|total amount of donation of that user has made(plus tip to GIVEasia).|High value users
AVG_Charity|Sum of Donation/Number of Donations.|donation behavior metric
Donation to GIVE|$ of donation to the team|our key revenue source


```
#Select the segmentation listed above

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

# I'm going to SELECT stuff from two JOINed tables
# Here is a table only includes donations to charities - T_Donation_to_Charities

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
    
# Another table only includes donation to GIVEasia team - T_Donation_to_GIVE

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

P.S I believe there is a better way to fliter out the data without having two joined tables.

# Visualization
I made these charts using EXCEL (and followed the data visualization guildlines by [Stephen Few.](https://www.amazon.com/Stephen-Few/e/B001H6IQ5M)) I also randomized all actual numbers (Excel RAND function) and removed meaningful axis names. I discovered a lot of interesting patterns and also confirmed some of my previous understandings about our users.

![Decreasing conversion rate](https://github.com/LukeHC/side-projects/blob/master/GIVEasia-transaction-data-wrangling/Decreasing%20conversion%20rate.png)

![Average conversion rate](https://github.com/LukeHC/side-projects/blob/master/GIVEasia-transaction-data-wrangling/Average%20conversion%20rate.png)

![Average tips](https://github.com/LukeHC/side-projects/blob/master/GIVEasia-transaction-data-wrangling/Average%20tips.png)
# Conclusion
I was able to get the cleaned data in a few seconds compared with a few hours before. If I have another 10 hours on this project, I'd like to learn how to directly quiry the database instead of the current ad-hoc "download-import-quiry" process. In this way I have the opportunity to build real-time metrics as well. 


PLACEHOLDER: MUST DO SPELLING AND GRAMMAR CHECK

