<div align=”center”>

# MODULE 4 PROJECT- TIME SERIES ANALYSIS

# INVESTING IN RENEWABLE ENERGY

</div>


## Introduction

With an increasing sense of environment conservation and preservation, there has been immense research and development in the renewable energy sector. Renewable energy consumption is seen to grow exponentially each year for some of these sources. This leaves vast investing opportunities in the sector.

Analyzing the combined forecasts of energy consumption, cost, and returns helps the investors make an educated decision on the source of energy they want to invest in for both, long term and short term benefits.

This notebook shows a detailed analysis and comparison on the energy consumption of Hydroelectric power, Geothermal, Solar, Wind, and Wood Energy over the next 5 years

>**Hydroelectric Power:** Hydroelectric power is harnessed from the force of flowing water <br />

>**Geothermal Energy:** Geothermal energy is the thermal energy extracted fro the heat within the earth, contained in rocks, fluids, to as deep as from the magma or from the radioactive decay of natural elements found within the earth's crust. <br />

>**Solar Energy:** Solar Power is generated by harnessing the energy from the sunlight either directly by concentrating through lenses or indirectly by using the photovoltaic cells to convert this energy into electricity. <br />

>**Wind Energy:** Similar to hydroelectric power, Wind power is generated by utilizing the force generated by wind and converting it into electricity. <br />

>**Wood Energy:** Wood energy is generated by the combustion of wood and its derivatives and is used majorly for generating electricity and cooking.


## Methodology

Each source of energy, Hydroelectric, Geothermal, Solar, Wind, and Wood, is analyzed individually using visualization and forecasts through Auto Arima method. A monthly frequency was set for the analysis.

The current scenarios and Forecasts are then compared.

The Data used for this project consist of renewable energy consumption beginning from 1973 till 2020. Since some of these sectors have just emerged a few years ago, they are relatively new and their consumption data do not date back to 1973. These values have been replaced with 0 and have not been included in the individual analysis.


## Project Specific Functions

```
def seasonal_decomp(series):

    """
    gives the seasonal decomposition of a given series
    --------------------------------------------------
    Input:
    series (series)
    --------------------------------------------------
    Output:
    Seasonal decomposition plots
    """
    
    fig = plt.figure()  
    fig = seasonal_decompose(series).plot()  
    fig.set_size_inches(15, 12)
```

```
def test_stationarity(series, window, cutoff):
    
    """
    Tests stationarity using Dickey-Fuller Test
    -------------------------
    Input:
    series (series)
    window (int): Size of the moving window for rolling mean and rolling standard deviation calculation.
    cuttoff (float): cutoff for p-value to determine stationarity
    -------------------------
    Output:
    Displays if the series is stationary
    Displays the Dickey-Fuller Test Summary
    ROlling mean and Rolling Standard Deviation Plots
    """
    
    rolmean = series.rolling(window).mean()
    rolstd = series.rolling(window).std()
    
    fig = plt.figure(figsize = (12,8))
    ori = plt.plot(series, color = 'blue', label = 'Original')
    mean = plt.plot(rolmean, color = 'red', label = 'Rolling Mean')
    std = plt.plot(rolstd, color = 'black', label = 'Rolling Standard Deviation')
    plt.legend(loc = 'best')
    plt.title('Rolling Mean and Standard Deviation')
    plt.show()
    
    print('Dicky-Fuller Test:')
    dftest = adfuller(series)
    dfoutput = pd.Series(dftest[0:4], index=['Test Statistic','p-value','#Lags Used','#Observations Used'])
    for key,value in dftest[4].items():
        dfoutput[f'Critical Value {key}']=value
    if dfoutput['p-value']<cutoff:
        print('The Series is likely Stattionary')
    else:
        print('The Series is likely Non-Stationary')
    print(dfoutput)
 ```
 
 ```
def shift(series,shift_no):
    
    """
    Shifts the time series to the given number of places
    -------------------
    Input:
    Series (Series)
    shift_no (int): the number of places that the series needs to shift
    """
    
    diff = series-series.shift(shift_no)
    diff = diff.dropna(inplace=False)
    return diff
 ```
 
 ```
 def p_acf(series):
    
    """
    Builds the autocorrelation and partial correlation plots of a given series
    ---------------
    Input:
    Series (series)
    ---------------
    Output:
    statsmodel autocorrelation and partial correlation plot
    """
    
    fig = plt.figure(figsize=(12,8))
    ax1 = fig.add_subplot(211)
    fig = sm.graphics.tsa.plot_acf(series, ax=ax1) 
    ax2 = fig.add_subplot(212)
    fig = sm.graphics.tsa.plot_pacf(series, ax=ax2)
 ```
 
 ```def sarimax(df,p,d,q,P,D,Q,s):
    
    """Grid searches the best p,d,q based on the AIC values
    -----------------
    Input:
    df (DataFrame): Data required to model SARIMAX model
    p (range): AR component for ARIMA modelling
    d (range): difference component for ARIMA modelling
    q (range): MA component for ARIMA modelling
    P (range): AR component for SARIMA modelling
    D (range): difference component for SARIMA modelling
    Q (range): MA component for SARIMA modelling
    S (range): Seasonal component for SARIMA modelling
    ------------------
    Output:
    Best order and seasonal order corresponding the lowest AIC value
    """
    
    pdq = list(itertools.product(p,d,q))
    PDQs = [(x[0],x[1],x[2],s) for x in list(itertools.product(P,D,Q))]
    
    ans = []
    for comb in pdq:
        for combs in PDQs:
            try:
                mod = sm.tsa.statespace.SARIMAX(df, order = comb, seasonal_order=combs,
                                                 enforce_stationarity=False, enforce_invertibility=False)
                output = mod.fit()
                ans.append([comb,combs,output.aic])
                print(f'ARIMA {comb} x {combs}12 : AIC Calculated ={output.aic}')
            except:
                continue
    
    ans_df=pd.DataFrame(ans, columns=['pdq','PDQs','AIC'])
#     ans_df.AIC.plot(figsize=(15,6))
    
    return(ans_df.loc[ans_df['AIC'].idxmin])
```

