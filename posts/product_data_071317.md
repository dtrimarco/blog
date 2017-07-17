
# Pandas for Product Analysis Part 1: Apply and Transform
 
Python's [pandas](http://pandas.pydata.org/) package is one of the most powerful tools for data analysis in the Python ecosystem. Built on top of NumPy, it makes working with tabular data quite effective and adds an astounding amount of functionality to your toolkit. Despite its strengths, there are some very useful functions that are challenging to grasp based on the pandas docs. `apply` and `transform` are two such examples. 

One quick note before we dive in: this series assumes basic working knowledge of pandas. There are several resources like [Dataquest](https://dataquest.io), [Data Camp](https://www.datacamp.com/) and pandas cheat sheets to get you up to speed if this is hard to follow.

## What are `apply` and `transform`?

In short, these two functions are used to operate on data structures, similarly to Python's built in `map` function. We will get into the differences, but typically they are used in combination with `groupby` to perform aggregate functions on various groups of a dataset. This a direct analogy to `GROUP BY` in SQL and I am going to assume familiarity with how it works (if you aren't, [here is a decent intro](http://pandas.pydata.org/pandas-docs/stable/groupby.html#splitting-an-object-into-groups)). The major difference is that we can leverage the flexibility of Python and pandas DataFrames to do basically whatever we want.

## Data
To keep things practical, let's start with event data from a hypothetical mobile game. I created some [randomly generated, but logical data](https://github.com/dtrimarco/blog/blob/master/utils/create_fake_data.ipynb) for us to analyze.


```python
import pandas as pd

data = pd.read_csv('test_user_data.csv')
print(data.head(10))
```

       user_id      event_timestamp        lat        lon event_type
    0     5000  2016-01-03 00:40:59  41.795738  87.517197      login
    1     5000  2016-01-03 00:50:59  41.795738  87.517197    level_1
    2     5000  2016-01-03 00:56:59  41.795738  87.517197    level_2
    3     5000  2016-01-03 01:01:59  41.795738  87.517197    level_3
    4     5000  2016-01-03 01:13:59  41.795738  87.517197    level_4
    5     5000  2016-01-03 01:28:59  41.795738  87.517197    level_5
    6     5000  2016-01-03 01:35:59  41.795738  87.517197    level_6
    7     5000  2016-01-03 01:45:59  41.795738  87.517197    level_7
    8     5000  2016-01-17 10:07:19  41.593679  87.719833      login
    9     5000  2016-01-17 10:19:19  41.593679  87.719833    level_8


The data contains one event per row and has 5 variables:

* **user_id**: Identifier for each user.
* **event_timestamp**: The time each event happened.
* **lat**: The latitude of the user when the event occurred.
* **lon**: The longitude of the user when the event occurred.
* **event_type**: The type of event that occurred: login, level, buy_coins and megapack.

## Basic differences between `apply` and `transform`

Suppose we wanted to count the number of events for each user. Both functions can do this, but in different ways. Let's try it first with `apply`.


```python
apply_ex = data.groupby('user_id').apply(len)
print(apply_ex.head())
```

    user_id
    5000    230
    5001    207
    5002    242
    5003    190
    5004    116
    dtype: int64


The output here is a pandas Series with each user_id as the index and the count of the number of events as values. Now to try the same thing with `transform`.


```python
transform_ex = data.groupby('user_id').transform(len)
print(transform_ex.head())
```

       event_timestamp  lat  lon  event_type
    0              230  230  230         230
    1              230  230  230         230
    2              230  230  230         230
    3              230  230  230         230
    4              230  230  230         230


What the heck happened here? This odd DataFrame highlights a key difference: `apply` by default returns an object with one element per group and `transform` returns an object of the exact same size as the input object. Unless specified, it operates column by column in order.

How about we clean this up a bit and create a new column in our original DataFrame that contains the total event count for each group in it.


```python
data['event_count'] = data.groupby('user_id')['user_id'].transform(len)
print(data.head(7))
```

       user_id      event_timestamp        lat        lon event_type  event_count
    0     5000  2016-01-03 00:40:59  41.795738  87.517197      login          230
    1     5000  2016-01-03 00:50:59  41.795738  87.517197    level_1          230
    2     5000  2016-01-03 00:56:59  41.795738  87.517197    level_2          230
    3     5000  2016-01-03 01:01:59  41.795738  87.517197    level_3          230
    4     5000  2016-01-03 01:13:59  41.795738  87.517197    level_4          230
    5     5000  2016-01-03 01:28:59  41.795738  87.517197    level_5          230
    6     5000  2016-01-03 01:35:59  41.795738  87.517197    level_6          230


Much better. All we had to do was assign to the new `event_count` column and then specify the `['user_id']` column after the `groupby` statement. Whether you would prefer to have this additional column of repeating values depends on what you intend to do with the data afterwards. Let's assume this is acceptable. Now for something a bit more involved.

## Custom Functions

Say we didn't have Google Analytics or Mixpanel implemented into our app and wanted to assign a monetary value to each event. Of course, we could loop through the entire DataFrame, but this can be very inefficient with a lot of data. Let's try it using a custom function.


```python
def add_value(x):
    if x == 'buy_coins':
        y = 1.00
    elif x == 'megapack':
        y = 10.00
    else:
        y=0.0
    
    return y
```

Here we've defined a very simple custom function that assigns values to each of the four event types. Now to `apply` it to our data.


```python
data['event_value'] = data['event_type'].apply(add_value)
print(data.head(7))
```

       user_id      event_timestamp        lat        lon event_type  event_count  \
    0     5000  2016-01-03 00:40:59  41.795738  87.517197      login          230   
    1     5000  2016-01-03 00:50:59  41.795738  87.517197    level_1          230   
    2     5000  2016-01-03 00:56:59  41.795738  87.517197    level_2          230   
    3     5000  2016-01-03 01:01:59  41.795738  87.517197    level_3          230   
    4     5000  2016-01-03 01:13:59  41.795738  87.517197    level_4          230   
    5     5000  2016-01-03 01:28:59  41.795738  87.517197    level_5          230   
    6     5000  2016-01-03 01:35:59  41.795738  87.517197    level_6          230   
    
       event_value  
    0          0.0  
    1          0.0  
    2          0.0  
    3          0.0  
    4          0.0  
    5          0.0  
    6          0.0  


That worked out nicely. Since we didn't care about event_values per user, `groupby` wasn't necessary. If we were to run this using `transform`, we'd get an error. Since it is run column-by-column, there isn't a practical way to reference other columns like with `apply`.

In the next post of the series, we'll continue using pandas to answer more interesting product questions like:
 
* How much time does it take our users to purchase after downloading the app?
* How many logins does it take our users for in-app purchases?
* What is the lifetime value of our users?
