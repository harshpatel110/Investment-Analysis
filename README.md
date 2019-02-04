# Investment-Analysis

## Importing required libraries

import pandas as pd 
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# CHECKPOINT 1 : Data Cleaning 1

## Creating required dataframes

companies = pd.read_csv("companies.txt", sep="\t", encoding = "iso-8859-1")
rounds2 = pd.read_csv("rounds2.csv", encoding = "iso-8859-1")

#checking if any company is there in rounds2 dataframe which is not present in the companies dataframe.
set(rounds2['company_permalink'].str.upper()) - set(companies['permalink'].str.upper())

## Removing Special Characters

#Let's check, is the special characters is the reason for not getting the expected NULL intersection.
#we are trying to convert those special characters to the simple English characters.
#converting the special characters to understandable english characters.
companies['permalink'] = companies['permalink'].str.encode('utf-8').str.decode('ascii', 'ignore')
rounds2['company_permalink'] = rounds2['company_permalink'].str.encode('utf-8').str.decode('ascii', 'ignore')
set(rounds2['company_permalink'].str.upper()) - set(companies['permalink'].str.upper())

## Question 1: How many unique companies are present in rounds2? 

rounds2['company_permalink'] = rounds2['company_permalink'].str.lower()
rounds2['company_permalink'].describe()

## Answer: 66368

## Question 2: How many unique companies are present in companies file? 

companies['permalink'] = companies['permalink'].str.lower()
companies['permalink'].describe()

## Answer:66368

## Question 3: In the companies data frame, which column can be used as the  unique key for each company?

rounds2.head()

companies.head()

## Answer: 'permalink'

## Question 4: Are there any companies in the rounds2 file which are not  present in companies? 

set(rounds2['company_permalink']) - set(companies['permalink'])

## Answer: No(N)

#Creating the master_frame dataframe by mering companies and rounds2 dataframes.
master_frame = pd.merge(rounds2, companies, left_on = 'company_permalink', right_on = 'permalink', how ='inner')
master_frame

## Question 5: Merge the two data frames so that all  variables (columns)  in the companies frame are added to the rounds2 data frame. Name the merged frame master_frame. How many observations are present in master_frame?

master_frame['company_permalink'].describe()

## Answer: 114949

# CHECKPOINT 2: Funding Type Analysis

#average raised amount for the Venture
venture = rounds2['funding_round_type'] == "venture"
venture = rounds2[venture]
#average raised amount for the Angel
angel = rounds2['funding_round_type'] == "angel"
angel = rounds2[angel]
#average raised amount for the Seed
seed = rounds2['funding_round_type'] == "seed"
seed = rounds2[seed]
#average raised amount for the Private Equity
private_equity = rounds2['funding_round_type'] == "private_equity"
private_equity = rounds2[private_equity]
print('venture = ', venture['raised_amount_usd'].mean(), ', angel = ', angel['raised_amount_usd'].mean())

## Question 1: Average funding amount of venture type

venture['raised_amount_usd'].mean()

## Answer: 11748949.129489528

## Question 2: Average funding amount of angel type

angel['raised_amount_usd'].mean()

## Answer: 958694.4697530865

## Question 3: Average funding amount of seed type

seed['raised_amount_usd'].mean()

## Answer: 719817.9969071728

## Question 4: Average funding amount of private equity type

private_equity['raised_amount_usd'].mean()

## Answer: 73308593.02944215

## Question 5: Considering that Spark Funds wants to invest between 5 to 15 million USD per  investment round, which investment type is the most suitable for them?

#Let's filter out the average investments between 5 to 15 million USD
print('venture = ', venture['raised_amount_usd'].mean() >= 5000000 and venture['raised_amount_usd'].mean() <= 15000000,
      '\nangel = ', angel['raised_amount_usd'].mean() >= 5000000 and angel['raised_amount_usd'].mean() <= 15000000,
      '\nseed = ', seed['raised_amount_usd'].mean() >= 5000000 and seed['raised_amount_usd'].mean() <= 15000000,
      '\nprivate_equity = ', private_equity['raised_amount_usd'].mean() >= 5000000 and private_equity['raised_amount_usd'].mean() <= 15000000)

## Answer: Venture

# Checkpoint 3: Country Analysis