```
def arima(df,arima,sarima):
    
    """Fits statsmodel SARIMAX model given order and seasonal order
    --------------
    Input:
    df (DataFrame): dataframe to be fit in the model
    arima (tuple): order for ARIMA model
    sarima (tuple):seasonal order for SARIMA model
    --------------
    Output
    Statsmodel SARIMAX model
    """
    
    arima = sm.tsa.statespace.SARIMAX(df, order = arima, seasonal_order=sarima,
                                    enforce_stationarity=False, enforce_invertibility=False)
    output = arima.fit()
    print(output.summary().tables[1])
    return output
```

```
def auto_arima(df,df_train, df_test,n_rows,seasonal=True, m=12):
    
    """Builds the best arima model from the p,q range of (0,5)
    -----------------
    Input:
    df (DataFrame)
    df_train (DataFrame): Train Split data
    df_test (DtataFrame): Test split data
    n_rows (integer): the value at which the train test split was made
    seasonal (bool): Seasonality to include or not
    m (int): seasonality
    -----------------
    Output:
    Best Sarima Model
    Best model summary
    prediction plot
    best model order and seasonal order
    """
    
    model = pm.auto_arima(df_train, seasonal=seasonal, m=m)

    # make your forecasts
    forecasts = model.predict(df_test.shape[0])  # predict N steps into the future
    
    print('FORECAST VISUALIZATION'+'\n')

    # Visualize the forecasts (blue=train, green=forecasts)
    x = np.arange(df.shape[0])
    plt.plot(x[:n_rows], df_train, c='blue')
    plt.plot(x[n_rows:], df_test, c='blue')
    plt.plot(x[n_rows:], forecasts, c='green')
    plt.show()
    
    # Model Summary
    print('\n\n','--'*20, '\n\n', 'MODEL SUMMARY','\n')
    
    display(model.summary())
    
    # Model Plot Diagnostics
    print('\n\n','--'*20, '\n\n', 'MODEL PLOT DIAGNOSTICS','\n')
    model.plot_diagnostics();
    
    #order and seasonal order
    print('\n\n','--'*20, '\n\n', 'ORDER, SEASONAL ORDER','\n')
    print(f'order = {model.order}')
    print(f'seasonal order = {model.seasonal_order}')
    
    return model.order, model.seasonal_order
```

