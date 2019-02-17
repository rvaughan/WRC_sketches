---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.0'
      jupytext_version: 0.8.6
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

```python
import notebookimport

ds= __import__("Report - Driver Splits")
```

```python
import sqlite3
from IPython.display import HTML
import pandas as pd

dbname='sweden19.db'
conn = sqlite3.connect(dbname)
rally='Sweden'
rc='RC1'
year=2019
```

```python
q_drivers='SELECT `driver.code` AS code  FROM startlists st JOIN startlist_classes sc ON sc.entryid = st.entryid AND name="{}"'

q_classes='SELECT DISTINCT(name) AS name FROM startlists st JOIN startlist_classes sc ON sc.entryid = st.entryid'

#Need to look this up properly
q_stages= ['SS{}'.format(i) for i in range(1,17) ]
```

```python
def stage_chart(rc,driver, stage):
    s2 = ds.getDriverSplitsReport(conn, rally, stage, driver, rc)
    display(HTML(s2))
```

```python
import ipywidgets as widgets
from ipywidgets import interact

classes = widgets.Dropdown(
    options=pd.read_sql(q_classes,conn)['name'].to_list(),
    value='RC1', description='Class:', disabled=False )

drivers = widgets.Dropdown(
    options=pd.read_sql(q_drivers.format(classes.value),conn)['code'].to_list(),
    description='Driver:', disabled=False)

stages = widgets.Dropdown(
    options=q_stages, description='Stage:', disabled=False)

def update_drivers(*args):
    qdrivers = q_drivers.format(classes.value)
    driverlist = pd.read_sql(qdrivers,conn)['code'].to_list()
    print(driverlist)
    drivers.options = driverlist
    
classes.observe(update_drivers, 'value')

interact(stage_chart, rc=classes, driver=drivers, stage=stages);
```

## How About More Control?

What about we let the user decide what to allow in each chart?

```python
classes2 = widgets.Dropdown(
    options=pd.read_sql(q_classes,conn)['name'].to_list(),
    value='RC1',
    description='Class:',
    disabled=False,
)

drivers2 = widgets.Dropdown(
    options=pd.read_sql(q_drivers.format(classes2.value),conn)['code'].to_list(),
    #value='NEU',
    description='Driver:',
    disabled=False,
)

stages2 = widgets.Dropdown(
    options=q_stages, description='Stage:', disabled=False)


def stage_chart2(rc,driver, stage, bars, roadPos, waypointRank):
    dropcols=[]
    if not roadPos:
        dropcols.append('Road Position')
    if not waypointRank:
        dropcols.append('Waypoint Rank')
        
    s2 = ds.getDriverSplitsReport(conn, rally, stage, driver, rc,
                                  bars=bars, dropcols=dropcols)
    display(HTML(s2))

splitBars = widgets.Checkbox( value=True, description='Split bars:',
                           disabled=False )
    
roadPos = widgets.Checkbox( value=True, description='Road pos:',
                           disabled=False )

waypointRank = widgets.Checkbox( value=True, description='Waypoint Rank:',
                           disabled=False )

def update_drivers2(*args):
    qdrivers = q_drivers.format(classes2.value)
    driverlist = pd.read_sql(qdrivers,conn)['code'].to_list()
    print(driverlist)
    drivers2.options = driverlist
    
classes2.observe(update_drivers2, 'value')


interact(stage_chart2, rc=classes2, driver=drivers2,
         stage=stages2, bars=splitBars, roadPos=roadPos,
        waypointRank=waypointRank);
```

```python

```