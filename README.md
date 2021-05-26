# Recurrent
Data-X: Helping SaverLife Fight Wealth Inequality by Driving Financial Savings

**Project Overview**
Overview of project (problem, solution)
Problem: There is a significant wealth inequality in the United States currently. Moreover, research has shown that this gap continues to increase, with those at the bottom not only lacking financial stability, but also access to 401(k), saving funds, loans, etc.  Unfortunately, with the recent Covid-19 pandemic, this gap is exacerbated further and savings have become more important and necessary than ever. In fact, many Americans live paycheck to paycheck with 40% of Americans having less than $400 in savings for emergencies. The team is passionate about bridging the inequality gap and aims to help those that are traditionally underserved from financial institutions find a more engaging way to start saving.

**Solution**: To address the wealth inequality gap, the team sought to partner with an equally driven organization. By networking with Berkeley’s community, the team established a connection with SaverLife, a non-profit organization that helps consumers and families build financial security by making saving money easier and more rewarding through their cash prizes. The partnership with SaverLife was crucial in unlocking key data points and in broadening the team’s understanding of the current landscape. SaverLife provided the team with an extensive database of transactional-level spending and income data for the targeted population, which was crucial in predicting users’ behaviors.

**Architecture**
_Data Sources_: As mentioned, SaverLife provided us with transactional spending and income data for their population. Due to privacy concerns, we did not have access to all of their data, but we were given a single picture that had all data tables and variables available in their database. Using SQL, we pulled a csv file with 280,125 observations, where each observation represents data for a single user’s checking account in one month. The step-by-step process of pulling this data is outlined in more detail below. Additionally, we met with SaverLife on a weekly basis to ensure our understanding is accurate and share our progress.  

_Models_: Initially, we ran a linear regression model to understand the relationship between savings and spending. This model yielded that the median savings each month across all users was $0, meaning that users would spend the entirety of their monthly income, which exemplifies the difficult challenge in encouraging long-term savings propensity. 
Next, we decided to run RNN which allowed us to use the time series aspect of the data. When predicting next month’s balance for a user, we trained the model by using their balance from prior months. To build the model, we included data about users’ spending and income, their frequency of interacting with the app, and flags for month, year, and Partner A users.

_UI_: The team developed a tableau workbook that allows each user to see their future predicted account balances, their predicted versus actual balances for past months, and their income, spending, and balance history. To motivate users to save more, the model may be adjusted in the future to show future predicted balances that are +10% as an aspirational goal. The current UI visualization was designed based on interviews with Saverlife to provide the most important and relevant information to their users.

_Major takeaways_: As mentioned above, the key takeaway from the linear regression model was that users spend the entirety of their monthly income. From the RNN model, we concluded that the model’s predictions highlight the need for developing more robust incentive strategies based on how engaged the user is. For users who are more engaged, the predictive model performs much better. Additionally, SaverLife could explore its incentive strategy and would perform better if it’s segmented based on whether a user utilizes Partner A or not. 

# Installation & Running
- Upload folder into Colab. 
- RNN-data-pull.ipynb pulls the most recent input data. The results of that pull are in the file TS_ALL_V2.csv. You shouldn’t have to run the data pull itself.
- RNN: Edit Current V3 of RNN_LTSM_DataX [y = balance].ipynb. Scroll to the section on preprocessing data and ensure that the input file is in the specified folder (change that filepath, as needed). You can then run the script in Colab itself.
- Linear Regression:
For Monthly Income Versus Spending.ipynb, perform the same steps as above.

# Extending Project
To anyone interested in extending the RNN, areas to look into are:
Exploring engagement data to drill into the interplay with SaverLife interaction with users and eventual net savings. 
Exploring different signup methods (Partner A, etc.) to explore how those various sign up methods could affect net savings).

# Data Sources
SaverLife gave us access to a portion of their database, excluding certain data tables and variables that contained sensitive or personal information

We received limited documentation regarding the meaning of tables and variables, contained mostly in a single picture that had the names of all data tables, variables in each table, and data type for each variable

This picture included all available data tables and variables regardless of privacy levels - we do not have access to all of these tables and variables

We explored this database and met with Saverlife regularly to confirm our understanding of relevant data tables and variables

We pulled data for our model from the database using SQL. Our SQL pull is included on our GitHub, under the name RNN Input Pull

The steps in our pull are as follows:
  - Step 1: Establish a connection to the Saverlife database. Everyone on the team received specific usernames and passwords allowing us access to the database. One of our credentials are in place on this file, granting access to the database
  - Step 2: Pull monthly spending, income, and transaction data for each user with a checking account
  - Step 3: Pull daily balances for these checking accounts
  - Step 4: Limit balances to the first observation of the month
  - Step 5: Pull the number of interactions that users have with the Saverlife website in each month
  - Step 6: Merge spending/income dataset with account balance dataset and interactions dataset
  - Step 7: Limit to users with at least 4 months of activity
  - Step 7: Export to a csv file

The final output is a csv file with 280,125 observations. Each observation represents data for a single user’s checking account in one month during January 2019 to May 2021.

The RNN Input Pull file should not need any changes to run, apart from changing the file location of the csv output.

**Tables / Variables Referenced**:

_Plaid Main Transactions_ - all transactions for the account that users’ designate as their primary savings vehicle. Includes checking, savings, PayPal, and other account types
  - Date
  - Amount_cents - the amount of the transaction, in cents. For example, a value of 100 would represent $1.00. Negative amounts represent money into the account (e.g. income, cashed check, or a reversed transaction), while positive values represent money out of the account (spending). We require this amount’s absolute value to be less than 1,000,000 ($10,000) to remove outliers. Based on discussions with Saverlife, we understand that this file includes a limited number of foreign currency transactions, and these transactions are likely represented by the large values we see (for example, $1 = 1,100 Korean Yen)
  - Category_id - category of spending/income. We are blocked from seeing specific vendor names for privacy reasons - however, the value of this variable gives us insight into the kind of vendor, e.g. restaurant, grocery store, or account transfer. We specifically remove category id 21001000 because this corresponds to an account transfer (for example, moving money from checking to savings) and did not want to count these transactions as income or spending.
Bank_account_id - Unique identifier for each bank account, connects this data table to the Bank Accounts data table

_Plaid Auxiliary Transactions_ - all transactions for the account that users’ grant Saverlife access to, but do not designate as their primary savings vehicle
This data table has the same variables and same structure as Plaid Main Transactions

_Bank Accounts_ - table with detail on each bank account linked by a user, including all accounts in Plaid Main Transactions and Plaid Auxiliary Transactions
  - Id - connects to bank_account_id in Plaid Main Transactions and Plaid Auxiliary Transactions
  - Current_balance_cents - the account’s current balance. This value changes every time a user’s account sees a new transaction
  - Account subtype - signifies if the account is savings, checking, PayPal, etc.
  - Plaid_financial_authentication_id - connects this dataset to plaid_financial_authentications

_Plaid financial authentications_ - gives additional information regarding accounts, including whether the account is still open
  - Id - connects this data table to the Plaid_financial_authentication_id variable in the Bank Accounts data table
  - Is_current - denotes whether the account still live and usable by the user. We require accounts to be current for purposes of calculating account balances
  - State - denotes whether the account is still “connected” to the SaverLife platform. We require accounts to be connected for purposes of calculating account balances

_Marketing conversions_ - list of various marketing campaigns initiated by SaverLife
  - Marketing_partner_id - connects this data table to Marketing Partners

_Marketing partners_ - Includes further details on partners referenced in the Marketing Conversions data table
  - Id - connects this data table to Marketing conversions
  - Name - Name of the marketing partner. Partner A is one of Saverlife’s largest marketing partners - we used this value as a dummy variable in our model

_Amplitude events_ - list of every “event” performed by a user. Events are user interactions with the Saverlife website, including login, navigating between pages, and clicks.
  - Event_time - the time that the event happened
  - Event_type - action performed by the user
  - User_id - connects to amplitude_id in the Users data table

_Users_ - list of every user that signed up with Saverlife, as well as various demographic data offered at signup
  - Id - corresponds to the User_id field in Plaid Main Transactions, Plaid Auxiliary Transactions, and most other data tables, although NOT Amplitude Events
  - Amplitude_id - connects this table to Amplitude Events

File: RNN Data Pull.ipynb

# Models

**Linear Regression**
This model was meant to test the linearity between various features of a user’s financial situation. We tried various combinations of features and ultimately settled on using income data to predict spending, and came out with an almost linear output. This means that users tend to spend most of their income for each month. As part of our methodology, we also removed outliers which tended to strongly influence the results. 

Input: TS_All_V2 .csv file from the data source output into our Python code for analysis
Output: predicted spending values, along with an R-squared value
Model Filename: Monthly Income Versus Spending.ipynb

**RNN**
Once we realized that there’s a bit of a time-based component to values, we thought of adding that into a new model. Based on our homeworks and talking with the GSI, we landed on an RNN model where we took in the last 3 months of feature data to try to predict the following month’s account balance.

Some thoughts:
We realized that account balance values were more stable than net savings values, so we used that as our y-value.
This model was our most successful attempt, reaching a scaled MAE of about 1-2% of account balance values, or roughly $900 in an absolute sense. 

Model Filename: Current V3 of RNN_LTSM_DataX [y = balance].ipynb
Run info: To run the model, run the file above, using one of the specified input methods (Colab/cloud or local). The run might take a few minutes but will output model prediction results using a median absolute error (MAE) loss function, as well as create an input to the UI layer.

# UI
A Tableau workbook has been developed to show individual model results to Saverlife users, along with further data to provide context for the model's predictions. This visualization was designed through interviews with Saverlife to provide the most important and relevant information to users, and will ideally be converted into a Tableau Server product for personal use or otherwise made available within the Saverlife app suite. Open the "Saverlife UI *.twbx’ file using Tableau Desktop (requires a license) or Tableau Reader (free) to see this dashboard.

Each user sees their predicted account balances, their predicted versus actual balances for past months, and their income, spending, and balance history. The future predicted balances are intended to motivate users to hit the 'goals' set for them, and may be adjusted in future to show model predictions + 10% as an 'aspiration'. The past months of predicted versus actual balances along with the true financial history data shows the user their progress and performance to date.

Currently, this workbook relies on flat .csv input ('TS_'.csv) and output files (SVLF_balance.csv) from team members and imports the necessary columns. To re-run the workbook, you would edit the Connections on the Data Source tab to point to the new files, which would then refresh the Tableau extracts behind the workbook. Ideally, this workbook would be refactored to pull the necessary input data directly from the Saverlife database. However, creating a direct connection between the model outputs and the Tableau workbook would require significantly more effort.

Filename: Saverlife UI 20210504.twbx

# Authors
Aleksandra Bogoevska

Alexander Sergian

Augustine Santillan

Christine Yee

Jeremy Smith

Steven Gan
