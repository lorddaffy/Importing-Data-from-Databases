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
