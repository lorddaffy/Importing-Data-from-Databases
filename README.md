# Importing-Data-from-Databases
*In this repo, I'm practicing on Pandas to connect the Databases, I will work with SQL lite*

__________________________________

> Create a database engine to manage connections to a database, data.db
```
# Import sqlalchemy's create_engine() function
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine("sqlite:///data.db")

# View the tables in the database
print(engine.table_names())
```
__________________________________

> You saw that data.db has two tables. weather has historical weather data for New York City. hpd311calls is a subset of call records made to the city's 311 help line about housing issues.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Load hpd311calls without any SQL
hpd_calls = pd.read_sql('hpd311calls', engine)

# View the first few rows of data
print(hpd_calls.head())

# Create a SQL query to load the entire weather table
query = 'SELECT * FROM weather;'

# Load weather with the SQL query
weather = pd.read_sql(query, engine)

# View the first few rows of data
print(weather.head())
```
__________________________________

> write a query to SELECT only the date and temperature columns to make a dataframe of high and low temperature readings.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create database engine for data.db
engine = create_engine('sqlite:///data.db')

# Write query to get date, tmax, and tmin from weather
query ='SELECT date ,tmax, tmin FROM weather;'

# Make a dataframe by passing query and engine to read_sql()
temperatures = pd.read_sql(query,engine)

# View the resulting dataframe
print(temperatures)
```
__________________________________

> The hpd311calls table in data.db has data on calls about various housing issues, from maintenance problems to information requests. use SQL to focus on calls about safety.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create database engine for data.db
engine = create_engine('sqlite:///data.db')

# Create query to get hpd311calls records about safety
query = 'SELECT * FROM hpd311calls Where complaint_type='SAFETY';'

# Query the database and assign result to safety_calls
safety_calls = pd.read_sql(query,engine)

# Graph the number of safety calls by borough
call_counts = safety_calls.groupby('borough').unique_key.count()
call_counts.plot.barh()
plt.show()
```
__________________________________

> The weather table contains daily high and low temperatures and precipitation amounts for New York City. Let's focus on inclement weather, where there was either an inch or more of snow or the high was at or below freezing (32° Fahrenheit). To do this, you'll need to build a query that look at values in both columns.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Create query for records with max temps <= 32 or snow >= 1
query = 'SELECT * FROM weather where tmax <=32 or snow >=1;'
# Query database and assign result to wintry_days
wintry_days = pd.read_sql(query,engine)

# View summary stats about the temperatures
print(wintry_days.describe())
```
__________________________________

> Since hpd311calls contains data about housing issues, we would expect most records to have a borough listed. Let's test this assumption by querying unique complaint_type/borough combinations.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Create query for unique combinations of borough and complaint_type
query = 'SELECT DISTINCT borough,complaint_type FROM hpd311calls;'

# Load results of query to a dataframe
issues_and_boros = pd.read_sql(query,engine)

# Check assumption about issues and boroughs
print(issues_and_boros)
```
__________________________________

> The hpd311calls table has a column, complaint_type, that categorizes call records by issue, such as heating or plumbing. In order to graph call volumes by issue, you'll write a SQL query that COUNTs records by complaint type.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Create query to get call counts by complaint_type
query = 'SELECT complaint_type, COUNT(*) FROM hpd311calls GROUP BY complaint_type;'

# Create dataframe of call counts by issue
calls_by_issue = pd.read_sql(query, engine)

# Graph the number of calls for each housing issue
calls_by_issue.plot.barh(x="complaint_type")
plt.show()
```
__________________________________

> The weather table contains daily readings for four months. In this exercise, you'll practice summarizing weather by month.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Create query to get temperature and precipitation by month
query = 'SELECT month,MAX(tmax), MIN(tmin),SUM(prcp) FROM weather GROUP BY month;'

# Get dataframe of monthly weather stats
weather_by_month = pd.read_sql(query, engine)

# View weather stats by month
print(weather_by_month)
```
__________________________________

> Complete the query to join weather to hpd311calls by their date and created_date columns, respectively.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Query to join weather to call records by date columns
query = 'SELECT * FROM hpd311calls JOIN weather ON hpd311calls.created_date = weather.date;'

# Create dataframe of joined tables
calls_with_weather = pd.read_sql(query,engine)

# View the dataframe to make sure all columns were joined
print(calls_with_weather.head())
```

__________________________________

> Weather exacerbates some housing problems more than others. Your task is to focus on water leak reports in hpd311calls and assemble a dataset that includes the day's precipitation levels from weather to see if there is any relationship between the two. You'll need to get the necessary weather column and filter rows.
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Query to get water leak calls and daily precipitation
query = """
SELECT hpd311calls.*, weather.prcp
  FROM hpd311calls
  JOIN weather
    ON hpd311calls.created_date = weather.date
  WHERE hpd311calls.complaint_type = 'WATER LEAK';"""

# Load query results into the leak_calls dataframe
leak_calls = pd.read_sql(query, engine)

# View the dataframe
print(leak_calls.head())
```

__________________________________

> In addition to the hpd311calls table, data.db has a weather table with daily high and low temperature readings for NYC. We want to get each day's count of heat/hot water calls with temperatures joined in. This can be done in one query, which we'll build in parts. In part one, we'll get just the data we want from hpd311calls. Then, in part two, we'll modify the query to join in weather data..
```
# Load libraries
import pandas as pd
from sqlalchemy import create_engine

# Create the database engine
engine = create_engine('sqlite:///data.db')

# Modify query to join tmax and tmin from weather by date
query = """
SELECT hpd311calls.created_date, 
	   COUNT(*), 
       weather.tmax,
       weather.tmin
  FROM hpd311calls 
       Join weather
       ON hpd311calls.created_date = weather.date
 WHERE hpd311calls.complaint_type = 'HEAT/HOT WATER' 
 GROUP BY hpd311calls.created_date;
 """

# Query database and save results as df
df = pd.read_sql(query, engine)

# View first 5 records
print(df.head())
```