#let's select only countries which are having venture as investment type.
master_frame = master_frame.loc[master_frame['funding_round_type'] == 'venture']
#list of english speaking coutries with 3 letter representation code is below.
#note this list is extracted from 'http://www.emmir.org/fileadmin/user_upload/admission/Countries_where_English_is_an_official_language.pdf'.
#3 letter country codes are extracted from 'https://www.nationsonline.org/oneworld/country_code_list.htm'.
english_countries = ["BWA", "CMR", "ETH", "ERI", "GMB", "GHA", "KEN", "LSO", "LBR", "MWI", "MUS", "NAM", "NGA", "RWA", "SYC", "SLE", "ZAF", "SSD", "SDN", "SWZ", "TZA", "UGA", "ZMB", "ZWE", "ATG", "BHS", "BRB", "BLZ", "CAN", "DMA", "GRD", "GUY", "JAM", "KNA", "LCA", "VCT", "TTO", "USA", "IND", "PAK", "PHL", "SGP", "AUS", "FJI", "KIR", "MHL", "FSM", "NRU", "NZL", "PLW", "PNG", "WSM", "SLB", "TON", "TUV", "VUT", "IRL", "MLT", "GBR"]
#keeping only english speaking countries in master_frame
master_frame= master_frame.loc[master_frame['country_code'].isin(english_countries)]
#getting the total raised_amount_usd of the each english speaking country.
english_countries = master_frame.groupby(['country_code'])['raised_amount_usd'].aggregate('sum')
#selecting only top 9 countries
top9 = pd.DataFrame(english_countries.sort_values(ascending = False).head(9))
#making index as a new column in top9 dataframe
top9['country_code'] = top9.index
#reseting index as traditional.
top9 = top9.reset_index(drop=True)
#Interchanging rows for better representation of dataframe top9
top9 =  top9[['country_code', 'raised_amount_usd']]
#top9 dataframe
top9

## Question 1: Top English speaking country

 top9.iloc[[0]]

## Answer : USA

## Question 2: Second English speaking country

top9.iloc[[1]]

## Answer: GBR

## Question 3: Third English speaking country

top9.iloc[[2]]

## Answer: IND

# Checkpoint 4: Sector Analysis 1

#Splitting the column 'category_list' and making it as a new column.
x = master_frame["category_list"].str.split("|", n = 1, expand = True)
master_frame['primary_sector'] = x[0]
master_frame

## Question 1:Code for a merged data frame with each primary sector mapped to its main sector 

#creating a dataframe called mapping by reading mapping.csv
mapping = pd.read_csv("mapping.csv", encoding = "iso-8859-1")
#converting dummy variables into categorical variables.
n = range(1,10)
for i in  n:
    mapping.iloc[:,i] =  mapping.iloc[:,i].replace(1, list(mapping)[i])
for i in n:
    mapping.iloc[:,i] =  mapping.iloc[:,i].replace(0, "")
mapping['main_sector'] = mapping.iloc[:,1].map(str) + mapping.iloc[:,2].map(str) + mapping.iloc[:,3].map(str) + mapping.iloc[:,4].map(str) + mapping.iloc[:,5].map(str) + mapping.iloc[:,6].map(str) + mapping.iloc[:,7].map(str) + mapping.iloc[:,8].map(str) + mapping.iloc[:,9].map(str)

#Keeping only required columns by dropping range of unnecessary columns mapping dataframe.
mapping.drop(mapping.iloc[:, 1:-1], inplace=True, axis=1)
mapping.head()

master_frame = master_frame.reset_index().merge(mapping, left_on='primary_sector', right_on= 'category_list', how="left").set_index('index')

master_frame.drop(master_frame.iloc[:, -2:-1], inplace=True, axis=1)

## Answer: below is the required merged dataframe which is having the required columns as last two.

master_frame['main_sector'].isna().describe()

# Checkpoint 5: Sector Analysis 2

#taking only amount between 5 to 15 million.
amount = range(5000000, 15000001)
master_frame = master_frame.loc[master_frame['raised_amount_usd'].isin(amount)]
master_frame

#Creating three dataframes D1, D2, D3 for top investing countries USA, GBR and IND respectively.
D1 = master_frame.loc[master_frame['country_code'] == 'USA']
D2 = master_frame.loc[master_frame['country_code'] == 'GBR']
D3 = master_frame.loc[master_frame['country_code'] == 'IND']
D1.groupby(['main_sector'])['main_sector'].aggregate('count')

#Summerise D1 by main_sector in terms of sum of raised_amount and count for the same.
D1_summary = D1.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['sum'], ascending = False)
D1_summary

#Summerise D2 by main_sector in terms of sum of raised_amount and count for the same.
D2_summary = D2.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['sum'], ascending = False)
D2_summary

#Summerise D3 by main_sector in terms of sum of raised_amount and count for the same.
D3_summary = D3.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['sum'], ascending = False)
D3_summary

## Question 1: Total number of Investments (count)

D1['raised_amount_usd'].notna().sum()

## Answer(c1): 12150

D2['raised_amount_usd'].notna().sum()

## Answer(c2): 628

D3['raised_amount_usd'].notna().sum()

## Answer(c3): 330

## Question 2: Total amount of investment (USD)

D1['raised_amount_usd'].sum()

## Answer(c1): 108531347515.0

D2['raised_amount_usd'].sum()

## Answer(c2): 5436843539.0

D3['raised_amount_usd'].sum()

## Answer(c3): 2976543602.0

D1_top = D1_summary.index[0]
D1_second =  D1_summary.index[1]
D2_top = D2_summary.index[0]
D2_second =  D2_summary.index[1]
D3_top = D3_summary.index[0]
D3_second =  D3_summary.index[1]

