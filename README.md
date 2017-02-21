# Who am I

Someone who loves to make data-informed decisions 

[LinkedIn](https://www.linkedin.com/in/lukehcliu/)

# Machine Learning with Python

## [Survival Prediction (link to Kaggle)](https://www.kaggle.com/skywalkerhc/titanic/machine-learning-for-survival-prediction-2)
From data wrangling, exploration to feature engineering, the simple logistic regression model predicts who can survive in the Titanic tragedy.

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Kaggle/Coefficient%20Est.png)

# SQL

## [GIVEasia Transaction Data Wrangling](https://github.com/LukeHC/The-Quantitative-Decision/blob/master/GIVEasia-transaction-data-wrangling/README.md)
I wrote SQL to simply transaction data cleaning process.
```
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
```

# Strategic Planning

From partnership assessment, new technology evaluation to market entry strategy. I had experiences in top global companies.

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/1.png)


![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/2.png)

# Data Visualizations
I'm a strong believer of [Edward Tufte's](https://en.wikipedia.org/wiki/Edward_Tufte) design principles.

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/GIVEasia-transaction-data-wrangling/Average%20conversion%20rate.png)

Guess what, I also rebuilt the entire data visualization guidelines when I was in the one-year compulsory alternative military service(替代役). Here is an example:

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Data%20Visualization/Before_After.png)

# Funnel Optimization
I increased conversion rates by sending personalized messages to website visitors, leads, customers, and promoters. I leveraged Typeform, MailChimp, Zendesk, Zapier and many other tools to build automated workflows. The following flowchart is an example of how the information was passed between different tools:

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/Lead%20nurturing.png)

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/NPO%20Onboarding.png)

# Operations

People are afraid of hiring someone who can TALK but can NOT do stuff. I feel the same way. Two stories to share:

## A
At [GIVEasia](https://give.asia), a crowdfunding platform for charitable purposes, I ran the customer support for 6 months. I replied angry customer's emails and called someone in the MiddleEast to resolve technical issues. I customized our customer support tools (Zendesk) like building Macros and automated workflows. 

I made a 13-page step-by-step guide on how to run operations. Here is the outline and some content:
![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/GIVE%20Operations%20Guide%20Outline.png)

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/Specefic%20guide%20on%20how%20to%20use%20Zendesk.png)

## B
At Zimplistic, a hardware startup at growing stage, I ran the product pilot program with a small team in 3 months. The purpose was to send functional products to some users and tested everything before mass production. This was a critical project for the company.

We designed the entire operations flow from taking orders, sending the machine, customer support and feedback analysis. Here is an example:

![](https://github.com/LukeHC/The-Quantitative-Decision/raw/master/Strategic%20Planning/Customer%20support%20flow.png)

Under each key sections, I tracked every action items on a massive spreadsheet, like who needs to deliver what on which date. After months of planning and practicing, we finally personally delivered the first product to someone's house.

# The last thing to say...

I'd like to say I built everything you've seen above from scratch (including all the fancy flowcharts). And yes, it's technically correct.

However, without the help of team members and guidance from external advisors, I could not have a chance to learn, build and break things. I want to say a huge Thank-you to everyone who had help me.
