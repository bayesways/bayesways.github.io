---
layout: post
title: Recreating the US birthday analysis
giscus_comments: true
date: 2023-03-14
related_posts: false
---

In this post I will try to model the number of births in the US, using seasonal, day of week effects and special holiday effects. 
The original analysis used Gaussian Processes and it presented in BDA3 and recreated in this [post](https://avehtari.github.io/casestudies/Birthdays/birthdays.html) by Aki Vehtari.
Here I try to the same analysis but using a simpler additive model based on Fourier Transforms, implemented in the open source package [Prophet](https://facebook.github.io/prophet/) by facebook (Cuong Duong, Ben Letham, Sean Taylor).


```python
import pandas as pd
from prophet import Prophet
import altair as alt
import holidays
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


```python
m = Prophet(changepoint_prior_scale = 0.01, yearly_seasonality=20)
## add effect for day of the year: 
m.add_seasonality(name='each_day', period=365.25, fourier_order=1000)
m.fit(df)
```

    17:03:30 - cmdstanpy - INFO - Chain [1] start processing
    17:03:59 - cmdstanpy - INFO - Chain [1] done processing





    <prophet.forecaster.Prophet at 0x7fe099e03910>




```python
future = m.make_future_dataframe(periods=365)
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
      <th>7665</th>
      <td>1989-12-27</td>
    </tr>
    <tr>
      <th>7666</th>
      <td>1989-12-28</td>
    </tr>
    <tr>
      <th>7667</th>
      <td>1989-12-29</td>
    </tr>
    <tr>
      <th>7668</th>
      <td>1989-12-30</td>
    </tr>
    <tr>
      <th>7669</th>
      <td>1989-12-31</td>
    </tr>
  </tbody>
</table>
</div>




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
      <th>7665</th>
      <td>1989-12-27</td>
      <td>1.496115</td>
      <td>1.083597</td>
      <td>1.898431</td>
    </tr>
    <tr>
      <th>7666</th>
      <td>1989-12-28</td>
      <td>1.610035</td>
      <td>1.186170</td>
      <td>1.975209</td>
    </tr>
    <tr>
      <th>7667</th>
      <td>1989-12-29</td>
      <td>1.703037</td>
      <td>1.309262</td>
      <td>2.081328</td>
    </tr>
    <tr>
      <th>7668</th>
      <td>1989-12-30</td>
      <td>0.749265</td>
      <td>0.346253</td>
      <td>1.151336</td>
    </tr>
    <tr>
      <th>7669</th>
      <td>1989-12-31</td>
      <td>-0.067765</td>
      <td>-0.463520</td>
      <td>0.283119</td>
    </tr>
  </tbody>
</table>
</div>




```python

from prophet.plot import add_changepoints_to_plot
fig1 = m.plot(forecast)
a = add_changepoints_to_plot(fig1.gca(), m, forecast)
```


    
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2023-03-14-recreating-the-US-birthday-analysis/birthday_files/birthday_5_0.png" class="img-fluid rounded z-depth-1" %} </div>


```python
fig2 = m.plot_components(forecast)

```
    
<div class="col-sm mt-3 mt-md-0">{% include figure.html path="assets/img/2023-03-14-recreating-the-US-birthday-analysis/birthday_files/birthday_6_0.png" class="img-fluid rounded z-depth-1" %} </div>
    



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
fig3
```





<div id="altair-viz-590bf83eb3014f548538a076eca95d48"></div>
<script type="text/javascript">
  var VEGA_DEBUG = (typeof VEGA_DEBUG == "undefined") ? {} : VEGA_DEBUG;
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-590bf83eb3014f548538a076eca95d48") {
      outputDiv = document.getElementById("altair-viz-590bf83eb3014f548538a076eca95d48");
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
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "data": {"name": "data-d6a4fe4b6c7f80adbcc45ab77e5c4508"}, "mark": "line", "encoding": {"x": {"field": "indicative_day", "type": "temporal"}, "y": {"field": "each_day", "type": "quantitative"}}, "$schema": "https://vega.github.io/schema/vega-lite/v4.17.0.json", "datasets": {"data-d6a4fe4b6c7f80adbcc45ab77e5c4508": [{"indicative_day": "1999-1-1", "each_day": -1.3016875285020422}, {"indicative_day": "1999-1-10", "each_day": -0.16348779645429776}, {"indicative_day": "1999-1-11", "each_day": -0.01799363473229105}, {"indicative_day": "1999-1-12", "each_day": -0.053864714811325556}, {"indicative_day": "1999-1-13", "each_day": -0.08195612409002818}, {"indicative_day": "1999-1-14", "each_day": -0.008688157468319534}, {"indicative_day": "1999-1-15", "each_day": -0.1077907759736848}, {"indicative_day": "1999-1-16", "each_day": -0.15654385325658118}, {"indicative_day": "1999-1-17", "each_day": -0.11327166657905499}, {"indicative_day": "1999-1-18", "each_day": -0.018325786951498894}, {"indicative_day": "1999-1-19", "each_day": -0.0550319646494107}, {"indicative_day": "1999-1-2", "each_day": -0.7940519061976484}, {"indicative_day": "1999-1-20", "each_day": -0.0739775903932631}, {"indicative_day": "1999-1-21", "each_day": -0.09966364345705354}, {"indicative_day": "1999-1-22", "each_day": -0.12716131314977103}, {"indicative_day": "1999-1-23", "each_day": -0.1759473082343611}, {"indicative_day": "1999-1-24", "each_day": -0.12606379621640026}, {"indicative_day": "1999-1-25", "each_day": -0.03244405748456434}, {"indicative_day": "1999-1-26", "each_day": -0.06733310638215294}, {"indicative_day": "1999-1-27", "each_day": -0.0760745101028437}, {"indicative_day": "1999-1-28", "each_day": -0.09360237469207534}, {"indicative_day": "1999-1-29", "each_day": -0.15886691634586803}, {"indicative_day": "1999-1-3", "each_day": -0.28328552464135265}, {"indicative_day": "1999-1-30", "each_day": -0.17254431073057888}, {"indicative_day": "1999-1-31", "each_day": -0.18472488039314644}, {"indicative_day": "1999-1-4", "each_day": -0.15351797251221702}, {"indicative_day": "1999-1-5", "each_day": -0.17984254588407791}, {"indicative_day": "1999-1-6", "each_day": -0.14312867596138898}, {"indicative_day": "1999-1-7", "each_day": -0.1504527587932818}, {"indicative_day": "1999-1-8", "each_day": -0.23718758734288178}, {"indicative_day": "1999-1-9", "each_day": -0.32163476756435966}, {"indicative_day": "1999-10-1", "each_day": 0.2699720545067486}, {"indicative_day": "1999-10-10", "each_day": 0.19135574568236033}, {"indicative_day": "1999-10-11", "each_day": 0.0563194193709628}, {"indicative_day": "1999-10-12", "each_day": 0.08229854826367111}, {"indicative_day": "1999-10-13", "each_day": -0.03682189218454544}, {"indicative_day": "1999-10-14", "each_day": 0.08728773290857703}, {"indicative_day": "1999-10-15", "each_day": 0.08354594912193834}, {"indicative_day": "1999-10-16", "each_day": 0.0017075501045705054}, {"indicative_day": "1999-10-17", "each_day": -0.008495581540153447}, {"indicative_day": "1999-10-18", "each_day": -0.054849720215038804}, {"indicative_day": "1999-10-19", "each_day": -0.04650212402519236}, {"indicative_day": "1999-10-2", "each_day": 0.24509803004906086}, {"indicative_day": "1999-10-20", "each_day": -0.037074343459821094}, {"indicative_day": "1999-10-21", "each_day": 0.02275891254015478}, {"indicative_day": "1999-10-22", "each_day": -0.0560327514422193}, {"indicative_day": "1999-10-23", "each_day": -0.07906802174937522}, {"indicative_day": "1999-10-24", "each_day": -0.06626835943085178}, {"indicative_day": "1999-10-25", "each_day": -0.018637260642223992}, {"indicative_day": "1999-10-26", "each_day": 0.006949742980155991}, {"indicative_day": "1999-10-27", "each_day": -0.008264438573951136}, {"indicative_day": "1999-10-28", "each_day": -0.004817698724890104}, {"indicative_day": "1999-10-29", "each_day": -0.08477910037569314}, {"indicative_day": "1999-10-3", "each_day": 0.20533699845193856}, {"indicative_day": "1999-10-30", "each_day": -0.06817678753319198}, {"indicative_day": "1999-10-31", "each_day": -0.21182436765428725}, {"indicative_day": "1999-10-4", "each_day": 0.18090231169802923}, {"indicative_day": "1999-10-5", "each_day": 0.1847259524312019}, {"indicative_day": "1999-10-6", "each_day": 0.1597747312193126}, {"indicative_day": "1999-10-7", "each_day": 0.13590228648734273}, {"indicative_day": "1999-10-8", "each_day": 0.1059203636079626}, {"indicative_day": "1999-10-9", "each_day": 0.1191230068841606}, {"indicative_day": "1999-11-1", "each_day": -0.029179053762674087}, {"indicative_day": "1999-11-10", "each_day": -0.023389783914108828}, {"indicative_day": "1999-11-11", "each_day": -0.014957635554108076}, {"indicative_day": "1999-11-12", "each_day": -0.09864887571681405}, {"indicative_day": "1999-11-13", "each_day": -0.13554905417315694}, {"indicative_day": "1999-11-14", "each_day": -0.03974003395811995}, {"indicative_day": "1999-11-15", "each_day": -0.026340748902997962}, {"indicative_day": "1999-11-16", "each_day": -0.03185591759669193}, {"indicative_day": "1999-11-17", "each_day": 0.012553079318899965}, {"indicative_day": "1999-11-18", "each_day": 0.055088461519160935}, {"indicative_day": "1999-11-19", "each_day": 0.03470492685356789}, {"indicative_day": "1999-11-2", "each_day": -0.08991647308027943}, {"indicative_day": "1999-11-20", "each_day": 0.07396419140459323}, {"indicative_day": "1999-11-21", "each_day": 0.05176887787211161}, {"indicative_day": "1999-11-22", "each_day": -0.22188466326159398}, {"indicative_day": "1999-11-23", "each_day": -0.16925033572342948}, {"indicative_day": "1999-11-24", "each_day": -0.2515315947049744}, {"indicative_day": "1999-11-25", "each_day": -0.22740690806910627}, {"indicative_day": "1999-11-26", "each_day": -0.33273235039151206}, {"indicative_day": "1999-11-27", "each_day": -0.3950897935427841}, {"indicative_day": "1999-11-28", "each_day": -0.26120540761382355}, {"indicative_day": "1999-11-29", "each_day": -0.0373285681011555}, {"indicative_day": "1999-11-3", "each_day": -0.014390500662336733}, {"indicative_day": "1999-11-30", "each_day": 0.06504403049763549}, {"indicative_day": "1999-11-4", "each_day": 0.030162724528828257}, {"indicative_day": "1999-11-5", "each_day": -0.042835992621528804}, {"indicative_day": "1999-11-6", "each_day": -0.06889453215681651}, {"indicative_day": "1999-11-7", "each_day": -0.023822405204814275}, {"indicative_day": "1999-11-8", "each_day": -0.02166626989496563}, {"indicative_day": "1999-11-9", "each_day": -0.025578211476639627}, {"indicative_day": "1999-12-1", "each_day": 0.10401354228046564}, {"indicative_day": "1999-12-10", "each_day": -0.13359609396649152}, {"indicative_day": "1999-12-11", "each_day": -0.16188036922299118}, {"indicative_day": "1999-12-12", "each_day": -0.06760472655291146}, {"indicative_day": "1999-12-13", "each_day": -0.2085361305989857}, {"indicative_day": "1999-12-14", "each_day": -0.052789447207455896}, {"indicative_day": "1999-12-15", "each_day": 0.09893610295773993}, {"indicative_day": "1999-12-16", "each_day": 0.19431548118825498}, {"indicative_day": "1999-12-17", "each_day": 0.19630601642467238}, {"indicative_day": "1999-12-18", "each_day": 0.2488535058391148}, {"indicative_day": "1999-12-19", "each_day": 0.32395140300890324}, {"indicative_day": "1999-12-2", "each_day": 0.0759026251936204}, {"indicative_day": "1999-12-20", "each_day": 0.2566057775273213}, {"indicative_day": "1999-12-21", "each_day": 0.11914363975423115}, {"indicative_day": "1999-12-22", "each_day": -0.21284528491705623}, {"indicative_day": "1999-12-23", "each_day": -0.548895074575928}, {"indicative_day": "1999-12-24", "each_day": -1.1273516996711745}, {"indicative_day": "1999-12-25", "each_day": -1.5481635013715147}, {"indicative_day": "1999-12-26", "each_day": -0.577772031899057}, {"indicative_day": "1999-12-27", "each_day": 0.32100230861634876}, {"indicative_day": "1999-12-28", "each_day": 0.6166993288924372}, {"indicative_day": "1999-12-29", "each_day": 0.6618072434220138}, {"indicative_day": "1999-12-3", "each_day": -0.05317939003583895}, {"indicative_day": "1999-12-30", "each_day": 0.758022899242529}, {"indicative_day": "1999-12-31", "each_day": 0.2143427427249748}, {"indicative_day": "1999-12-4", "each_day": -0.07758362788593239}, {"indicative_day": "1999-12-5", "each_day": -0.06237055231803472}, {"indicative_day": "1999-12-6", "each_day": -0.11300850513852219}, {"indicative_day": "1999-12-7", "each_day": -0.15229887231388348}, {"indicative_day": "1999-12-8", "each_day": -0.07210520053825505}, {"indicative_day": "1999-12-9", "each_day": -0.11129442326962635}, {"indicative_day": "1999-2-1", "each_day": -0.06435705205543062}, {"indicative_day": "1999-2-10", "each_day": -0.02547983596826729}, {"indicative_day": "1999-2-11", "each_day": -0.07859959727148906}, {"indicative_day": "1999-2-12", "each_day": -0.00489352145947125}, {"indicative_day": "1999-2-13", "each_day": -0.2530180114222627}, {"indicative_day": "1999-2-14", "each_day": 0.216600578388773}, {"indicative_day": "1999-2-15", "each_day": -0.05332339130364079}, {"indicative_day": "1999-2-16", "each_day": -0.08735200557748343}, {"indicative_day": "1999-2-17", "each_day": -0.04637955956264119}, {"indicative_day": "1999-2-18", "each_day": -0.06994953126946553}, {"indicative_day": "1999-2-19", "each_day": -0.13044081371976887}, {"indicative_day": "1999-2-2", "each_day": 0.002222370899137386}, {"indicative_day": "1999-2-20", "each_day": -0.043533187078643995}, {"indicative_day": "1999-2-21", "each_day": -0.1142196099616656}, {"indicative_day": "1999-2-22", "each_day": 0.02935696990236951}, {"indicative_day": "1999-2-23", "each_day": -0.03611217531919976}, {"indicative_day": "1999-2-24", "each_day": 0.0181391913842484}, {"indicative_day": "1999-2-25", "each_day": -0.0021711879290748776}, {"indicative_day": "1999-2-26", "each_day": -0.06933866755766956}, {"indicative_day": "1999-2-27", "each_day": -0.1593626855923896}, {"indicative_day": "1999-2-28", "each_day": -0.09216299325839608}, {"indicative_day": "1999-2-29", "each_day": -0.44549831550734603}, {"indicative_day": "1999-2-3", "each_day": -0.04066016379047365}, {"indicative_day": "1999-2-4", "each_day": -0.11810217557578499}, {"indicative_day": "1999-2-5", "each_day": -0.12631100446679439}, {"indicative_day": "1999-2-6", "each_day": -0.1393593847629202}, {"indicative_day": "1999-2-7", "each_day": -0.1324764892376433}, {"indicative_day": "1999-2-8", "each_day": -0.10013121692211198}, {"indicative_day": "1999-2-9", "each_day": -0.09887056478735885}, {"indicative_day": "1999-3-1", "each_day": -0.010503177820420224}, {"indicative_day": "1999-3-10", "each_day": -0.0414834631652652}, {"indicative_day": "1999-3-11", "each_day": -0.03994453048199231}, {"indicative_day": "1999-3-12", "each_day": -0.07041506645327267}, {"indicative_day": "1999-3-13", "each_day": -0.19688582492442114}, {"indicative_day": "1999-3-14", "each_day": -0.07248640645931277}, {"indicative_day": "1999-3-15", "each_day": -0.0999839026775785}, {"indicative_day": "1999-3-16", "each_day": -0.09493473781645842}, {"indicative_day": "1999-3-17", "each_day": 0.04060062388540702}, {"indicative_day": "1999-3-18", "each_day": -0.06460253583197259}, {"indicative_day": "1999-3-19", "each_day": -0.10066636694318276}, {"indicative_day": "1999-3-2", "each_day": -0.0493062641142164}, {"indicative_day": "1999-3-20", "each_day": -0.09006730494194792}, {"indicative_day": "1999-3-21", "each_day": -0.07642175739316939}, {"indicative_day": "1999-3-22", "each_day": -0.10571058527452382}, {"indicative_day": "1999-3-23", "each_day": -0.10288699440313592}, {"indicative_day": "1999-3-24", "each_day": -0.1338877842095682}, {"indicative_day": "1999-3-25", "each_day": -0.04901907245274005}, {"indicative_day": "1999-3-26", "each_day": -0.12302799508038144}, {"indicative_day": "1999-3-27", "each_day": -0.12341707152578704}, {"indicative_day": "1999-3-28", "each_day": -0.08558146693346781}, {"indicative_day": "1999-3-29", "each_day": -0.07159252758886636}, {"indicative_day": "1999-3-3", "each_day": 0.08125596181417002}, {"indicative_day": "1999-3-30", "each_day": -0.12466324642339413}, {"indicative_day": "1999-3-31", "each_day": -0.10096162051643393}, {"indicative_day": "1999-3-4", "each_day": -0.004549851141679937}, {"indicative_day": "1999-3-5", "each_day": -0.06574174837981239}, {"indicative_day": "1999-3-6", "each_day": -0.10283132762499585}, {"indicative_day": "1999-3-7", "each_day": -0.09074244245373297}, {"indicative_day": "1999-3-8", "each_day": -0.0240996356994655}, {"indicative_day": "1999-3-9", "each_day": -0.12307067467508097}, {"indicative_day": "1999-4-1", "each_day": -0.27813009284079404}, {"indicative_day": "1999-4-10", "each_day": -0.1983617499587368}, {"indicative_day": "1999-4-11", "each_day": -0.18922493945497443}, {"indicative_day": "1999-4-12", "each_day": -0.1558209654000536}, {"indicative_day": "1999-4-13", "each_day": -0.2647262766191971}, {"indicative_day": "1999-4-14", "each_day": -0.13903648015305498}, {"indicative_day": "1999-4-15", "each_day": -0.09413870724901886}, {"indicative_day": "1999-4-16", "each_day": -0.16627569810560888}, {"indicative_day": "1999-4-17", "each_day": -0.1935266123402108}, {"indicative_day": "1999-4-18", "each_day": -0.13419590492530956}, {"indicative_day": "1999-4-19", "each_day": -0.15389974316949076}, {"indicative_day": "1999-4-2", "each_day": -0.048032555383991794}, {"indicative_day": "1999-4-20", "each_day": -0.20410370016894946}, {"indicative_day": "1999-4-21", "each_day": -0.16867498926517982}, {"indicative_day": "1999-4-22", "each_day": -0.12012110437665081}, {"indicative_day": "1999-4-23", "each_day": -0.18513807494689016}, {"indicative_day": "1999-4-24", "each_day": -0.23749166137360025}, {"indicative_day": "1999-4-25", "each_day": -0.1938431165747881}, {"indicative_day": "1999-4-26", "each_day": -0.15539881202266573}, {"indicative_day": "1999-4-27", "each_day": -0.23751519757953485}, {"indicative_day": "1999-4-28", "each_day": -0.17453476262401904}, {"indicative_day": "1999-4-29", "each_day": -0.25131263201931875}, {"indicative_day": "1999-4-3", "each_day": -0.1402776478548151}, {"indicative_day": "1999-4-30", "each_day": -0.23902573497889681}, {"indicative_day": "1999-4-4", "each_day": -0.08807977708551251}, {"indicative_day": "1999-4-5", "each_day": -0.18865467704934713}, {"indicative_day": "1999-4-6", "each_day": -0.14206587723815636}, {"indicative_day": "1999-4-7", "each_day": -0.13869772175982817}, {"indicative_day": "1999-4-8", "each_day": -0.10365725232611556}, {"indicative_day": "1999-4-9", "each_day": -0.20072669665329343}, {"indicative_day": "1999-5-1", "each_day": -0.10773736812802982}, {"indicative_day": "1999-5-10", "each_day": -0.1170170557241572}, {"indicative_day": "1999-5-11", "each_day": -0.129733452815525}, {"indicative_day": "1999-5-12", "each_day": -0.14023540765404738}, {"indicative_day": "1999-5-13", "each_day": -0.21604991315333294}, {"indicative_day": "1999-5-14", "each_day": -0.19620426022114032}, {"indicative_day": "1999-5-15", "each_day": -0.12259273552223587}, {"indicative_day": "1999-5-16", "each_day": -0.15048456199136465}, {"indicative_day": "1999-5-17", "each_day": -0.14953105405337247}, {"indicative_day": "1999-5-18", "each_day": -0.12373936820474798}, {"indicative_day": "1999-5-19", "each_day": -0.1263494301757375}, {"indicative_day": "1999-5-2", "each_day": -0.15865650347302923}, {"indicative_day": "1999-5-20", "each_day": 0.023363982654586894}, {"indicative_day": "1999-5-21", "each_day": -0.11170212236648769}, {"indicative_day": "1999-5-22", "each_day": -0.06681430686027666}, {"indicative_day": "1999-5-23", "each_day": -0.04819078827573551}, {"indicative_day": "1999-5-24", "each_day": -0.041344016816070604}, {"indicative_day": "1999-5-25", "each_day": -0.14247268313353828}, {"indicative_day": "1999-5-26", "each_day": -0.21993304274726752}, {"indicative_day": "1999-5-27", "each_day": -0.09739818525245514}, {"indicative_day": "1999-5-28", "each_day": -0.20795025112031723}, {"indicative_day": "1999-5-29", "each_day": -0.07381276800773288}, {"indicative_day": "1999-5-3", "each_day": -0.18173242053280464}, {"indicative_day": "1999-5-30", "each_day": -0.2712715632682913}, {"indicative_day": "1999-5-31", "each_day": -0.20582763655059272}, {"indicative_day": "1999-5-4", "each_day": -0.2232743072411659}, {"indicative_day": "1999-5-5", "each_day": -0.10808888418146147}, {"indicative_day": "1999-5-6", "each_day": -0.13569847573247174}, {"indicative_day": "1999-5-7", "each_day": -0.1660308596346734}, {"indicative_day": "1999-5-8", "each_day": -0.16687920605317946}, {"indicative_day": "1999-5-9", "each_day": -0.20131928229046758}, {"indicative_day": "1999-6-1", "each_day": 0.0031393828060392428}, {"indicative_day": "1999-6-10", "each_day": 0.020375657469869888}, {"indicative_day": "1999-6-11", "each_day": -0.07536102595926605}, {"indicative_day": "1999-6-12", "each_day": -0.06301264390616398}, {"indicative_day": "1999-6-13", "each_day": -0.1927039058781133}, {"indicative_day": "1999-6-14", "each_day": -0.04448357339348485}, {"indicative_day": "1999-6-15", "each_day": -0.056497299033614855}, {"indicative_day": "1999-6-16", "each_day": 0.0449718850629673}, {"indicative_day": "1999-6-17", "each_day": 0.04202003315778706}, {"indicative_day": "1999-6-18", "each_day": -0.04220860251531061}, {"indicative_day": "1999-6-19", "each_day": -0.07015889910764217}, {"indicative_day": "1999-6-2", "each_day": 0.006004489749509827}, {"indicative_day": "1999-6-20", "each_day": 0.04036409398708781}, {"indicative_day": "1999-6-21", "each_day": -0.049016920846756046}, {"indicative_day": "1999-6-22", "each_day": -0.029756572704545407}, {"indicative_day": "1999-6-23", "each_day": -0.030250205211860587}, {"indicative_day": "1999-6-24", "each_day": 0.04509681824756376}, {"indicative_day": "1999-6-25", "each_day": 0.03934134598483708}, {"indicative_day": "1999-6-26", "each_day": 0.03710966324617169}, {"indicative_day": "1999-6-27", "each_day": 0.03706598384207201}, {"indicative_day": "1999-6-28", "each_day": 0.11552889597889687}, {"indicative_day": "1999-6-29", "each_day": 0.09000949710584007}, {"indicative_day": "1999-6-3", "each_day": -0.041909219225374245}, {"indicative_day": "1999-6-30", "each_day": 0.17955007669869336}, {"indicative_day": "1999-6-4", "each_day": -0.10737931970041374}, {"indicative_day": "1999-6-5", "each_day": -0.10710140528374078}, {"indicative_day": "1999-6-6", "each_day": 0.005620646150415896}, {"indicative_day": "1999-6-7", "each_day": -0.09151471433257388}, {"indicative_day": "1999-6-8", "each_day": -0.14809635957494485}, {"indicative_day": "1999-6-9", "each_day": -0.05260643262436998}, {"indicative_day": "1999-7-1", "each_day": 0.2454452086345132}, {"indicative_day": "1999-7-10", "each_day": 0.2535695046449866}, {"indicative_day": "1999-7-11", "each_day": 0.13445467801935007}, {"indicative_day": "1999-7-12", "each_day": 0.10106931997093069}, {"indicative_day": "1999-7-13", "each_day": 0.02859601809172114}, {"indicative_day": "1999-7-14", "each_day": 0.24145060249867914}, {"indicative_day": "1999-7-15", "each_day": 0.2917574286799216}, {"indicative_day": "1999-7-16", "each_day": 0.2083251954797649}, {"indicative_day": "1999-7-17", "each_day": 0.19049432002148037}, {"indicative_day": "1999-7-18", "each_day": 0.11279687915007211}, {"indicative_day": "1999-7-19", "each_day": 0.12492356547410453}, {"indicative_day": "1999-7-2", "each_day": 0.2471222204953866}, {"indicative_day": "1999-7-20", "each_day": 0.22730202650647152}, {"indicative_day": "1999-7-21", "each_day": 0.19374655609743657}, {"indicative_day": "1999-7-22", "each_day": 0.281150036969938}, {"indicative_day": "1999-7-23", "each_day": 0.21279038616153056}, {"indicative_day": "1999-7-24", "each_day": 0.18193566462857125}, {"indicative_day": "1999-7-25", "each_day": 0.1811923123857468}, {"indicative_day": "1999-7-26", "each_day": 0.1897136387289007}, {"indicative_day": "1999-7-27", "each_day": 0.2564117506058412}, {"indicative_day": "1999-7-28", "each_day": 0.2797254144166218}, {"indicative_day": "1999-7-29", "each_day": 0.30681610065507636}, {"indicative_day": "1999-7-3", "each_day": 0.04160125083386287}, {"indicative_day": "1999-7-30", "each_day": 0.2212272267263563}, {"indicative_day": "1999-7-31", "each_day": 0.18226974437994667}, {"indicative_day": "1999-7-4", "each_day": -0.9173376168596974}, {"indicative_day": "1999-7-5", "each_day": -0.20499297465590618}, {"indicative_day": "1999-7-6", "each_day": 0.22734043509399415}, {"indicative_day": "1999-7-7", "each_day": 0.42619554545469523}, {"indicative_day": "1999-7-8", "each_day": 0.4151204436215464}, {"indicative_day": "1999-7-9", "each_day": 0.26541166611286476}, {"indicative_day": "1999-8-1", "each_day": 0.2219734093079857}, {"indicative_day": "1999-8-10", "each_day": 0.2542582438704688}, {"indicative_day": "1999-8-11", "each_day": 0.2022414488256}, {"indicative_day": "1999-8-12", "each_day": 0.3333299057905831}, {"indicative_day": "1999-8-13", "each_day": 0.16290892623806216}, {"indicative_day": "1999-8-14", "each_day": 0.2650724194957417}, {"indicative_day": "1999-8-15", "each_day": 0.2831040030741819}, {"indicative_day": "1999-8-16", "each_day": 0.23676588632206677}, {"indicative_day": "1999-8-17", "each_day": 0.2250079750530002}, {"indicative_day": "1999-8-18", "each_day": 0.2649634778923896}, {"indicative_day": "1999-8-19", "each_day": 0.270077565447041}, {"indicative_day": "1999-8-2", "each_day": 0.17841942456762}, {"indicative_day": "1999-8-20", "each_day": 0.25514799207548566}, {"indicative_day": "1999-8-21", "each_day": 0.2123443851739204}, {"indicative_day": "1999-8-22", "each_day": 0.18512898072052084}, {"indicative_day": "1999-8-23", "each_day": 0.15733066710659077}, {"indicative_day": "1999-8-24", "each_day": 0.16321014360438146}, {"indicative_day": "1999-8-25", "each_day": 0.21367218817806022}, {"indicative_day": "1999-8-26", "each_day": 0.3241495055001735}, {"indicative_day": "1999-8-27", "each_day": 0.2720616673968796}, {"indicative_day": "1999-8-28", "each_day": 0.30608224866556355}, {"indicative_day": "1999-8-29", "each_day": 0.2567738461058104}, {"indicative_day": "1999-8-3", "each_day": 0.21724239019405028}, {"indicative_day": "1999-8-30", "each_day": 0.2746452681732259}, {"indicative_day": "1999-8-31", "each_day": 0.26349011297317615}, {"indicative_day": "1999-8-4", "each_day": 0.16894387101379701}, {"indicative_day": "1999-8-5", "each_day": 0.20483418135890546}, {"indicative_day": "1999-8-6", "each_day": 0.21857159334781645}, {"indicative_day": "1999-8-7", "each_day": 0.23299080505285277}, {"indicative_day": "1999-8-8", "each_day": 0.36742669744630757}, {"indicative_day": "1999-8-9", "each_day": 0.21175001728361545}, {"indicative_day": "1999-9-1", "each_day": -0.06585960759346099}, {"indicative_day": "1999-9-10", "each_day": 0.36102343182080937}, {"indicative_day": "1999-9-11", "each_day": 0.29867270422110004}, {"indicative_day": "1999-9-12", "each_day": 0.2973214291365146}, {"indicative_day": "1999-9-13", "each_day": 0.1801757429026565}, {"indicative_day": "1999-9-14", "each_day": 0.3423339938280492}, {"indicative_day": "1999-9-15", "each_day": 0.32956233575004656}, {"indicative_day": "1999-9-16", "each_day": 0.43465697534240216}, {"indicative_day": "1999-9-17", "each_day": 0.35897833525104367}, {"indicative_day": "1999-9-18", "each_day": 0.35449977476647154}, {"indicative_day": "1999-9-19", "each_day": 0.3811784184389873}, {"indicative_day": "1999-9-2", "each_day": 0.05103112458139436}, {"indicative_day": "1999-9-20", "each_day": 0.42353301647346064}, {"indicative_day": "1999-9-21", "each_day": 0.41018707515067226}, {"indicative_day": "1999-9-22", "each_day": 0.4084152924209283}, {"indicative_day": "1999-9-23", "each_day": 0.4191223432066638}, {"indicative_day": "1999-9-24", "each_day": 0.37188527140754524}, {"indicative_day": "1999-9-25", "each_day": 0.3701597397391732}, {"indicative_day": "1999-9-26", "each_day": 0.408371057246445}, {"indicative_day": "1999-9-27", "each_day": 0.37489745046336526}, {"indicative_day": "1999-9-28", "each_day": 0.29524320383300207}, {"indicative_day": "1999-9-29", "each_day": 0.31193032592517655}, {"indicative_day": "1999-9-3", "each_day": 0.10156705083010882}, {"indicative_day": "1999-9-30", "each_day": 0.34312741580732004}, {"indicative_day": "1999-9-4", "each_day": 0.1912222605888119}, {"indicative_day": "1999-9-5", "each_day": 0.07735099224308409}, {"indicative_day": "1999-9-6", "each_day": 0.13117816786963202}, {"indicative_day": "1999-9-7", "each_day": 0.11966764559793115}, {"indicative_day": "1999-9-8", "each_day": 0.3148529094861128}, {"indicative_day": "1999-9-9", "each_day": 0.46369691963564524}]}}, {"mode": "vega-lite"});
</script>




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
fig3 + fig4 
```





<div id="altair-viz-519a49308e1d47b7b49e8813aa79f8e7"></div>
<script type="text/javascript">
  var VEGA_DEBUG = (typeof VEGA_DEBUG == "undefined") ? {} : VEGA_DEBUG;
  (function(spec, embedOpt){
    let outputDiv = document.currentScript.previousElementSibling;
    if (outputDiv.id !== "altair-viz-519a49308e1d47b7b49e8813aa79f8e7") {
      outputDiv = document.getElementById("altair-viz-519a49308e1d47b7b49e8813aa79f8e7");
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
  })({"config": {"view": {"continuousWidth": 400, "continuousHeight": 300}}, "layer": [{"data": {"name": "data-d6a4fe4b6c7f80adbcc45ab77e5c4508"}, "mark": "line", "encoding": {"x": {"field": "indicative_day", "type": "temporal"}, "y": {"field": "each_day", "type": "quantitative"}}}, {"data": {"name": "data-6490e5d4bbc61f7fe504b170ff6e392e"}, "mark": "text", "encoding": {"text": {"field": "holiday", "type": "nominal"}, "x": {"field": "indicative_day", "type": "temporal"}, "y": {"field": "effect size", "type": "quantitative"}}}], "$schema": "https://vega.github.io/schema/vega-lite/v4.17.0.json", "datasets": {"data-d6a4fe4b6c7f80adbcc45ab77e5c4508": [{"indicative_day": "1999-1-1", "each_day": -1.3016875285020422}, {"indicative_day": "1999-1-10", "each_day": -0.16348779645429776}, {"indicative_day": "1999-1-11", "each_day": -0.01799363473229105}, {"indicative_day": "1999-1-12", "each_day": -0.053864714811325556}, {"indicative_day": "1999-1-13", "each_day": -0.08195612409002818}, {"indicative_day": "1999-1-14", "each_day": -0.008688157468319534}, {"indicative_day": "1999-1-15", "each_day": -0.1077907759736848}, {"indicative_day": "1999-1-16", "each_day": -0.15654385325658118}, {"indicative_day": "1999-1-17", "each_day": -0.11327166657905499}, {"indicative_day": "1999-1-18", "each_day": -0.018325786951498894}, {"indicative_day": "1999-1-19", "each_day": -0.0550319646494107}, {"indicative_day": "1999-1-2", "each_day": -0.7940519061976484}, {"indicative_day": "1999-1-20", "each_day": -0.0739775903932631}, {"indicative_day": "1999-1-21", "each_day": -0.09966364345705354}, {"indicative_day": "1999-1-22", "each_day": -0.12716131314977103}, {"indicative_day": "1999-1-23", "each_day": -0.1759473082343611}, {"indicative_day": "1999-1-24", "each_day": -0.12606379621640026}, {"indicative_day": "1999-1-25", "each_day": -0.03244405748456434}, {"indicative_day": "1999-1-26", "each_day": -0.06733310638215294}, {"indicative_day": "1999-1-27", "each_day": -0.0760745101028437}, {"indicative_day": "1999-1-28", "each_day": -0.09360237469207534}, {"indicative_day": "1999-1-29", "each_day": -0.15886691634586803}, {"indicative_day": "1999-1-3", "each_day": -0.28328552464135265}, {"indicative_day": "1999-1-30", "each_day": -0.17254431073057888}, {"indicative_day": "1999-1-31", "each_day": -0.18472488039314644}, {"indicative_day": "1999-1-4", "each_day": -0.15351797251221702}, {"indicative_day": "1999-1-5", "each_day": -0.17984254588407791}, {"indicative_day": "1999-1-6", "each_day": -0.14312867596138898}, {"indicative_day": "1999-1-7", "each_day": -0.1504527587932818}, {"indicative_day": "1999-1-8", "each_day": -0.23718758734288178}, {"indicative_day": "1999-1-9", "each_day": -0.32163476756435966}, {"indicative_day": "1999-10-1", "each_day": 0.2699720545067486}, {"indicative_day": "1999-10-10", "each_day": 0.19135574568236033}, {"indicative_day": "1999-10-11", "each_day": 0.0563194193709628}, {"indicative_day": "1999-10-12", "each_day": 0.08229854826367111}, {"indicative_day": "1999-10-13", "each_day": -0.03682189218454544}, {"indicative_day": "1999-10-14", "each_day": 0.08728773290857703}, {"indicative_day": "1999-10-15", "each_day": 0.08354594912193834}, {"indicative_day": "1999-10-16", "each_day": 0.0017075501045705054}, {"indicative_day": "1999-10-17", "each_day": -0.008495581540153447}, {"indicative_day": "1999-10-18", "each_day": -0.054849720215038804}, {"indicative_day": "1999-10-19", "each_day": -0.04650212402519236}, {"indicative_day": "1999-10-2", "each_day": 0.24509803004906086}, {"indicative_day": "1999-10-20", "each_day": -0.037074343459821094}, {"indicative_day": "1999-10-21", "each_day": 0.02275891254015478}, {"indicative_day": "1999-10-22", "each_day": -0.0560327514422193}, {"indicative_day": "1999-10-23", "each_day": -0.07906802174937522}, {"indicative_day": "1999-10-24", "each_day": -0.06626835943085178}, {"indicative_day": "1999-10-25", "each_day": -0.018637260642223992}, {"indicative_day": "1999-10-26", "each_day": 0.006949742980155991}, {"indicative_day": "1999-10-27", "each_day": -0.008264438573951136}, {"indicative_day": "1999-10-28", "each_day": -0.004817698724890104}, {"indicative_day": "1999-10-29", "each_day": -0.08477910037569314}, {"indicative_day": "1999-10-3", "each_day": 0.20533699845193856}, {"indicative_day": "1999-10-30", "each_day": -0.06817678753319198}, {"indicative_day": "1999-10-31", "each_day": -0.21182436765428725}, {"indicative_day": "1999-10-4", "each_day": 0.18090231169802923}, {"indicative_day": "1999-10-5", "each_day": 0.1847259524312019}, {"indicative_day": "1999-10-6", "each_day": 0.1597747312193126}, {"indicative_day": "1999-10-7", "each_day": 0.13590228648734273}, {"indicative_day": "1999-10-8", "each_day": 0.1059203636079626}, {"indicative_day": "1999-10-9", "each_day": 0.1191230068841606}, {"indicative_day": "1999-11-1", "each_day": -0.029179053762674087}, {"indicative_day": "1999-11-10", "each_day": -0.023389783914108828}, {"indicative_day": "1999-11-11", "each_day": -0.014957635554108076}, {"indicative_day": "1999-11-12", "each_day": -0.09864887571681405}, {"indicative_day": "1999-11-13", "each_day": -0.13554905417315694}, {"indicative_day": "1999-11-14", "each_day": -0.03974003395811995}, {"indicative_day": "1999-11-15", "each_day": -0.026340748902997962}, {"indicative_day": "1999-11-16", "each_day": -0.03185591759669193}, {"indicative_day": "1999-11-17", "each_day": 0.012553079318899965}, {"indicative_day": "1999-11-18", "each_day": 0.055088461519160935}, {"indicative_day": "1999-11-19", "each_day": 0.03470492685356789}, {"indicative_day": "1999-11-2", "each_day": -0.08991647308027943}, {"indicative_day": "1999-11-20", "each_day": 0.07396419140459323}, {"indicative_day": "1999-11-21", "each_day": 0.05176887787211161}, {"indicative_day": "1999-11-22", "each_day": -0.22188466326159398}, {"indicative_day": "1999-11-23", "each_day": -0.16925033572342948}, {"indicative_day": "1999-11-24", "each_day": -0.2515315947049744}, {"indicative_day": "1999-11-25", "each_day": -0.22740690806910627}, {"indicative_day": "1999-11-26", "each_day": -0.33273235039151206}, {"indicative_day": "1999-11-27", "each_day": -0.3950897935427841}, {"indicative_day": "1999-11-28", "each_day": -0.26120540761382355}, {"indicative_day": "1999-11-29", "each_day": -0.0373285681011555}, {"indicative_day": "1999-11-3", "each_day": -0.014390500662336733}, {"indicative_day": "1999-11-30", "each_day": 0.06504403049763549}, {"indicative_day": "1999-11-4", "each_day": 0.030162724528828257}, {"indicative_day": "1999-11-5", "each_day": -0.042835992621528804}, {"indicative_day": "1999-11-6", "each_day": -0.06889453215681651}, {"indicative_day": "1999-11-7", "each_day": -0.023822405204814275}, {"indicative_day": "1999-11-8", "each_day": -0.02166626989496563}, {"indicative_day": "1999-11-9", "each_day": -0.025578211476639627}, {"indicative_day": "1999-12-1", "each_day": 0.10401354228046564}, {"indicative_day": "1999-12-10", "each_day": -0.13359609396649152}, {"indicative_day": "1999-12-11", "each_day": -0.16188036922299118}, {"indicative_day": "1999-12-12", "each_day": -0.06760472655291146}, {"indicative_day": "1999-12-13", "each_day": -0.2085361305989857}, {"indicative_day": "1999-12-14", "each_day": -0.052789447207455896}, {"indicative_day": "1999-12-15", "each_day": 0.09893610295773993}, {"indicative_day": "1999-12-16", "each_day": 0.19431548118825498}, {"indicative_day": "1999-12-17", "each_day": 0.19630601642467238}, {"indicative_day": "1999-12-18", "each_day": 0.2488535058391148}, {"indicative_day": "1999-12-19", "each_day": 0.32395140300890324}, {"indicative_day": "1999-12-2", "each_day": 0.0759026251936204}, {"indicative_day": "1999-12-20", "each_day": 0.2566057775273213}, {"indicative_day": "1999-12-21", "each_day": 0.11914363975423115}, {"indicative_day": "1999-12-22", "each_day": -0.21284528491705623}, {"indicative_day": "1999-12-23", "each_day": -0.548895074575928}, {"indicative_day": "1999-12-24", "each_day": -1.1273516996711745}, {"indicative_day": "1999-12-25", "each_day": -1.5481635013715147}, {"indicative_day": "1999-12-26", "each_day": -0.577772031899057}, {"indicative_day": "1999-12-27", "each_day": 0.32100230861634876}, {"indicative_day": "1999-12-28", "each_day": 0.6166993288924372}, {"indicative_day": "1999-12-29", "each_day": 0.6618072434220138}, {"indicative_day": "1999-12-3", "each_day": -0.05317939003583895}, {"indicative_day": "1999-12-30", "each_day": 0.758022899242529}, {"indicative_day": "1999-12-31", "each_day": 0.2143427427249748}, {"indicative_day": "1999-12-4", "each_day": -0.07758362788593239}, {"indicative_day": "1999-12-5", "each_day": -0.06237055231803472}, {"indicative_day": "1999-12-6", "each_day": -0.11300850513852219}, {"indicative_day": "1999-12-7", "each_day": -0.15229887231388348}, {"indicative_day": "1999-12-8", "each_day": -0.07210520053825505}, {"indicative_day": "1999-12-9", "each_day": -0.11129442326962635}, {"indicative_day": "1999-2-1", "each_day": -0.06435705205543062}, {"indicative_day": "1999-2-10", "each_day": -0.02547983596826729}, {"indicative_day": "1999-2-11", "each_day": -0.07859959727148906}, {"indicative_day": "1999-2-12", "each_day": -0.00489352145947125}, {"indicative_day": "1999-2-13", "each_day": -0.2530180114222627}, {"indicative_day": "1999-2-14", "each_day": 0.216600578388773}, {"indicative_day": "1999-2-15", "each_day": -0.05332339130364079}, {"indicative_day": "1999-2-16", "each_day": -0.08735200557748343}, {"indicative_day": "1999-2-17", "each_day": -0.04637955956264119}, {"indicative_day": "1999-2-18", "each_day": -0.06994953126946553}, {"indicative_day": "1999-2-19", "each_day": -0.13044081371976887}, {"indicative_day": "1999-2-2", "each_day": 0.002222370899137386}, {"indicative_day": "1999-2-20", "each_day": -0.043533187078643995}, {"indicative_day": "1999-2-21", "each_day": -0.1142196099616656}, {"indicative_day": "1999-2-22", "each_day": 0.02935696990236951}, {"indicative_day": "1999-2-23", "each_day": -0.03611217531919976}, {"indicative_day": "1999-2-24", "each_day": 0.0181391913842484}, {"indicative_day": "1999-2-25", "each_day": -0.0021711879290748776}, {"indicative_day": "1999-2-26", "each_day": -0.06933866755766956}, {"indicative_day": "1999-2-27", "each_day": -0.1593626855923896}, {"indicative_day": "1999-2-28", "each_day": -0.09216299325839608}, {"indicative_day": "1999-2-29", "each_day": -0.44549831550734603}, {"indicative_day": "1999-2-3", "each_day": -0.04066016379047365}, {"indicative_day": "1999-2-4", "each_day": -0.11810217557578499}, {"indicative_day": "1999-2-5", "each_day": -0.12631100446679439}, {"indicative_day": "1999-2-6", "each_day": -0.1393593847629202}, {"indicative_day": "1999-2-7", "each_day": -0.1324764892376433}, {"indicative_day": "1999-2-8", "each_day": -0.10013121692211198}, {"indicative_day": "1999-2-9", "each_day": -0.09887056478735885}, {"indicative_day": "1999-3-1", "each_day": -0.010503177820420224}, {"indicative_day": "1999-3-10", "each_day": -0.0414834631652652}, {"indicative_day": "1999-3-11", "each_day": -0.03994453048199231}, {"indicative_day": "1999-3-12", "each_day": -0.07041506645327267}, {"indicative_day": "1999-3-13", "each_day": -0.19688582492442114}, {"indicative_day": "1999-3-14", "each_day": -0.07248640645931277}, {"indicative_day": "1999-3-15", "each_day": -0.0999839026775785}, {"indicative_day": "1999-3-16", "each_day": -0.09493473781645842}, {"indicative_day": "1999-3-17", "each_day": 0.04060062388540702}, {"indicative_day": "1999-3-18", "each_day": -0.06460253583197259}, {"indicative_day": "1999-3-19", "each_day": -0.10066636694318276}, {"indicative_day": "1999-3-2", "each_day": -0.0493062641142164}, {"indicative_day": "1999-3-20", "each_day": -0.09006730494194792}, {"indicative_day": "1999-3-21", "each_day": -0.07642175739316939}, {"indicative_day": "1999-3-22", "each_day": -0.10571058527452382}, {"indicative_day": "1999-3-23", "each_day": -0.10288699440313592}, {"indicative_day": "1999-3-24", "each_day": -0.1338877842095682}, {"indicative_day": "1999-3-25", "each_day": -0.04901907245274005}, {"indicative_day": "1999-3-26", "each_day": -0.12302799508038144}, {"indicative_day": "1999-3-27", "each_day": -0.12341707152578704}, {"indicative_day": "1999-3-28", "each_day": -0.08558146693346781}, {"indicative_day": "1999-3-29", "each_day": -0.07159252758886636}, {"indicative_day": "1999-3-3", "each_day": 0.08125596181417002}, {"indicative_day": "1999-3-30", "each_day": -0.12466324642339413}, {"indicative_day": "1999-3-31", "each_day": -0.10096162051643393}, {"indicative_day": "1999-3-4", "each_day": -0.004549851141679937}, {"indicative_day": "1999-3-5", "each_day": -0.06574174837981239}, {"indicative_day": "1999-3-6", "each_day": -0.10283132762499585}, {"indicative_day": "1999-3-7", "each_day": -0.09074244245373297}, {"indicative_day": "1999-3-8", "each_day": -0.0240996356994655}, {"indicative_day": "1999-3-9", "each_day": -0.12307067467508097}, {"indicative_day": "1999-4-1", "each_day": -0.27813009284079404}, {"indicative_day": "1999-4-10", "each_day": -0.1983617499587368}, {"indicative_day": "1999-4-11", "each_day": -0.18922493945497443}, {"indicative_day": "1999-4-12", "each_day": -0.1558209654000536}, {"indicative_day": "1999-4-13", "each_day": -0.2647262766191971}, {"indicative_day": "1999-4-14", "each_day": -0.13903648015305498}, {"indicative_day": "1999-4-15", "each_day": -0.09413870724901886}, {"indicative_day": "1999-4-16", "each_day": -0.16627569810560888}, {"indicative_day": "1999-4-17", "each_day": -0.1935266123402108}, {"indicative_day": "1999-4-18", "each_day": -0.13419590492530956}, {"indicative_day": "1999-4-19", "each_day": -0.15389974316949076}, {"indicative_day": "1999-4-2", "each_day": -0.048032555383991794}, {"indicative_day": "1999-4-20", "each_day": -0.20410370016894946}, {"indicative_day": "1999-4-21", "each_day": -0.16867498926517982}, {"indicative_day": "1999-4-22", "each_day": -0.12012110437665081}, {"indicative_day": "1999-4-23", "each_day": -0.18513807494689016}, {"indicative_day": "1999-4-24", "each_day": -0.23749166137360025}, {"indicative_day": "1999-4-25", "each_day": -0.1938431165747881}, {"indicative_day": "1999-4-26", "each_day": -0.15539881202266573}, {"indicative_day": "1999-4-27", "each_day": -0.23751519757953485}, {"indicative_day": "1999-4-28", "each_day": -0.17453476262401904}, {"indicative_day": "1999-4-29", "each_day": -0.25131263201931875}, {"indicative_day": "1999-4-3", "each_day": -0.1402776478548151}, {"indicative_day": "1999-4-30", "each_day": -0.23902573497889681}, {"indicative_day": "1999-4-4", "each_day": -0.08807977708551251}, {"indicative_day": "1999-4-5", "each_day": -0.18865467704934713}, {"indicative_day": "1999-4-6", "each_day": -0.14206587723815636}, {"indicative_day": "1999-4-7", "each_day": -0.13869772175982817}, {"indicative_day": "1999-4-8", "each_day": -0.10365725232611556}, {"indicative_day": "1999-4-9", "each_day": -0.20072669665329343}, {"indicative_day": "1999-5-1", "each_day": -0.10773736812802982}, {"indicative_day": "1999-5-10", "each_day": -0.1170170557241572}, {"indicative_day": "1999-5-11", "each_day": -0.129733452815525}, {"indicative_day": "1999-5-12", "each_day": -0.14023540765404738}, {"indicative_day": "1999-5-13", "each_day": -0.21604991315333294}, {"indicative_day": "1999-5-14", "each_day": -0.19620426022114032}, {"indicative_day": "1999-5-15", "each_day": -0.12259273552223587}, {"indicative_day": "1999-5-16", "each_day": -0.15048456199136465}, {"indicative_day": "1999-5-17", "each_day": -0.14953105405337247}, {"indicative_day": "1999-5-18", "each_day": -0.12373936820474798}, {"indicative_day": "1999-5-19", "each_day": -0.1263494301757375}, {"indicative_day": "1999-5-2", "each_day": -0.15865650347302923}, {"indicative_day": "1999-5-20", "each_day": 0.023363982654586894}, {"indicative_day": "1999-5-21", "each_day": -0.11170212236648769}, {"indicative_day": "1999-5-22", "each_day": -0.06681430686027666}, {"indicative_day": "1999-5-23", "each_day": -0.04819078827573551}, {"indicative_day": "1999-5-24", "each_day": -0.041344016816070604}, {"indicative_day": "1999-5-25", "each_day": -0.14247268313353828}, {"indicative_day": "1999-5-26", "each_day": -0.21993304274726752}, {"indicative_day": "1999-5-27", "each_day": -0.09739818525245514}, {"indicative_day": "1999-5-28", "each_day": -0.20795025112031723}, {"indicative_day": "1999-5-29", "each_day": -0.07381276800773288}, {"indicative_day": "1999-5-3", "each_day": -0.18173242053280464}, {"indicative_day": "1999-5-30", "each_day": -0.2712715632682913}, {"indicative_day": "1999-5-31", "each_day": -0.20582763655059272}, {"indicative_day": "1999-5-4", "each_day": -0.2232743072411659}, {"indicative_day": "1999-5-5", "each_day": -0.10808888418146147}, {"indicative_day": "1999-5-6", "each_day": -0.13569847573247174}, {"indicative_day": "1999-5-7", "each_day": -0.1660308596346734}, {"indicative_day": "1999-5-8", "each_day": -0.16687920605317946}, {"indicative_day": "1999-5-9", "each_day": -0.20131928229046758}, {"indicative_day": "1999-6-1", "each_day": 0.0031393828060392428}, {"indicative_day": "1999-6-10", "each_day": 0.020375657469869888}, {"indicative_day": "1999-6-11", "each_day": -0.07536102595926605}, {"indicative_day": "1999-6-12", "each_day": -0.06301264390616398}, {"indicative_day": "1999-6-13", "each_day": -0.1927039058781133}, {"indicative_day": "1999-6-14", "each_day": -0.04448357339348485}, {"indicative_day": "1999-6-15", "each_day": -0.056497299033614855}, {"indicative_day": "1999-6-16", "each_day": 0.0449718850629673}, {"indicative_day": "1999-6-17", "each_day": 0.04202003315778706}, {"indicative_day": "1999-6-18", "each_day": -0.04220860251531061}, {"indicative_day": "1999-6-19", "each_day": -0.07015889910764217}, {"indicative_day": "1999-6-2", "each_day": 0.006004489749509827}, {"indicative_day": "1999-6-20", "each_day": 0.04036409398708781}, {"indicative_day": "1999-6-21", "each_day": -0.049016920846756046}, {"indicative_day": "1999-6-22", "each_day": -0.029756572704545407}, {"indicative_day": "1999-6-23", "each_day": -0.030250205211860587}, {"indicative_day": "1999-6-24", "each_day": 0.04509681824756376}, {"indicative_day": "1999-6-25", "each_day": 0.03934134598483708}, {"indicative_day": "1999-6-26", "each_day": 0.03710966324617169}, {"indicative_day": "1999-6-27", "each_day": 0.03706598384207201}, {"indicative_day": "1999-6-28", "each_day": 0.11552889597889687}, {"indicative_day": "1999-6-29", "each_day": 0.09000949710584007}, {"indicative_day": "1999-6-3", "each_day": -0.041909219225374245}, {"indicative_day": "1999-6-30", "each_day": 0.17955007669869336}, {"indicative_day": "1999-6-4", "each_day": -0.10737931970041374}, {"indicative_day": "1999-6-5", "each_day": -0.10710140528374078}, {"indicative_day": "1999-6-6", "each_day": 0.005620646150415896}, {"indicative_day": "1999-6-7", "each_day": -0.09151471433257388}, {"indicative_day": "1999-6-8", "each_day": -0.14809635957494485}, {"indicative_day": "1999-6-9", "each_day": -0.05260643262436998}, {"indicative_day": "1999-7-1", "each_day": 0.2454452086345132}, {"indicative_day": "1999-7-10", "each_day": 0.2535695046449866}, {"indicative_day": "1999-7-11", "each_day": 0.13445467801935007}, {"indicative_day": "1999-7-12", "each_day": 0.10106931997093069}, {"indicative_day": "1999-7-13", "each_day": 0.02859601809172114}, {"indicative_day": "1999-7-14", "each_day": 0.24145060249867914}, {"indicative_day": "1999-7-15", "each_day": 0.2917574286799216}, {"indicative_day": "1999-7-16", "each_day": 0.2083251954797649}, {"indicative_day": "1999-7-17", "each_day": 0.19049432002148037}, {"indicative_day": "1999-7-18", "each_day": 0.11279687915007211}, {"indicative_day": "1999-7-19", "each_day": 0.12492356547410453}, {"indicative_day": "1999-7-2", "each_day": 0.2471222204953866}, {"indicative_day": "1999-7-20", "each_day": 0.22730202650647152}, {"indicative_day": "1999-7-21", "each_day": 0.19374655609743657}, {"indicative_day": "1999-7-22", "each_day": 0.281150036969938}, {"indicative_day": "1999-7-23", "each_day": 0.21279038616153056}, {"indicative_day": "1999-7-24", "each_day": 0.18193566462857125}, {"indicative_day": "1999-7-25", "each_day": 0.1811923123857468}, {"indicative_day": "1999-7-26", "each_day": 0.1897136387289007}, {"indicative_day": "1999-7-27", "each_day": 0.2564117506058412}, {"indicative_day": "1999-7-28", "each_day": 0.2797254144166218}, {"indicative_day": "1999-7-29", "each_day": 0.30681610065507636}, {"indicative_day": "1999-7-3", "each_day": 0.04160125083386287}, {"indicative_day": "1999-7-30", "each_day": 0.2212272267263563}, {"indicative_day": "1999-7-31", "each_day": 0.18226974437994667}, {"indicative_day": "1999-7-4", "each_day": -0.9173376168596974}, {"indicative_day": "1999-7-5", "each_day": -0.20499297465590618}, {"indicative_day": "1999-7-6", "each_day": 0.22734043509399415}, {"indicative_day": "1999-7-7", "each_day": 0.42619554545469523}, {"indicative_day": "1999-7-8", "each_day": 0.4151204436215464}, {"indicative_day": "1999-7-9", "each_day": 0.26541166611286476}, {"indicative_day": "1999-8-1", "each_day": 0.2219734093079857}, {"indicative_day": "1999-8-10", "each_day": 0.2542582438704688}, {"indicative_day": "1999-8-11", "each_day": 0.2022414488256}, {"indicative_day": "1999-8-12", "each_day": 0.3333299057905831}, {"indicative_day": "1999-8-13", "each_day": 0.16290892623806216}, {"indicative_day": "1999-8-14", "each_day": 0.2650724194957417}, {"indicative_day": "1999-8-15", "each_day": 0.2831040030741819}, {"indicative_day": "1999-8-16", "each_day": 0.23676588632206677}, {"indicative_day": "1999-8-17", "each_day": 0.2250079750530002}, {"indicative_day": "1999-8-18", "each_day": 0.2649634778923896}, {"indicative_day": "1999-8-19", "each_day": 0.270077565447041}, {"indicative_day": "1999-8-2", "each_day": 0.17841942456762}, {"indicative_day": "1999-8-20", "each_day": 0.25514799207548566}, {"indicative_day": "1999-8-21", "each_day": 0.2123443851739204}, {"indicative_day": "1999-8-22", "each_day": 0.18512898072052084}, {"indicative_day": "1999-8-23", "each_day": 0.15733066710659077}, {"indicative_day": "1999-8-24", "each_day": 0.16321014360438146}, {"indicative_day": "1999-8-25", "each_day": 0.21367218817806022}, {"indicative_day": "1999-8-26", "each_day": 0.3241495055001735}, {"indicative_day": "1999-8-27", "each_day": 0.2720616673968796}, {"indicative_day": "1999-8-28", "each_day": 0.30608224866556355}, {"indicative_day": "1999-8-29", "each_day": 0.2567738461058104}, {"indicative_day": "1999-8-3", "each_day": 0.21724239019405028}, {"indicative_day": "1999-8-30", "each_day": 0.2746452681732259}, {"indicative_day": "1999-8-31", "each_day": 0.26349011297317615}, {"indicative_day": "1999-8-4", "each_day": 0.16894387101379701}, {"indicative_day": "1999-8-5", "each_day": 0.20483418135890546}, {"indicative_day": "1999-8-6", "each_day": 0.21857159334781645}, {"indicative_day": "1999-8-7", "each_day": 0.23299080505285277}, {"indicative_day": "1999-8-8", "each_day": 0.36742669744630757}, {"indicative_day": "1999-8-9", "each_day": 0.21175001728361545}, {"indicative_day": "1999-9-1", "each_day": -0.06585960759346099}, {"indicative_day": "1999-9-10", "each_day": 0.36102343182080937}, {"indicative_day": "1999-9-11", "each_day": 0.29867270422110004}, {"indicative_day": "1999-9-12", "each_day": 0.2973214291365146}, {"indicative_day": "1999-9-13", "each_day": 0.1801757429026565}, {"indicative_day": "1999-9-14", "each_day": 0.3423339938280492}, {"indicative_day": "1999-9-15", "each_day": 0.32956233575004656}, {"indicative_day": "1999-9-16", "each_day": 0.43465697534240216}, {"indicative_day": "1999-9-17", "each_day": 0.35897833525104367}, {"indicative_day": "1999-9-18", "each_day": 0.35449977476647154}, {"indicative_day": "1999-9-19", "each_day": 0.3811784184389873}, {"indicative_day": "1999-9-2", "each_day": 0.05103112458139436}, {"indicative_day": "1999-9-20", "each_day": 0.42353301647346064}, {"indicative_day": "1999-9-21", "each_day": 0.41018707515067226}, {"indicative_day": "1999-9-22", "each_day": 0.4084152924209283}, {"indicative_day": "1999-9-23", "each_day": 0.4191223432066638}, {"indicative_day": "1999-9-24", "each_day": 0.37188527140754524}, {"indicative_day": "1999-9-25", "each_day": 0.3701597397391732}, {"indicative_day": "1999-9-26", "each_day": 0.408371057246445}, {"indicative_day": "1999-9-27", "each_day": 0.37489745046336526}, {"indicative_day": "1999-9-28", "each_day": 0.29524320383300207}, {"indicative_day": "1999-9-29", "each_day": 0.31193032592517655}, {"indicative_day": "1999-9-3", "each_day": 0.10156705083010882}, {"indicative_day": "1999-9-30", "each_day": 0.34312741580732004}, {"indicative_day": "1999-9-4", "each_day": 0.1912222605888119}, {"indicative_day": "1999-9-5", "each_day": 0.07735099224308409}, {"indicative_day": "1999-9-6", "each_day": 0.13117816786963202}, {"indicative_day": "1999-9-7", "each_day": 0.11966764559793115}, {"indicative_day": "1999-9-8", "each_day": 0.3148529094861128}, {"indicative_day": "1999-9-9", "each_day": 0.46369691963564524}], "data-6490e5d4bbc61f7fe504b170ff6e392e": [{"holiday": "Christmas Day", "ds": "1969-12-25T00:00:00", "effect size": -1.5481635013715147, "indicative_day": "1999-12-25"}, {"holiday": "Columbus Day", "ds": "1969-10-12T00:00:00", "effect size": 0.06828880393034772, "indicative_day": "1999-10-12"}, {"holiday": "Independence Day", "ds": "1969-07-04T00:00:00", "effect size": -0.9173376168596974, "indicative_day": "1999-7-4"}, {"holiday": "Labor Day", "ds": "1969-09-01T00:00:00", "effect size": 0.005487652338790151, "indicative_day": "1999-9-1"}, {"holiday": "Martin Luther King Jr. Day", "ds": "1986-01-20T00:00:00", "effect size": -0.07579941928242043, "indicative_day": "1999-1-20"}, {"holiday": "Memorial Day", "ds": "1969-05-30T00:00:00", "effect size": -0.26028451994984686, "indicative_day": "1999-5-30"}, {"holiday": "New Year's Day", "ds": "1969-01-01T00:00:00", "effect size": -1.3016875285020422, "indicative_day": "1999-1-1"}, {"holiday": "Thanksgiving", "ds": "1969-11-27T00:00:00", "effect size": -0.35470997764938367, "indicative_day": "1999-11-27"}, {"holiday": "Veterans Day", "ds": "1969-11-11T00:00:00", "effect size": -0.031176605230960653, "indicative_day": "1999-11-11"}, {"holiday": "Washington's Birthday", "ds": "1969-02-22T00:00:00", "effect size": -0.08999410110108104, "indicative_day": "1999-2-22"}]}}, {"mode": "vega-lite"});
</script>




```python

```