## Question 3: Top Sector name (no. of investment-wise)
## Question 4: Second Sector name (no. of investment-wise)
## Question 5: Third Sector name (no. of investment-wise)
## Question 6: Number of investments in top sector (3)
## Question 7: Number of investments in second sector (4)
## Question 8: Number of investments in third sector (5)

D1_summary

## Answer3(c1): Others
## Answer4(c1): Cleantech / Semiconductors
## Answer5(c1): Social, Finance, Analytics, Advertising
## Answer6(c1): 2923
## Answer7(c1): 2297
## Answer8(c1): 1912

D2_summary

## Answer3(c2): Others
## Answer4(c2): Cleantech / Semiconductors
## Answer5(c2): Social, Finance, Analytics, Advertising
## Answer6(c1): 143
## Answer7(c1): 127
## Answer8(c1): 98

D3_summary

## Answer3(c2): Others
## Answer4(c2): News, Search and Messaging
## Answer5(c2): Entertainment
## Answer6(c1): 109
## Answer7(c1): 52
## Answer8(c1): 33

## Question 9: Top Sector name (no. of investment-wise)

D1_top = D1.loc[D1['main_sector'] == D1_top]

D1_top.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c1): Virtustream

D2_top = D2.loc[D2['main_sector'] == D2_top]

D2_top.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c2): Electric Cloud

D3_top = D3.loc[D3['main_sector'] == D3_top]

D3_top.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c3): FirstCry.com

## Question 10: Second Sector name (no. of investment-wise)

D1_second = D1.loc[D1['main_sector'] == D1_second]

D1_second.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c1): Biodesix

D2_second = D2.loc[D2['main_sector'] == D2_second]

D2_second.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c2): EUSA Pharma

D3_second = D3.loc[D3['main_sector'] == D3_second]

D3_second.groupby(['name'])['raised_amount_usd'].aggregate('sum').sort_values(ascending=False, inplace=False).head(1)

## Answer(c3): GupShup

# Checkpoint 6: Plots

#Aggreating necessary funding type and averaged raised amount in one dataframe called fund_avg. 
funding_type = pd.Series(['venture', 'angel', 'seed', 'private_equity'])
avg_raised_amount = pd.Series([venture['raised_amount_usd'].mean(), angel['raised_amount_usd'].mean(), seed['raised_amount_usd'].mean(), private_equity['raised_amount_usd'].mean()])
fund_avg = pd.concat([funding_type, avg_raised_amount], axis=1).reset_index(drop=True)
fund_avg = fund_avg.rename(columns={0: 'funding_type', 1: 'avg_raised_amount'})
fund_avg

## Question 1: A plot showing the fraction of total investments (globally) in venture, seed, and private equity, and the average amount of investment in each funding type. This chart should make it clear that a certain funding type (FT) is best suited for Spark Funds.

## Answer: Plot1

plt.figure(figsize=(15,4))
plot1 = sns.barplot(x = avg_raised_amount, y = funding_type,)
plot1.set(xlabel = 'avg_raised_amount(in millions)', ylabel = 'funding_type')
plot1.set_xlim(0, 80000000)
fig1 = plot1.get_figure()
fig1.savefig("plot1.png")

## Question 2: A plot showing the top 9 countries against the total amount of investments of funding type FT. This should make the top 3 countries (Country 1, Country 2, and Country 3) very clear.

## Answer: Plot2

#top9 is the dataframe which is having list of top9 countries having funding type as venture and total raised amount.
plt.figure(figsize=(16,5))
plot2 = sns.barplot(x = top9['raised_amount_usd'], y = top9['country_code'])
plot2.set(xlabel = 'raised_amount(in millions)', ylabel = 'country')
fig2 = plot2.get_figure()
fig2.savefig("plot2.png")

## Question3 : A plot showing the number of investments in the top 3 sectors of the top 3 countries on one chart (for the chosen investment type FT). 

## Answer: Plot3

D1_top3 = D1.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['count'], ascending = False).head(3).reset_index()
D2_top3 = D2.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['count'], ascending = False).head(3).reset_index()
D3_top3 = D3.groupby(['main_sector'])['raised_amount_usd'].aggregate(('sum', 'count')).sort_values(by=['count'], ascending = False).head(3).reset_index()
D1_top3['main_sector'] = 'USA- ' + D1_top3.iloc[:,0].map(str)  
D2_top3['main_sector'] = 'GBR- ' + D2_top3.iloc[:,0].map(str)  
D3_top3['main_sector'] = 'IND- ' + D3_top3.iloc[:,0].map(str)
top3 = pd.concat([D1_top3, D2_top3, D3_top3], axis=0)
top3
plot3 = sns.barplot(x = top3['count'], y = top3['main_sector'])
plot3.set_xlim(0, 3500)
plot3.set(xlabel = 'Number of investments')
fig3 = plot3.get_figure()
fig3.savefig("plot3.png")

# End
