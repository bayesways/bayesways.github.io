---
layout: post
title: Recreating the US birthday analysis with Prophet
giscus_comments: true
date: 2023-03-14
related_posts: false
---

## Background

In this post I will try to model the number of births in the US, using seasonal effects, day of week effects and special holiday effects. 
The original analysis used Gaussian Processes, it was included in BDA3 (on the cover actually). In the first version of the book the model was too big to be fit with Stan, so it was fit with a different package called `GPStuff`. [Here](https://research.cs.aalto.fi/pml/software/gpstuff/demo_births.shtml) is the code that created the materials of the book. 

Since then Stan has improved further and it looks like it is now possible to fit the original model Stan, as explained in this recent [post](https://avehtari.github.io/casestudies/Birthdays/birthdays.html) by Aki Vehtari.

As described in the book, and a series of posts such as [this](https://statmodeling.stat.columbia.edu/2016/05/18/birthday-analysis-friday-the-13th-update/) on Andrew Gelman's blog, the question was whether there are excess births on Valentine's day and fewer births on Halloween. This question provided an excuse to examine the effect of any holiday in the US, and more generally any day of the year. It also served as an excuse to demonstrate the use of Gaussian Processes as a time series decomposition technique.

Here is the headline chart from that analys (recreated here from the gpstuff [page](https://research.cs.aalto.fi/pml/software/gpstuff/demo_births.shtml)): 
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2023-03-14-recreating-the-US-birthday-analysis/birthday_files/births_pic2.png" class="img-fluid rounded z-depth-1" %} </div>


## Plan for this post
Here I try to the same analysis but using a simpler additive model based on Fourier Transforms, instead of Gaussian Processes. 
I am also using it as an excuse to learn how [Prophet](https://facebook.github.io/prophet/) works, a package for time series forecasting implemented by a team at facebook.

Tha package is pretty simple to use and fast to fit. I was able to get most of the paterns right with little experimentation, though not completely right. The ease of use and the ability to add custom components makes Prophet a great tool. 
The most obvious drawback is that it's made to accommodate daily data and there is no easy way to generalize to other frequencies. 


## Setup

Load packages

```python
import pandas as pd
from prophet import Prophet
import altair as alt
import holidays
from prophet.plot import add_changepoints_to_plot
alt.data_transformers.enable('default', max_rows=None)
```

Get data
```python
df = pd.read_csv('https://raw.githubusercontent.com/avehtari/casestudies/master/Birthdays/data/births_usa_1969.csv')
df['ds'] = pd.to_datetime(df.year.astype(str)+'-'+df.month.astype(str)+'-'+df.day.astype(str))
df.rename({'births':'y'}, axis=1, inplace=True)
df = df[['ds','y']]
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ds</th>
      <th>y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1969-01-01</td>
      <td>8486</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1969-01-02</td>
      <td>9002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1969-01-03</td>
      <td>9542</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1969-01-04</td>
      <td>8960</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1969-01-05</td>
      <td>8390</td>
    </tr>
  </tbody>
</table>
</div>

## Model Definition
Define the standard model and add a custom effect that I call "each_day". 
It has `period` 365.25 because I want to model each day of the year. It also has a high `fourier_order` to make it very flexible. I experimented with lower values and found that I need a fairly high value to pick up the signals I am looking for. More experimentation is needed to pick this value though.

```python
m = Prophet(changepoint_prior_scale = 0.01, yearly_seasonality=10)
## add effect for day of the year effect for more long term patters
m.add_seasonality(name='each_day_long_term', period=365.25, fourier_order=4)
## add effect for day of the year: 
m.add_seasonality(name='each_day', period=365.25, fourier_order=100)
m.fit(df)
```

    17:03:30 - cmdstanpy - INFO - Chain [1] start processing
    17:03:59 - cmdstanpy - INFO - Chain [1] done processing





    <prophet.forecaster.Prophet at 0x7fe099e03910>


Fit the model to data

```python
future = m.make_future_dataframe(periods=1)
future.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ds</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7301</th>
      <td>1988-12-28</td>
    </tr>
    <tr>
      <th>7302</th>
      <td>1988-12-29</td>
    </tr>
    <tr>
      <th>7303</th>
      <td>1988-12-30</td>
    </tr>
    <tr>
      <th>7304</th>
      <td>1988-12-31</td>
    </tr>
    <tr>
      <th>7305</th>
      <td>1989-01-01</td>
    </tr>
  </tbody>
</table>
</div>


Generate estimates and forecasts


```python
forecast = m.predict(future)
forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail()
```






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ds</th>
      <th>yhat</th>
      <th>yhat_lower</th>
      <th>yhat_upper</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7301</th>
      <td>1988-12-28</td>
      <td>11680.646441</td>
      <td>11239.300965</td>
      <td>12094.109632</td>
    </tr>
    <tr>
      <th>7302</th>
      <td>1988-12-29</td>
      <td>11724.885805</td>
      <td>11294.168795</td>
      <td>12215.288371</td>
    </tr>
    <tr>
      <th>7303</th>
      <td>1988-12-30</td>
      <td>11502.343208</td>
      <td>11071.599362</td>
      <td>11957.702909</td>
    </tr>
    <tr>
      <th>7304</th>
      <td>1988-12-31</td>
      <td>9078.021852</td>
      <td>8622.992957</td>
      <td>9533.391236</td>
    </tr>
    <tr>
      <th>7305</th>
      <td>1989-01-01</td>
      <td>7960.215673</td>
      <td>7521.253713</td>
      <td>8409.581555</td>
    </tr>
  </tbody>
</table>
</div>



Create the basic plot of the underlying trend.
I chose to visualize the change points for more clarity. The exacat number of points is a choice that requires callibration but I think it does not affect my problem too much so I move on.

```python

fig1 = m.plot(forecast)
a = add_changepoints_to_plot(fig1.gca(), m, forecast)
```


    
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2023-03-14-recreating-the-US-birthday-analysis/birthday_files/birthday_7_0.png" class="img-fluid rounded z-depth-1" %} </div>

Create the plot of all the different components. This is an automatic output from `Prophet` and it is fairly close to the headline chart of the original analysis. But we are missing the key chart of the day of year (see below).

```python
fig2 = m.plot_components(forecast)

```
    
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2023-03-14-recreating-the-US-birthday-analysis/birthday_files/birthday_8_0.png" class="img-fluid rounded z-depth-1" %} </div>
    

And finally here we create the day of the year plot we were aiming for.
```python
day_of_year_effect = forecast[['ds','each_day']].copy()
day_of_year_effect['indicative_day'] = '1999' +'-' + day_of_year_effect['ds'].dt.month.astype(str) +'-' + day_of_year_effect['ds'].dt.day.astype(str)
day_of_year_effect = day_of_year_effect.groupby('indicative_day')[['each_day']].mean().reset_index()
day_of_year_effect
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>indicative_day</th>
      <th>each_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1999-1-1</td>
      <td>-1.301688</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1999-1-10</td>
      <td>-0.163488</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1999-1-11</td>
      <td>-0.017994</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1999-1-12</td>
      <td>-0.053865</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1999-1-13</td>
      <td>-0.081956</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>361</th>
      <td>1999-9-5</td>
      <td>0.077351</td>
    </tr>
    <tr>
      <th>362</th>
      <td>1999-9-6</td>
      <td>0.131178</td>
    </tr>
    <tr>
      <th>363</th>
      <td>1999-9-7</td>
      <td>0.119668</td>
    </tr>
    <tr>
      <th>364</th>
      <td>1999-9-8</td>
      <td>0.314853</td>
    </tr>
    <tr>
      <th>365</th>
      <td>1999-9-9</td>
      <td>0.463697</td>
    </tr>
  </tbody>
</table>
<p>366 rows × 2 columns</p>
</div>




```python
fig3 = alt.Chart(day_of_year_effect).mark_line().encode(
    x='indicative_day:T',
    y='each_day'
)
```




```python
holiday_data = pd.DataFrame.from_dict(holidays.US(years=forecast['ds'].dt.year.values).items())
holiday_data.columns = ['ds', 'holiday']
holiday_data['ds'] = pd.to_datetime(holiday_data['ds'])

ignore_observed_dates = True
ignore_index = holiday_data['holiday'][holiday_data['holiday'].str.contains('Observed')].index
holiday_data_without_observed = holiday_data[~holiday_data.index.isin(ignore_index)]
if ignore_observed_dates:
    holiday_data = holiday_data_without_observed
```


```python
holiday_effect = forecast[['ds', 'each_day']].merge(holiday_data, on='ds', how='outer').groupby('holiday').agg({'ds':'first', 'each_day':'mean'}).reset_index()
holiday_effect.rename({'each_day':'effect size'}, axis=1, inplace=True)
holiday_effect['indicative_day'] = '1999' +'-' + holiday_effect['ds'].dt.month.astype(str) +'-' + holiday_effect['ds'].dt.day.astype(str)

```


```python
fig4 = alt.Chart(holiday_effect).mark_text().encode(
    x='indicative_day:T',
    y='effect size',
    text='holiday'
)
```


```python
(fig3 + fig4 ).properties(width=600, height=250)
```





<div id="altair-viz-5f10a9cb885a4675a84cfcada1776064"></div>
<script type="text/javascript">
  var VEGA_DEBUG = (typeof VEGA_DEBUG == "undefined") ? {} : VEGA_DEBUG;
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-5f10a9cb885a4675a84cfcada1776064") {
      outputDiv = document.getElementById("altair-viz-5f10a9cb885a4675a84cfcada1776064");
    }
    const paths = {
      "vega": "https://cdn.jsdelivr.net/npm//vega@5?noext",
      "vega-lib": "https://cdn.jsdelivr.net/npm//vega-lib?noext",
      "vega-lite": "https://cdn.jsdelivr.net/npm//vega-lite@4.17.0?noext",
      "vega-embed": "https://cdn.jsdelivr.net/npm//vega-embed@6?noext",
    };

    function maybeLoadScript(lib, version) {
      var key = `${lib.replace("-", "")}_version`;
      return (VEGA_DEBUG[key] == version) ?
        Promise.resolve(paths[lib]) :
        new Promise(function(resolve, reject) {
          var s = document.createElement('script');
          document.getElementsByTagName("head")[0].appendChild(s);
          s.async = true;
          s.onload = () => {
            VEGA_DEBUG[key] = version;
            return resolve(paths[lib]);
          };
          s.onerror = () => reject(`Error loading script: ${paths[lib]}`);
          s.src = paths[lib];
        });
    }

    function showError(err) {
      outputDiv.innerHTML = `<div class="error" style="color:red;">${err}</div>`;
      throw err;
    }

    function displayChart(vegaEmbed) {
      vegaEmbed(outputDiv, spec, embedOpt)
        .catch(err => showError(`Javascript Error: ${err.message}<br>This usually means there's a typo in your chart specification. See the javascript console for the full traceback.`));
    }

    if(typeof define === "function" && define.amd) {
      requirejs.config({paths});
      require(["vega-embed"], displayChart, err => showError(`Error loading script: ${err.message}`));
    } else {
      maybeLoadScript("vega", "5")
        .then(() => maybeLoadScript("vega-lite", "4.17.0"))
        .then(() => maybeLoadScript("vega-embed", "6"))
        .catch(showError)
        .then(() => displayChart(vegaEmbed));
    }
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "layer": [{"data": {"name": "data-d373b8b478dabceb4050e1a3a9f78682"}, "mark": "line", "encoding": {"x": {"field": "indicative_day", "type": "temporal"}, "y": {"field": "each_day", "type": "quantitative"}}}, {"data": {"name": "data-25713872bd7ea71ecc827a8ebbaa6350"}, "mark": "text", "encoding": {"text": {"field": "holiday", "type": "nominal"}, "x": {"field": "indicative_day", "type": "temporal"}, "y": {"field": "effect size", "type": "quantitative"}}}], "height": 250, "width": 600, "$schema": "https://vega.github.io/schema/vega-lite/v4.17.0.json", "datasets": {"data-d373b8b478dabceb4050e1a3a9f78682": [{"indicative_day": "1999-1-1", "each_day": -935.7558852651148}, {"indicative_day": "1999-1-10", "each_day": -167.22771517510358}, {"indicative_day": "1999-1-11", "each_day": 41.69885106293092}, {"indicative_day": "1999-1-12", "each_day": 36.745962343884486}, {"indicative_day": "1999-1-13", "each_day": -43.683339049571586}, {"indicative_day": "1999-1-14", "each_day": -7.256157432204264}, {"indicative_day": "1999-1-15", "each_day": -0.49051032887007506}, {"indicative_day": "1999-1-16", "each_day": -97.8576898381739}, {"indicative_day": "1999-1-17", "each_day": -95.74960389957083}, {"indicative_day": "1999-1-18", "each_day": 31.86298345941254}, {"indicative_day": "1999-1-19", "each_day": 57.695956293300696}, {"indicative_day": "1999-1-2", "each_day": -1000.2816025419343}, {"indicative_day": "1999-1-20", "each_day": -37.52892311828231}, {"indicative_day": "1999-1-21", "each_day": -72.49957526527263}, {"indicative_day": "1999-1-22", "each_day": -58.84118026132844}, {"indicative_day": "1999-1-23", "each_day": -103.9915377566982}, {"indicative_day": "1999-1-24", "each_day": -121.90630998182658}, {"indicative_day": "1999-1-25", "each_day": -39.62069375884529}, {"indicative_day": "1999-1-26", "each_day": -1.0023441706092113}, {"indicative_day": "1999-1-27", "each_day": -77.13314248376221}, {"indicative_day": "1999-1-28", "each_day": -130.41135957893803}, {"indicative_day": "1999-1-29", "each_day": -125.7779418372298}, {"indicative_day": "1999-1-3", "each_day": -381.8294907262302}, {"indicative_day": "1999-1-30", "each_day": -169.9086459625696}, {"indicative_day": "1999-1-31", "each_day": -204.43159629837675}, {"indicative_day": "1999-1-4", "each_day": -9.986476979049794}, {"indicative_day": "1999-1-5", "each_day": -114.06712279424126}, {"indicative_day": "1999-1-6", "each_day": -172.715689343198}, {"indicative_day": "1999-1-7", "each_day": -83.18535497250606}, {"indicative_day": "1999-1-8", "each_day": -158.7035984458442}, {"indicative_day": "1999-1-9", "each_day": -290.5893367021596}, {"indicative_day": "1999-10-1", "each_day": 216.523669372401}, {"indicative_day": "1999-10-10", "each_day": 117.3328085314949}, {"indicative_day": "1999-10-11", "each_day": 88.35624709103766}, {"indicative_day": "1999-10-12", "each_day": 16.04608785597697}, {"indicative_day": "1999-10-13", "each_day": 6.526906500616962}, {"indicative_day": "1999-10-14", "each_day": 53.39476114674765}, {"indicative_day": "1999-10-15", "each_day": 63.07218199220646}, {"indicative_day": "1999-10-16", "each_day": 20.402022251105972}, {"indicative_day": "1999-10-17", "each_day": -18.135344681859586}, {"indicative_day": "1999-10-18", "each_day": -32.60097228834169}, {"indicative_day": "1999-10-19", "each_day": -28.336275378597982}, {"indicative_day": "1999-10-2", "each_day": 158.6405460165194}, {"indicative_day": "1999-10-20", "each_day": -3.4040688263262053}, {"indicative_day": "1999-10-21", "each_day": 5.954273215563765}, {"indicative_day": "1999-10-22", "each_day": -41.53121983548594}, {"indicative_day": "1999-10-23", "each_day": -95.10595378887363}, {"indicative_day": "1999-10-24", "each_day": -78.10830554809392}, {"indicative_day": "1999-10-25", "each_day": -24.04854188832745}, {"indicative_day": "1999-10-26", "each_day": -10.512970289865832}, {"indicative_day": "1999-10-27", "each_day": -28.02268704961359}, {"indicative_day": "1999-10-28", "each_day": -45.60421816560179}, {"indicative_day": "1999-10-29", "each_day": -96.86609294818065}, {"indicative_day": "1999-10-3", "each_day": 126.23663953598286}, {"indicative_day": "1999-10-30", "each_day": -181.26760225153797}, {"indicative_day": "1999-10-31", "each_day": -211.91415618802344}, {"indicative_day": "1999-10-4", "each_day": 120.98991453138781}, {"indicative_day": "1999-10-5", "each_day": 125.36635526706861}, {"indicative_day": "1999-10-6", "each_day": 113.44684473384264}, {"indicative_day": "1999-10-7", "each_day": 69.73656736317585}, {"indicative_day": "1999-10-8", "each_day": 37.752793332370814}, {"indicative_day": "1999-10-9", "each_day": 70.23275949818023}, {"indicative_day": "1999-11-1", "each_day": -155.01260345045887}, {"indicative_day": "1999-11-10", "each_day": 17.29787810908705}, {"indicative_day": "1999-11-11", "each_day": -6.843439626582186}, {"indicative_day": "1999-11-12", "each_day": -23.749232274863186}, {"indicative_day": "1999-11-13", "each_day": -42.55863835824413}, {"indicative_day": "1999-11-14", "each_day": -25.552784347272162}, {"indicative_day": "1999-11-15", "each_day": 43.27981038445536}, {"indicative_day": "1999-11-16", "each_day": 91.17693161805903}, {"indicative_day": "1999-11-17", "each_day": 87.55404360335119}, {"indicative_day": "1999-11-18", "each_day": 108.91152450919148}, {"indicative_day": "1999-11-19", "each_day": 166.72194872899394}, {"indicative_day": "1999-11-2", "each_day": -84.3158188996765}, {"indicative_day": "1999-11-20", "each_day": 153.2765087592175}, {"indicative_day": "1999-11-21", "each_day": 38.41111842980369}, {"indicative_day": "1999-11-22", "each_day": -83.55608147912676}, {"indicative_day": "1999-11-23", "each_day": -173.67359620616935}, {"indicative_day": "1999-11-24", "each_day": -268.4098140222778}, {"indicative_day": "1999-11-25", "each_day": -349.8469678934718}, {"indicative_day": "1999-11-26", "each_day": -381.77104866713387}, {"indicative_day": "1999-11-27", "each_day": -378.8818489453457}, {"indicative_day": "1999-11-28", "each_day": -308.47405438490966}, {"indicative_day": "1999-11-29", "each_day": -112.2950796322184}, {"indicative_day": "1999-11-3", "each_day": -51.15438119287971}, {"indicative_day": "1999-11-30", "each_day": 93.39884758758122}, {"indicative_day": "1999-11-4", "each_day": -40.16541555801091}, {"indicative_day": "1999-11-5", "each_day": -48.758767581053874}, {"indicative_day": "1999-11-6", "each_day": -73.51578810629715}, {"indicative_day": "1999-11-7", "each_day": -64.387379747513}, {"indicative_day": "1999-11-8", "each_day": -7.334981752257742}, {"indicative_day": "1999-11-9", "each_day": 31.053482371791763}, {"indicative_day": "1999-12-1", "each_day": 112.47590964582164}, {"indicative_day": "1999-12-10", "each_day": -81.19607809580974}, {"indicative_day": "1999-12-11", "each_day": -26.768199999878128}, {"indicative_day": "1999-12-12", "each_day": -47.33761731683906}, {"indicative_day": "1999-12-13", "each_day": -88.62414336426346}, {"indicative_day": "1999-12-14", "each_day": 43.72397490997906}, {"indicative_day": "1999-12-15", "each_day": 249.87720076969308}, {"indicative_day": "1999-12-16", "each_day": 291.6369465109428}, {"indicative_day": "1999-12-17", "each_day": 266.4623934185708}, {"indicative_day": "1999-12-18", "each_day": 365.09652852151504}, {"indicative_day": "1999-12-19", "each_day": 428.66055376524065}, {"indicative_day": "1999-12-2", "each_day": -7.037447635432182}, {"indicative_day": "1999-12-20", "each_day": 291.3668888960177}, {"indicative_day": "1999-12-21", "each_day": 113.25064898795783}, {"indicative_day": "1999-12-22", "each_day": -110.39582213350778}, {"indicative_day": "1999-12-23", "each_day": -683.6444298294327}, {"indicative_day": "1999-12-24", "each_day": -1395.7989591043115}, {"indicative_day": "1999-12-25", "each_day": -1454.3236428974549}, {"indicative_day": "1999-12-26", "each_day": -677.4919782626296}, {"indicative_day": "1999-12-27", "each_day": 197.61222330845803}, {"indicative_day": "1999-12-28", "each_day": 678.1143054280035}, {"indicative_day": "1999-12-29", "each_day": 898.6101511823645}, {"indicative_day": "1999-12-3", "each_day": -61.208055539195016}, {"indicative_day": "1999-12-30", "each_day": 755.0459979605791}, {"indicative_day": "1999-12-31", "each_day": -32.53015358177504}, {"indicative_day": "1999-12-4", "each_day": -49.30533040295741}, {"indicative_day": "1999-12-5", "each_day": -92.28396855042838}, {"indicative_day": "1999-12-6", "each_day": -131.66953678243283}, {"indicative_day": "1999-12-7", "each_day": -74.65624317897584}, {"indicative_day": "1999-12-8", "each_day": -29.080753219759686}, {"indicative_day": "1999-12-9", "each_day": -72.43640779879829}, {"indicative_day": "1999-2-1", "each_day": -96.70319360381444}, {"indicative_day": "1999-2-10", "each_day": 54.998997759052806}, {"indicative_day": "1999-2-11", "each_day": -10.328474744496077}, {"indicative_day": "1999-2-12", "each_day": -54.12633580021348}, {"indicative_day": "1999-2-13", "each_day": 15.855948510659076}, {"indicative_day": "1999-2-14", "each_day": 84.60330693986553}, {"indicative_day": "1999-2-15", "each_day": 61.20577131902977}, {"indicative_day": "1999-2-16", "each_day": 3.630501226949434}, {"indicative_day": "1999-2-17", "each_day": -33.82508502984295}, {"indicative_day": "1999-2-18", "each_day": -66.49273238561993}, {"indicative_day": "1999-2-19", "each_day": -90.518106397009}, {"indicative_day": "1999-2-2", "each_day": 29.39541329099755}, {"indicative_day": "1999-2-20", "each_day": -80.99939394513878}, {"indicative_day": "1999-2-21", "each_day": -50.24193831339997}, {"indicative_day": "1999-2-22", "each_day": -20.242283842904662}, {"indicative_day": "1999-2-23", "each_day": 7.339463225409266}, {"indicative_day": "1999-2-24", "each_day": 18.310184903264485}, {"indicative_day": "1999-2-25", "each_day": -9.142565315530673}, {"indicative_day": "1999-2-26", "each_day": -64.23083522904346}, {"indicative_day": "1999-2-27", "each_day": -116.58108270058581}, {"indicative_day": "1999-2-28", "each_day": -137.11630148022894}, {"indicative_day": "1999-2-29", "each_day": -121.18669783158775}, {"indicative_day": "1999-2-3", "each_day": -17.291363529970308}, {"indicative_day": "1999-2-4", "each_day": -130.60107693820788}, {"indicative_day": "1999-2-5", "each_day": -123.72771384408338}, {"indicative_day": "1999-2-6", "each_day": -73.519192199311}, {"indicative_day": "1999-2-7", "each_day": -101.29714102146902}, {"indicative_day": "1999-2-8", "each_day": -104.33597986424786}, {"indicative_day": "1999-2-9", "each_day": -4.060833892686466}, {"indicative_day": "1999-3-1", "each_day": -69.15125299755417}, {"indicative_day": "1999-3-10", "each_day": -37.215975446165764}, {"indicative_day": "1999-3-11", "each_day": -54.371884159651664}, {"indicative_day": "1999-3-12", "each_day": -92.98543836681799}, {"indicative_day": "1999-3-13", "each_day": -119.49668903073457}, {"indicative_day": "1999-3-14", "each_day": -119.70211036160303}, {"indicative_day": "1999-3-15", "each_day": -84.62265059434438}, {"indicative_day": "1999-3-16", "each_day": -14.020901958111242}, {"indicative_day": "1999-3-17", "each_day": 36.21929981173235}, {"indicative_day": "1999-3-18", "each_day": 6.4601834017454935}, {"indicative_day": "1999-3-19", "each_day": -58.49529437396294}, {"indicative_day": "1999-3-2", "each_day": 37.232252816640006}, {"indicative_day": "1999-3-20", "each_day": -69.60026546013924}, {"indicative_day": "1999-3-21", "each_day": -41.76594423561106}, {"indicative_day": "1999-3-22", "each_day": -49.263380346095225}, {"indicative_day": "1999-3-23", "each_day": -78.70017871350221}, {"indicative_day": "1999-3-24", "each_day": -71.69293598296856}, {"indicative_day": "1999-3-25", "each_day": -54.65734915330595}, {"indicative_day": "1999-3-26", "each_day": -80.53787324743979}, {"indicative_day": "1999-3-27", "each_day": -108.27478952665528}, {"indicative_day": "1999-3-28", "each_day": -78.12438264345569}, {"indicative_day": "1999-3-29", "each_day": -39.05435452247046}, {"indicative_day": "1999-3-3", "each_day": 84.83732231677743}, {"indicative_day": "1999-3-30", "each_day": -73.49272245381822}, {"indicative_day": "1999-3-31", "each_day": -151.9146234717227}, {"indicative_day": "1999-3-4", "each_day": 26.331333753166813}, {"indicative_day": "1999-3-5", "each_day": -58.99551884467176}, {"indicative_day": "1999-3-6", "each_day": -88.06487286005405}, {"indicative_day": "1999-3-7", "each_day": -76.45974554238744}, {"indicative_day": "1999-3-8", "each_day": -64.23726635638651}, {"indicative_day": "1999-3-9", "each_day": -49.34280571977802}, {"indicative_day": "1999-4-1", "each_day": -175.33480798397062}, {"indicative_day": "1999-4-10", "each_day": -178.4296442933059}, {"indicative_day": "1999-4-11", "each_day": -167.59985177573594}, {"indicative_day": "1999-4-12", "each_day": -163.8165103041972}, {"indicative_day": "1999-4-13", "each_day": -155.19873566806618}, {"indicative_day": "1999-4-14", "each_day": -108.50866871153593}, {"indicative_day": "1999-4-15", "each_day": -68.47832702857805}, {"indicative_day": "1999-4-16", "each_day": -90.34857390581166}, {"indicative_day": "1999-4-17", "each_day": -127.66183971989372}, {"indicative_day": "1999-4-18", "each_day": -116.43706420990031}, {"indicative_day": "1999-4-19", "each_day": -91.99321865379859}, {"indicative_day": "1999-4-2", "each_day": -127.4733752287174}, {"indicative_day": "1999-4-20", "each_day": -101.74884814952105}, {"indicative_day": "1999-4-21", "each_day": -109.30083403030945}, {"indicative_day": "1999-4-22", "each_day": -92.92523084721118}, {"indicative_day": "1999-4-23", "each_day": -106.50200617303237}, {"indicative_day": "1999-4-24", "each_day": -153.30797693567365}, {"indicative_day": "1999-4-25", "each_day": -153.7517449137093}, {"indicative_day": "1999-4-26", "each_day": -105.8640056373888}, {"indicative_day": "1999-4-27", "each_day": -107.84898652318574}, {"indicative_day": "1999-4-28", "each_day": -174.40005892380591}, {"indicative_day": "1999-4-29", "each_day": -204.31914353289653}, {"indicative_day": "1999-4-3", "each_day": -89.72338131090646}, {"indicative_day": "1999-4-30", "each_day": -159.48482036653382}, {"indicative_day": "1999-4-4", "each_day": -111.18013921526024}, {"indicative_day": "1999-4-5", "each_day": -141.0979224484337}, {"indicative_day": "1999-4-6", "each_day": -120.2791340227246}, {"indicative_day": "1999-4-7", "each_day": -82.24903163034746}, {"indicative_day": "1999-4-8", "each_day": -97.4831186724265}, {"indicative_day": "1999-4-9", "each_day": -152.93444477760468}, {"indicative_day": "1999-5-1", "each_day": -115.42192589493455}, {"indicative_day": "1999-5-10", "each_day": -93.29595046215533}, {"indicative_day": "1999-5-11", "each_day": -44.49701451458332}, {"indicative_day": "1999-5-12", "each_day": -94.70707465145308}, {"indicative_day": "1999-5-13", "each_day": -157.98532469809376}, {"indicative_day": "1999-5-14", "each_day": -134.6214900738162}, {"indicative_day": "1999-5-15", "each_day": -78.59178904059331}, {"indicative_day": "1999-5-16", "each_day": -76.31298242134257}, {"indicative_day": "1999-5-17", "each_day": -89.08797628624919}, {"indicative_day": "1999-5-18", "each_day": -49.20635049842059}, {"indicative_day": "1999-5-19", "each_day": 6.521309498788183}, {"indicative_day": "1999-5-2", "each_day": -122.97007611003046}, {"indicative_day": "1999-5-20", "each_day": 12.208733662913328}, {"indicative_day": "1999-5-21", "each_day": -14.897305115803652}, {"indicative_day": "1999-5-22", "each_day": -20.77917034736404}, {"indicative_day": "1999-5-23", "each_day": -7.407707381731993}, {"indicative_day": "1999-5-24", "each_day": -25.251376689736666}, {"indicative_day": "1999-5-25", "each_day": -93.51942032383405}, {"indicative_day": "1999-5-26", "each_day": -152.39836229490498}, {"indicative_day": "1999-5-27", "each_day": -134.88837995529667}, {"indicative_day": "1999-5-28", "each_day": -93.91722705033425}, {"indicative_day": "1999-5-29", "each_day": -146.64214828328062}, {"indicative_day": "1999-5-3", "each_day": -150.3300479098979}, {"indicative_day": "1999-5-30", "each_day": -247.97142121750932}, {"indicative_day": "1999-5-31", "each_day": -206.46716706137445}, {"indicative_day": "1999-5-4", "each_day": -154.85167647342735}, {"indicative_day": "1999-5-5", "each_day": -127.07423587583492}, {"indicative_day": "1999-5-6", "each_day": -93.1178599317723}, {"indicative_day": "1999-5-7", "each_day": -102.2295343652224}, {"indicative_day": "1999-5-8", "each_day": -153.56332091445685}, {"indicative_day": "1999-5-9", "each_day": -162.93973738340372}, {"indicative_day": "1999-6-1", "each_day": -15.122679175448102}, {"indicative_day": "1999-6-10", "each_day": 14.827199141635381}, {"indicative_day": "1999-6-11", "each_day": -40.70404302987099}, {"indicative_day": "1999-6-12", "each_day": -128.75289683261371}, {"indicative_day": "1999-6-13", "each_day": -152.9883248812908}, {"indicative_day": "1999-6-14", "each_day": -112.89540253865388}, {"indicative_day": "1999-6-15", "each_day": -26.445831938431905}, {"indicative_day": "1999-6-16", "each_day": 56.09292160217965}, {"indicative_day": "1999-6-17", "each_day": 42.586883836837465}, {"indicative_day": "1999-6-18", "each_day": -43.08676975537468}, {"indicative_day": "1999-6-19", "each_day": -57.04261745596623}, {"indicative_day": "1999-6-2", "each_day": 81.89554557857899}, {"indicative_day": "1999-6-20", "each_day": 7.378474726932813}, {"indicative_day": "1999-6-21", "each_day": 6.6190910117916335}, {"indicative_day": "1999-6-22", "each_day": -52.09335090133154}, {"indicative_day": "1999-6-23", "each_day": -19.895050626669224}, {"indicative_day": "1999-6-24", "each_day": 68.89803207490053}, {"indicative_day": "1999-6-25", "each_day": 53.999581531974385}, {"indicative_day": "1999-6-26", "each_day": -7.88522253767912}, {"indicative_day": "1999-6-27", "each_day": 41.02721738260017}, {"indicative_day": "1999-6-28", "each_day": 116.72465997526847}, {"indicative_day": "1999-6-29", "each_day": 92.43753206303779}, {"indicative_day": "1999-6-3", "each_day": -19.09687030374734}, {"indicative_day": "1999-6-30", "each_day": 121.56699737783947}, {"indicative_day": "1999-6-4", "each_day": -114.45516973727081}, {"indicative_day": "1999-6-5", "each_day": -67.77111749505596}, {"indicative_day": "1999-6-6", "each_day": -25.415524195467565}, {"indicative_day": "1999-6-7", "each_day": -92.3206164018463}, {"indicative_day": "1999-6-8", "each_day": -130.21797388379701}, {"indicative_day": "1999-6-9", "each_day": -49.36447566419862}, {"indicative_day": "1999-7-1", "each_day": 283.49013204308403}, {"indicative_day": "1999-7-10", "each_day": 211.23500308857425}, {"indicative_day": "1999-7-11", "each_day": 154.00953589570543}, {"indicative_day": "1999-7-12", "each_day": 51.63420707710069}, {"indicative_day": "1999-7-13", "each_day": 77.57912402474953}, {"indicative_day": "1999-7-14", "each_day": 217.9471260746953}, {"indicative_day": "1999-7-15", "each_day": 284.5072392504927}, {"indicative_day": "1999-7-16", "each_day": 228.5063835129682}, {"indicative_day": "1999-7-17", "each_day": 155.73325626820306}, {"indicative_day": "1999-7-18", "each_day": 123.58497898741044}, {"indicative_day": "1999-7-19", "each_day": 124.94697507420156}, {"indicative_day": "1999-7-2", "each_day": 224.07192786099532}, {"indicative_day": "1999-7-20", "each_day": 166.6317683872436}, {"indicative_day": "1999-7-21", "each_day": 226.26889050615972}, {"indicative_day": "1999-7-22", "each_day": 234.83399639723947}, {"indicative_day": "1999-7-23", "each_day": 178.7650937472904}, {"indicative_day": "1999-7-24", "each_day": 123.82297509058336}, {"indicative_day": "1999-7-25", "each_day": 116.08010248855234}, {"indicative_day": "1999-7-26", "each_day": 148.4370615235005}, {"indicative_day": "1999-7-27", "each_day": 204.13597126761277}, {"indicative_day": "1999-7-28", "each_day": 250.96511981860567}, {"indicative_day": "1999-7-29", "each_day": 239.61336680036203}, {"indicative_day": "1999-7-3", "each_day": -274.1465114145851}, {"indicative_day": "1999-7-30", "each_day": 176.24062728527886}, {"indicative_day": "1999-7-31", "each_day": 134.2768175492343}, {"indicative_day": "1999-7-4", "each_day": -706.5590986341202}, {"indicative_day": "1999-7-5", "each_day": -484.755164627645}, {"indicative_day": "1999-7-6", "each_day": 146.08319656126645}, {"indicative_day": "1999-7-7", "each_day": 484.231775418667}, {"indicative_day": "1999-7-8", "each_day": 381.8348661715918}, {"indicative_day": "1999-7-9", "each_day": 233.3861125301787}, {"indicative_day": "1999-8-1", "each_day": 147.64987699269827}, {"indicative_day": "1999-8-10", "each_day": 177.70834546830503}, {"indicative_day": "1999-8-11", "each_day": 163.27420689636932}, {"indicative_day": "1999-8-12", "each_day": 164.08341428019384}, {"indicative_day": "1999-8-13", "each_day": 159.79555416726038}, {"indicative_day": "1999-8-14", "each_day": 162.15575166223286}, {"indicative_day": "1999-8-15", "each_day": 180.29146747370447}, {"indicative_day": "1999-8-16", "each_day": 188.74206984063375}, {"indicative_day": "1999-8-17", "each_day": 184.34383100802532}, {"indicative_day": "1999-8-18", "each_day": 201.27549476994204}, {"indicative_day": "1999-8-19", "each_day": 229.33682854978528}, {"indicative_day": "1999-8-2", "each_day": 167.42741827117206}, {"indicative_day": "1999-8-20", "each_day": 213.56736204607878}, {"indicative_day": "1999-8-21", "each_day": 158.01068652982048}, {"indicative_day": "1999-8-22", "each_day": 121.47167815220288}, {"indicative_day": "1999-8-23", "each_day": 116.75432963534885}, {"indicative_day": "1999-8-24", "each_day": 127.20653483523822}, {"indicative_day": "1999-8-25", "each_day": 172.6502571625528}, {"indicative_day": "1999-8-26", "each_day": 240.42793485327698}, {"indicative_day": "1999-8-27", "each_day": 248.75158406375007}, {"indicative_day": "1999-8-28", "each_day": 193.2277243208835}, {"indicative_day": "1999-8-29", "each_day": 176.54061278954964}, {"indicative_day": "1999-8-3", "each_day": 153.14314302080066}, {"indicative_day": "1999-8-30", "each_day": 189.53548459296064}, {"indicative_day": "1999-8-31", "each_day": 88.92646334144136}, {"indicative_day": "1999-8-4", "each_day": 128.41857231901753}, {"indicative_day": "1999-8-5", "each_day": 128.86015351638517}, {"indicative_day": "1999-8-6", "each_day": 159.19615718315438}, {"indicative_day": "1999-8-7", "each_day": 201.90684551765372}, {"indicative_day": "1999-8-8", "each_day": 226.2941895777472}, {"indicative_day": "1999-8-9", "each_day": 211.01776295110122}, {"indicative_day": "1999-9-1", "each_day": -99.70022192035898}, {"indicative_day": "1999-9-10", "each_day": 333.01813276864306}, {"indicative_day": "1999-9-11", "each_day": 263.8400890229173}, {"indicative_day": "1999-9-12", "each_day": 192.0642710201286}, {"indicative_day": "1999-9-13", "each_day": 178.56812452997875}, {"indicative_day": "1999-9-14", "each_day": 267.9625018648084}, {"indicative_day": "1999-9-15", "each_day": 377.619635341646}, {"indicative_day": "1999-9-16", "each_day": 399.58556675183723}, {"indicative_day": "1999-9-17", "each_day": 359.23461003839265}, {"indicative_day": "1999-9-18", "each_day": 341.60157663248066}, {"indicative_day": "1999-9-19", "each_day": 356.99647758196977}, {"indicative_day": "1999-9-2", "each_day": -147.7231047989925}, {"indicative_day": "1999-9-20", "each_day": 378.9601110630621}, {"indicative_day": "1999-9-21", "each_day": 401.32143851783087}, {"indicative_day": "1999-9-22", "each_day": 400.26752618873996}, {"indicative_day": "1999-9-23", "each_day": 356.65025898735666}, {"indicative_day": "1999-9-24", "each_day": 320.65496698404866}, {"indicative_day": "1999-9-25", "each_day": 337.1039431328878}, {"indicative_day": "1999-9-26", "each_day": 352.01563146385945}, {"indicative_day": "1999-9-27", "each_day": 309.4124025345274}, {"indicative_day": "1999-9-28", "each_day": 258.31539144064516}, {"indicative_day": "1999-9-29", "each_day": 254.44324512682815}, {"indicative_day": "1999-9-3", "each_day": -15.729425851897577}, {"indicative_day": "1999-9-30", "each_day": 257.26876945733267}, {"indicative_day": "1999-9-4", "each_day": 49.65504252009268}, {"indicative_day": "1999-9-5", "each_day": -53.60298839071881}, {"indicative_day": "1999-9-6", "each_day": -101.9119886917774}, {"indicative_day": "1999-9-7", "each_day": 60.63811911702271}, {"indicative_day": "1999-9-8", "each_day": 278.3418418389805}, {"indicative_day": "1999-9-9", "each_day": 364.3417978022155}], "data-25713872bd7ea71ecc827a8ebbaa6350": [{"holiday": "Christmas Day", "ds": "1969-12-25T00:00:00", "effect size": -1454.3236428974549, "indicative_day": "1999-12-25"}, {"holiday": "Columbus Day", "ds": "1969-10-12T00:00:00", "effect size": 54.71341425499736, "indicative_day": "1999-10-12"}, {"holiday": "Independence Day", "ds": "1969-07-04T00:00:00", "effect size": -706.5590986341202, "indicative_day": "1999-7-4"}, {"holiday": "Labor Day", "ds": "1969-09-01T00:00:00", "effect size": -51.91467381565013, "indicative_day": "1999-9-1"}, {"holiday": "Martin Luther King Jr. Day", "ds": "1986-01-20T00:00:00", "effect size": 1.1219016959634083, "indicative_day": "1999-1-20"}, {"holiday": "Memorial Day", "ds": "1969-05-30T00:00:00", "effect size": -168.9892150532758, "indicative_day": "1999-5-30"}, {"holiday": "New Year's Day", "ds": "1969-01-01T00:00:00", "effect size": -935.7558852651148, "indicative_day": "1999-1-1"}, {"holiday": "Thanksgiving", "ds": "1969-11-27T00:00:00", "effect size": -288.6468788930782, "indicative_day": "1999-11-27"}, {"holiday": "Veterans Day", "ds": "1969-11-11T00:00:00", "effect size": -21.29044960545044, "indicative_day": "1999-11-11"}, {"holiday": "Washington's Birthday", "ds": "1969-02-22T00:00:00", "effect size": -28.984745311099154, "indicative_day": "1999-2-22"}]}}, {"mode": "vega-lite"});
</script>

