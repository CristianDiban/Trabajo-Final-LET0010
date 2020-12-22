```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import plotly.express as px
import seaborn as sns
import plotly.graph_objects as go
```


```python
futbol=pd.read_csv('datos-sin-procesar/results.csv',header=0,parse_dates=[0])
futbol.head()
```


```python
futbol['tournament'].value_counts()
```


```python
futbol.groupby(futbol['date'].dt.year)['tournament'].count().plot(kind='bar',figsize=(15,10))
```


```python
futbol.dtypes
futbol['date']=pd.to_datetime(futbol['date'])
futbol.isna().any()

futbol['Year']=futbol['date'].dt.year
futbol['Count']=1
```


```python
futbol_year=futbol.groupby('Year')['Count'].sum().reset_index()
```


```python
fig1=px.line(futbol_year,y='Count',x='Year',height=600,width=800)
fig1.update_layout(title='Número de partidos jugados por año',title_x=0.5)
fig1.update_traces(line_color='#FF0000')
fig1.show()
```


```python
futbol_h=futbol.groupby('home_team')['Count'].sum().reset_index().sort_values(by='Count',ascending=False).head(20)
futbol_a=futbol.groupby('away_team')['Count'].sum().reset_index().sort_values(by='Count',ascending=False).head(20)
```


```python
fig2=plt.figure(figsize=(20,15))
ax1=fig2.add_subplot(211)
sns.barplot('home_team','Count',data=futbol_h,ax=ax1,palette='rocket')
label=futbol_h['home_team']
ax1.set_xticklabels(label,rotation=90,size=15)
ax1.set_title('20 equipos más activos de local',size=25)
ax1.set_xlabel('Local',size=15)
ax1.set_ylabel('Partidos jugados',size=15)

ax2=fig2.add_subplot(212)
sns.barplot('away_team','Count',data=futbol_a,ax=ax2,palette='summer')
label=futbol_a['away_team']
ax2.set_xticklabels(label,rotation=90,size=15)
ax2.set_title('20 equipos más activos de visitante',size=25)
ax2.set_xlabel('Visita',size=15)
ax2.set_ylabel('Partidos jugados',size=15)

fig2.tight_layout(pad=3)
```


```python
futbol_tour=futbol.groupby('tournament')['Count'].sum().reset_index()
fig3=px.pie(futbol_tour,values='Count',names='tournament',hole=0.3)
fig3.update_layout(title='Distribución de cantidad de partidos por torneo',title_x=0.25)
fig3.show()
```


```python
futbol['Dif. de gol']=abs(futbol['home_score']-futbol['away_score'])
```


```python
fig4=px.histogram(futbol,x='Dif. de gol',marginal='violin')
fig4.update_layout(title= 'Distribución de la diferencia de gol',title_x = 0.5)
fig4.update_traces(opacity=0.9)
fig4.show()
```


```python
futbol['Total goals']=futbol['home_score']+futbol['away_score']
```


```python
futbol_gpg=futbol.groupby('tournament')[['Count','Total goals']].sum().reset_index()
futbol_gpg['GPG']=np.round(futbol_gpg['Total goals']/futbol_gpg['Count'],0)
futbol_gpg.sort_values(by='GPG',ascending=False,inplace=True)
```


```python
fig5=px.bar(futbol_gpg,x='tournament',y='GPG',color='GPG',
            height=800,width=1000,labels={'GPG':'Average goals per game','tournament':'Tournament'})
fig5.show()
```


```python
futbol_nn=futbol[futbol['neutral']==False]

futbol_gpg_home=futbol_nn.groupby('home_team')[['Count','home_score']].sum().reset_index()
futbol_gpg_home=futbol_gpg_home[futbol_gpg_home['Count']>30]
futbol_gpg_home['GPG']=np.round(futbol_gpg_home['home_score']/futbol_gpg_home['Count'],0)
futbol_gpg_home.sort_values(by='GPG',ascending=False,inplace=True)

futbol_gpg_away=futbol_nn.groupby('home_team')[['Count','away_score']].sum().reset_index()
futbol_gpg_away=futbol_gpg_away[futbol_gpg_away['Count']>30]
futbol_gpg_away['GPG']=np.round(futbol_gpg_away['away_score']/futbol_gpg_away['Count'],0)
futbol_gpg_away.sort_values(by='GPG',ascending=False,inplace=True)
```


```python
futbol_gpg_home=futbol_gpg_home.head(20)
futbol_gpg_away=futbol_gpg_away.head(20)
```


```python
fig6=plt.figure(figsize=(15,15))
ax1=fig6.add_subplot(211)
sns.barplot('home_team','GPG',data=futbol_gpg_home,palette='gist_earth',ax=ax1)
label1=futbol_gpg_home['home_team']
ax1.set_xticklabels(label1,rotation=75)
ax1.set_xlabel('Team name',size=15)
ax1.set_ylabel('Promedio de gol por partido',size=10)
ax1.set_title('Top 20 scoring home teams',size=25)

ax2=fig6.add_subplot(212)
sns.barplot('home_team','GPG',data=futbol_gpg_away,palette='coolwarm',ax=ax2)
label2=futbol_gpg_away['home_team']
ax2.set_xticklabels(label2,rotation=75)
ax2.set_xlabel('Nombre Selección',size=15)
ax2.set_ylabel('Promedio de goles por partido',size=10)
ax2.set_title('Top 20 scoring away teams',size=25)

fig6.tight_layout(pad=3)
```


```python
futbol_venue=futbol.groupby('neutral')['Count'].sum().reset_index()
```


```python
fig7 = px.pie(futbol_venue,values='Count',names='neutral',hole=0.4,color_discrete_sequence=['red','blue'])

fig7.update_layout(title='Tipo de localía',
                   title_x=.5,
                   annotations=[dict(text='Campo Neutral',font_size=15, showarrow=False,height=800,width=700)])

fig7.update_traces(textfont_size=15,textinfo='percent+label')


fig7.show()
```


```python
futbol['total_score'] = futbol['home_score']+futbol['away_score']
futbol_score = futbol['total_score'].value_counts().to_frame().reset_index().rename(columns={'index':'Total_Score','total_score':'Count'}).sort_values('Total_Score',ascending="False")

fig = go.Figure(go.Bar(
    x=futbol_score['Total_Score'],y=futbol_score['Count'],
    marker={'color': futbol_score['Count'], 
    'colorscale': 'Viridis'},  
    text=futbol_score['Count'],
    textposition = "outside",
))

fig.update_layout(title_text='Distribución de Goles',
                  xaxis_title="Goles",
                  yaxis_title="Cantidad",
                  title_x=0.5)
fig.show()
```


```python
# Where was country matche was held?
fig = px.scatter_geo(data_frame=futbol, 
                     locations= 'country',
                     locationmode='country names',
                     animation_frame='tournament',
                    title = '<b>Country where match played')
fig.show()
```


```python
# Which countries host the most matches where they themselves are not participating in

fig = px.choropleth(locations= futbol.query("neutral == True")['country'].value_counts().index,
              locationmode='country names',
              color= futbol.query("neutral == True")['country'].value_counts(),
                   color_continuous_scale = 'Reds',
                    title = '<b>Countries host not participating',
                   labels = {'color' : 'Match played'})
fig.show()
```


```python
# Where was country matches was held?
fi = px.scatter_geo(data_frame=futbol, 
                    locations= 'country',
                    locationmode='country names',
                    animation_frame='date',
                    title = '<b>Country where match played')
fi.show()
```


```python

```
