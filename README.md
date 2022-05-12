# __Sectors_Performance__
## __Overview__
Analysis of the sector performance and trend in a crash for the S&P 500 market.
The S&P 500 includes 500 major U.S. public companies focused primarily on market capitalisation. The S&P 500 is widely regarded as one of the best indicators of large-cap U.S. stocks and the broader stock market.<br/>
<br/>
Use Panda and Yahoo Finance API to get S&P50 sector data for the last four Finance recessions and visualise some aspects of the, and try to get some insights or predict into each sector's performance during crashes.<br/>
<br/>

---
## __The Answer I Am Looking For__
Can we predict the price trend for find a buying or selling signal in advance during the crashes?<br/>

__Decomposition Problem:__
* Will some sectors make less loss than others?<br/>
* What makes the price change differently?<br/>
* Can the previous price help to make a judgment for the price trend?<br/>
* Is there a correlation between sectors?<br/>

---
## __Recourse__
All the data are from Yahoo Finance API.<br/>

---
## __Step by Step Approach__
__Step 1 - Extract Data__<br/>
Can be divided into roughly two different data extracts:<br/>
> 1. Single symbol in a certain period<br/>
> 2. All 11 sectors' data in a certain period<br/>

<br/>
Sample for extract data: For singal symble in a certain period<br/>

``` python
import yfinance as yf
from datetime import datetime, timedelta
# Stock_Market_Selloff - 2015,8,18 - 2015,11,3
# Bitcoin_Crash - 2018,9,21 - 2019,4,23
# Covid_19  - 2020,2,20 - 2020,8,18
# Russian_Invasion_of_Ukraine - 2022,2,10 - 2022,3,29

# Set up crash name and periods
Period = "Market_Selloff"
start_date = datetime(2015,8,18).date()
end_date = datetime(2015,11,3).date()

# '^GSPC' is the symble for total S&P 500 market
SP500_df_volume = pd.DataFrame(DataReader('^GSPC', 'yahoo', start_date, end_date)['Volume'])
```
<br/>

Sample for extract data: Use loop to get all 11 sectors data in a a certain period<br/>

``` python
sector_list = ['^SP500-40','^SP500-25','^SP500-30',"^SP500-35","^SP500-20","^SP500-45","^SP500-15","^SP500-60","^SP500-50","^SP500-55","^GSPE"]

sector_name = ["Financials","Consumer Discretionary","Consumer Staples","Health","Industrials","Information Tech","Materials","Real Estate","Tele Services","Utilities","Energy"]

prerformance = ['^SP500-40','^SP500-25','^SP500-30',"^SP500-35","^SP500-20","^SP500-45","^SP500-15","^SP500-60","^SP500-50","^SP500-55","^GSPE"]
for stock in sector_list: 
    prerformance[sector_list.index(stock)] = yf.download(stock, start_date, end_date)
```
<br/>

__Step 2 - Plot Data__<br/>
Can be divided into roughly two different data plots: Single Plot & Subplot<br/>
> 1. Single plot<br/>
> 2. Subplot<br/>

<br/>

Sample for plot data: Single plot<br/>
<img src="https://github.com/Ash-Tao/Sectors_Performance/blob/main/Output/Russian_Invasion_of_Ukraine/Adj%20Close%20Change%20for%20S%26P500%20(Russian_Invasion).png"><br/>

``` python
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style('whitegrid')
plt.style.use("fivethirtyeight")
%matplotlib inline
import matplotlib.dates as mdates

# Find the index, price and date under a certain condition.
start_point_index = 0
start_point_date = SP500_df['Adj Close'].index[start_point_index].date()
start_point_value = round(SP500_df['Adj Close'][start_point_index],2)

lowest_point_index = SP500_df['Adj Close'].tolist().index(min(SP500_df['Adj Close'].tolist()))
lowest_point_date = SP500_df['Adj Close'].index[lowest_point_index].date()
lowest_point_value = round(SP500_df['Adj Close'][lowest_point_index],2)

end_point_index = len(SP500_df['Adj Close'])-1
end_point_date = SP500_df['Adj Close'].index[end_point_index].date()
end_point_value = round(SP500_df['Adj Close'][end_point_index],2)

# Set layout for the plot & reate a plot
plt.figure(figsize=(12, 7))
plt.rc('font', size=12)  
plt.plot(SP500_df.index,SP500_df['Adj Close'])

plt.plot(SP500_df['Adj Close'].index[start_point_index],start_point_value,"ro")
plt.annotate(f'The start point: {start_point_date}:  ${start_point_value}', (mdates.date2num(SP500_df['Adj Close'].index[start_point_index]),start_point_value), xytext=(10, 10), 
            textcoords='offset points', arrowprops=dict(arrowstyle='-|>',mutation_scale=30,facecolor='r'),size = 12)

plt.plot(SP500_df['Adj Close'].index[lowest_point_index],lowest_point_value,"ro")
plt.annotate(f'The lowest point: {lowest_point_date}:  ${lowest_point_value}', (mdates.date2num(SP500_df['Adj Close'].index[lowest_point_index]),lowest_point_value), xytext=(10, 10), 
            textcoords='offset points', arrowprops=dict(arrowstyle='-|>',mutation_scale=30,facecolor='r'),size = 12)

plt.plot(SP500_df['Adj Close'].index[end_point_index],end_point_value,"ro")
plt.annotate(f'The end point: {end_point_date}:  ${end_point_value}', (mdates.date2num(SP500_df['Adj Close'].index[end_point_index]),lowest_point_value), xytext=(-220, 390), 
            textcoords='offset points',size = 12)

plt.ylabel('Adj Close')
plt.xlabel(None)
plt.title(f"Closing Price for S&P500 ({Period})")
plt.tight_layout()
```
<br/>