```
def forecast(model,df,forecast_start,steps=60,figsize=(15,8),color_conf='g',alpha_conf=0.3):
    
    """gives a specified step dynamic forecast for a statsmodel SARIMAX model
    --------------
    Input:
    model : Statsmodel SARIMAX model
    df (DataFrame)
    forecast_start (str): date in the format 'YYYY-MM-DD' where the forecast should start
    steps (int): steps into the future the forecasts should be created. Default is 60
    figsize (tuple): matplotlib figsize for the forecast output plot. default is (15,8)
    color_conf (str): color of the confidence interval. default is g
    alpha_conf (int)
    --------------
    Output:
    Matplotlib Forecast plot with confidence interval
    """
    
    forecast = model.get_forecast(steps=steps)
    ax = df.plot(label='Observed', figsize=figsize)
    forecast.predicted_mean.plot(label='Dynamic Forecast', ax=ax)

    ax.fill_between(forecast.conf_int().index,
                   forecast.conf_int().iloc[:,0],
                   forecast.conf_int().iloc[:,1],
                   color=color_conf,alpha=alpha_conf)

    ax.fill_betweenx(ax.get_ylim(), pd.to_datetime(forecast_start), forecast.predicted_mean.index[-1], alpha=.1, zorder=-1)

    plt.legend()
    plt.show()
```


## Current Energy Consumption Trends

The consumption rates of all sources were first compared to get a better understanding of the current trends.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Renewable_Current_All.png">
<div align="center"> Figure 1: Current Trends in Renewable Energy Consumption </div>
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Renewable_Current_All_Y.png">
<div align="center"> Figure 2: Current Trends in Renewable Energy Consumption (Annual)</div>
<br />

Figure 1 and 2 illustrate the present scenario of the various total renewable energy consumptions beginning 1973. It is seen that hydroelectric power consumption has always been plateaued at the heighest levels averaging at about 250 Trillion BTU. Wood energ consumption started increasing up till 1980 and then plateaued at 175 Trillion BTU. Geo Thermal Energy consumption is only slightly increasing, almost constant at the lowest value. currently at about 10 Trillion BTU. Wind and Solar Energy have only recently gained popularity and are growing at tremendous rates compared to the other sources of renewables. Large amount of research and innovation is seen in these sectors along with an increase in the number of residential and commercial projects.
<br />

Each sector further show seasonality in their consumption plots with a constant rise and fall in the consumption per year. Larger seasonality is also seen with hydroelectric and wood energy consumption every four to 5 years.


## Time Series Analysis of the Individual Energy Sectors

To minimize the risks in investing, it is paramount to understand the nature of an industry in the near future. Time series analysis  using ARIMA is hence performed on each consumption series, the results to which are then compared to paint a clear picture.

### Hydroelectric Power Consumption

Figure 3 shows the seasonal decompose of hydroelectric power consumption. No clear trend is seen except for a larger seasonality every 5 years. This kind of trend generally means that our series is stationaty, which is one of the requirements of ARIMA modelling. Stationarity was confirmed using the Dickey-Fuller stationarity test. The seasonal plot shows a clear annual seasonality.
<br />

<p align="center">
    <img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Seasonal_Decomp_Hyele.png">
    *Figure example*
    
<div align="center"> Figure 3: Seasonal Decompose - Hydroelectric Power Consumption </div>
</p>
<br />

Auto correlation and correlation plots (Figure 4) were then used to specify the range of p (AR component) and q (MA component) for ARIMA modelling.
<br />

<center><img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/PAC-AC-Hyele.png"></center>
<div align="center"> Figure 4: Partial and Autocorrelation Plots - Hydroelectric Power Consumption </div>
<br />

Based on these observations the range of p and q are set at (0,5). An ARIMA model was fit using pmdarima's auto_arima function. The seasonal component was set at 60. The output of this model is shown in Figure 5.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/hyele_1.png">
<div align="center" font-style="italic"> Figure 5: Hydroelectric Power Consumption - Base Model </div>
<br />

To tune the model for a better fit, the data was sliced to include only the recent trends, beginning 2002. Figure 6 shows the ARIMA model then fit using seasonality component of 48.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/hyele_2.png">
<div align="center" font-style="italic"> Figure 6: Hydroelectric Power Consumption - Model 2 </div>
<br />

5 year forecast thus generated using this model is shown in Figure 7. It is seen that the consumption rates continue to plateau at fixed amounts with yearly seasonality.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/hyele_forecast.png">
<div align="center" font-style="italic"> Figure 7: Hydroelectric Power Consumption - 5 year Forecast </div>