Sample for plot data: Subplot<br/>
<img src="https://github.com/Ash-Tao/Sectors_Performance/blob/main/Output/Russian_Invasion_of_Ukraine/MA%20by%20Sectors%20(Russian_Invasion).png"><br/>

``` python
# inset 3 moving average data into the prerformance
ma_day = [10, 20, 50]
for ma in ma_day:
    for i, sector in enumerate(sector_list, 1):
        column_name = f"MA for {ma} days"
        prerformance[i-1][column_name] = prerformance[i-1]['Adj Close'].rolling(ma).mean()

# Create a subplots
fig, axes = plt.subplots(nrows=4, ncols=3)
plt.rc('font', size=18) 
fig.set_figheight(35)
fig.set_figwidth(50)
prerformance[0][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[0,0])
axes[0,0].set_xlabel(None)
axes[0,0].set_title(f'{sector_name[0]}')
prerformance[1][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[0,1])
axes[0,1].set_xlabel(None)
axes[0,1].set_title(f'{sector_name[1]}')
prerformance[2][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[0,2])
axes[0,2].set_xlabel(None)
axes[0,2].set_title(f'{sector_name[2]}')
prerformance[3][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[1,0])
axes[1,0].set_xlabel(None)
axes[1,0].set_title(f'{sector_name[3]}')
prerformance[4][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[1,1])
axes[1,1].set_xlabel(None)
axes[1,1].set_title(f'{sector_name[4]}')
prerformance[5][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[1,2])
axes[1,2].set_xlabel(None)
axes[1,2].set_title(f'{sector_name[5]}')
prerformance[6][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[2,0])
axes[2,0].set_xlabel(None)
axes[2,0].set_title(f'{sector_name[6]}')
prerformance[7][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[2,1])
axes[2,1].set_xlabel(None)
axes[2,1].set_title(f'{sector_name[7]}')
prerformance[8][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[2,2])
axes[2,2].set_xlabel(None)
axes[2,2].set_title(f'{sector_name[8]}')
prerformance[9][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[3,0])
axes[3,0].set_xlabel(None)
axes[3,0].set_title(f'{sector_name[9]}')
prerformance[10][['Adj Close', 'MA for 10 days', 'MA for 20 days', 'MA for 50 days']].plot(ax=axes[3,1])
axes[3,1].set_xlabel(None)
axes[3,1].set_title(f'{sector_name[10]}')
plt.savefig(f"MA by Sectors ({Period}).png")
```
<br/>

---
## The Answer I Got
__1.	Some sectors return to the start-point more quickly than others.__<br/>
Some sectors will recover better than others in a crash. Consumer Discretionary, Consumer Staples and Information Tech are the first to recover in several crashes. And Financials, often the market returns to the starting point, the sector's prices are still low.<br/>
<br/>
Due to the different causes of crashes, the correlation between the various sectors will also be different. Global events with a wide range of influence will make the correlation of each sector stronger, and the increase between the sectors will be more synchronized. Conversely, specific Events in the field have different effects on each sector. There is a specific pattern between it and the price.<br/>
<br/>

__2.	Trade volume makes the difference in returns (price change).__<br/>
Trade volume is a major factor affecting market prices. Daily Return and Daily Volume are closely related to each other. The peaks and deeps on Return & Volume always seem to appear in pairs. <br/>
<br/>
The turning point of volume often indicates the turning point of return. If the volume trend changes (increasing to decreasing, or decreasing to increasing), which means that the price is facing a reversal point, and it is a time to switch investment strategy.<br/>
<br/>

__3.	The previous price can help to make calls of buying or selling.__<br/>
We can predict price trend based on the previous price by reviewing the crossover on the moving average. The previous price can help smooth the price and judge the direction of the price through different Crossovers.<br/>
<br/>
It is time to buy if the price crosses above the moving average (or shorter-term MA crosses above the longer-term MA), and a sell signal if it is below.<br/>
<br/>

__4.	Correlations between sectors are different due to the different nature of global events that caused the crash.__<br/>
Most of the sectors have a relationship of Moderate or above during the crashes. Overall, the sector correlations in 2015 August Stock Market Selloff & Covid – 19 are more concentrated than in Bitcoin Crash & Russian Invasion of Ukraine.<br/>
<br/>
The correlations will be stronger during a crash if the events cause this crash is more globally, the prices of all sectors will fall more consistently.
<br/>
People's predictions regarding the future after the event will also affect the market.<br/>

---
## Files in This Repository
- [Sector.ipynb](https://github.com/Ash-Tao/Sectors_Performance/blob/main/Sector.ipynb)<br/>
- [Results and Conclusion](https://github.com/Ash-Tao/Sectors_Performance/tree/main/Results%20and%20Conclusion)<br/>
  - Documentation.docx<br/>
  - Presentation.pptx<br/>
- [sp500](https://github.com/Ash-Tao/Sectors_Performance/tree/main/sp500)<br/>
  - sp500_companies.csv<br/>
- [Output](https://github.com/Ash-Tao/Sectors_Performance/tree/main/Output)<br/>
  - Bitcoin_Crash<br/>
  - Covid_19<br/>
  - Russian_Invasion_of_Ukraine<br/>
  - Stock_Market_Selloff<br/>