### Geothermal Energy Consumption

Figure 8 shows the seasonal decompose of geothermal energy consumption. The trend shows that geothermal energy consumption was growing at good rates in the begenning but have plateaued over the past couple of years, while the seasonal plot shows a clear annual seasonality.
<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Seasonal_Decomp_Geo.png">
<div align="center"> Figure 8: Seasonal Decompose - Geothermal Energy Consumption </div>
<br />

The Dicky-Fuller Stationarity test resulted in the series being non-stationary. The first difference was then tested as being stationary. Hence, d for ARIMA modelling would be 1, which was also seen in the results of auto arima.

<br />

Auto correlation and partial correlation plots (Figure 9) of the differenced series was then used to to specify the range of p and q for ARIMA modelling. These were set at (0,5) for both p and q.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/PAC-AC-Geo.png">
<div align="center"> Figure 9: Partial and Autocorrelation Plots - Geothermal Energy Consumption </div>
<br />
Using these values, an ARIMA model was then fit with the seasonal component set at 12. The output of this model is shown in Figure 10.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/geo_1.png">
<div align="center"> Figure 10: Geothermal Energy Consumption Model Output </div>
<br />
Figure 11 shows the 5 year forecasts generated using this model. It is seen that the consumption rates continue to plateau at fixed amounts with yearly seasonality. The forecasts are however less trustable for the years farther down as the confidence interval increases.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/geo_forecast.png">
<div align="center"> Figure 11: Geothermal Energy Consumption - 5 year Forecast </div>

### Solar Energy Consumption

Figure 12 shows the seasonal decompose of solar energy consumption. The trend shows that solar energy consumption was only recently recognized and has been growing at tremendous rates after that. The seasonal plot shows a clear annual seasonality. The data was modified to exclude the years where there was no data available.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Seasonal_Decomp_Solar.png">
<div align="center"> Figure 12: Seasonal Decompose - Solar Energy Consumption </div>
<br />
Since there was a sudden increase in the rates of consumption for solar energy, it was paramount to feed more of the recent data in the training set for modelling. Figure 13 shows the output of the ARIMA model thus fit with seasonal component set at 12.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/solar_1.png">
<div align="center"> Figure 13: Solar Energy Consumption Model Output </div>
<br />
Figure 14 shows the 5 year forecasts generated using this model. It is seen that the consumption rates continue to rise at elevated rates and continued yearly seasonality.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/solar_forecast.png">
<div align="center"> Figure 14: Solar Energy Consumption - 5 year Forecast </div>

### Wind Energy Consumption

Figure 15 shows the seasonal decompose of wind energy consumption. The trend shows that similar to solar, wind energy consumption was also only recently recognized and has been growing at tremendous rates after that. The seasonal plot shows a clear annual seasonality. The data was modified to exclude the years where there was no data available.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Seasonal_Decomp_Wind.png">
<div align="center"> Figure 15: Seasonal Decompose - Wind Energy Consumption </div>
<br />
Since there was a sudden increase in the rates of consumption for solar energy, it was paramount to feed more of the recent data in the training set for modelling. Figure 16 shows the output of the ARIMA model thus fit with seasonal component set at 12.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/wind_1.png">
<div align="center"> Figure 16: Wind Energy Consumption Model Output </div>
<br />
Figure 17 shows the 5 year forecasts generated using this model. It is seen that the consumption rates continue to rise at elevated rates and continued yearly seasonality.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Wind_forecast.png">
<div align="center"> Figure 17: Wind Energy Consumption - 5 year Forecast </div>

### Wood Energy Consumption

Figure 18 shows the seasonal decompose of geothermal energy consumption. The trend seems pretty stationary. Dickey-Fuller test for stationarity however showed otherwise. The first difference was then tested as being stationary. Hence, d for ARIMA modelling would be 1, which was also seen in the results of auto arima.The seasonal plot shows annual seasonality. There was a boom in the sector during the 1980's uptill 1990, afterwhich the industry fell and has since then remained more or less stagnant at 180 Trillion BTU.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Seasonal_Decomp_Wood.png">
<div align="center"> Figure 18: Seasonal Decompose - Wood Energy Consumption </div>
<br />
Auto correlation and partial correlation plots (Figure 19) of the differenced series was then used to to specify the range of p and q for ARIMA modelling. These were set at (0,5) for both p and q.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/PAC-AC-Wood.png">
<div align="center"> Figure 19: Partial and Autocorrelation Plots - Wood Energy Consumption </div>
<br />
Since the trend has not changed much within the last decade, only the data begenning 2010 is used for this model so as to minimize the influence of the golden decade for this sector.ARIMA model was then fit with the seasonal component set at 12. The output of this model is shown in Figure 20.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/wood_1.png">
<div align="center"> Figure 20: Wood Energy Consumption Model Output </div>
<br />
Figure 21 shows the 5 year forecasts generated using this model. It is seen that the consumption rates continue to plateau at fixed amounts with yearly seasonality. The forecasts are however less trustable for the years farther down as the confidence interval increases.

<br />

<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/wood_forecast.png">
<div align="center"> Figure 21: Wood Energy Consumption - 5 year Forecast </div>


## Energy Consumption Forecast Comparison

All forecasts were compared to visually analyse the comparative growth in each sector. This is demonstrated in Figures 22 and 23.
<br />
<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Forecast_Compare.png">
<div align="center"> Figure 22: Comparative Analysis </div>
<br />
<img align="center" src="https://raw.githubusercontent.com/NehaP92/dsc-mod-4-project-v2-1-onl01-dtsc-pt-041320/master/Forecast_Compare_Annual.png">
<div align="center"> Figure 23: Comparative Analysis (Annual) </div>
<br />
Geothermal Energy, Wood Energy, and Hydroelectric Power Consumption show stagnant or minimal growth around the plateaued values, while Solar and Wind Energy Consumptions continue to rise significantly surpassing the values of Hydroelectric Power and Wood Energy consumptions. Due to seasonality, during few months, consumption for these may cross over for a brief moment should the forecast be extended a few years down the line. The slope of increase for these consumptions are almost parallel with solar demonstrating a slightly steeper slope, indicating a possible overlap should we conduct a 10 year forecast. 


## Conclusions

Current Trend:
- All energy consumption trends show seasonal variations per month.
- Current trends show that Hydroelectric Power Consumption has been dominating the market for decades now.
- Solar and Wind Energy Consumption is rising exponentially and shows promising growth and opportunities
- Geothermal energy consumption is almost stagnant with continuing to grow with only a minimum margin.
- Wood energy consumption has also been stagnant but significant, with numbers only a little less than Hydroelectric Power consumption. Both these sources have plateaued over time and continue the same trend with no significant growth over time.

Forecast:
- Geothermal Energy, Wood Energy, and Hydroelectric Power Consumption is predicted to remain stagnant or show minimal growth around the plateaued values.
- Solar and Wind Energy Consumptions continue to rise significantly surpassing the values of Hydroelectric Power and Wood Energy consumptions.
- Due to seasonality, during few months, consumption for these may cross over for a brief moment should the forecast be extended a few years down the line.
- The slope of increase for these consumptions are almost parallel with solar demonstrating a slightly steeper slope, indicating an overlap in the future years. 

Based on research and the current market, there has been enormous growth in Solar Energy Production. For a clear recommendation it is therefore important to analyze and take into consideration the demand (consumption), supply (Production), cost of production, and the price of the energy source before making any investment decision.


## Recommendations

Based on the type of investment that an investor wishes to make, below are a few recommendations following this time series analysis:
- Looking at the forecasts, investors looking for mid term investments should look into investing in wind farms since it has tremendous opportunities and seen to grow and exceed the consumptions of all other energy sources.
- For short term investments, the investors should take into consideration the seasonal variations which is common to all energy sources. These investors should look into investing in Wind Energy, since in the consumption rates for Wind would cross over those of Hydroelectric power in the later half of 2020.
- Long term investors are strongly recommended to look into the market trends for Energy production in solar and wind. Since there has been tremendous growth and promotion of solar farms and residential/commercial solar projects, the trend is likely to increase the slope of solar energy consumption with it crossing over wind energy consumption sooner than later.


## Future Work

- Conduct a time series analysis for production, cost and returns related to each source of renewable energy for a better insights into the investing opportunities
- Build an interactive dashboard as a tool for the investors to recommend the best source of energy to invest in based on the amount and period of investment. This tool will take into consideration all four aspects, Demand, Supply, Cost and Returns to recommend the most profitable sector to invest in.