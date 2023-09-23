# sqlite_database_operations
This is a repository for Week 3 of HHA 504

# Data Exploration and Analysis
First, I loaded in the given data for Stony Brook's price per code. I performed some exploratory actions like looking for missing data with df.isnull(). 
I chose not to remove any rows with missing values because it does not make sense to do so.
After which, I removed white spacces and special characters using 
```
import re
def cleaning(df):
    def clean_name(name):
        cleaned_name = re.sub(r'[^a-zA-Z0-9]', '', name)
        return cleaned_name.lower()

    df.columns = [clean_name(col) for col in df.columns]
    return df

df = cleaning(df)
df.sample(5)
```
Next, I generated descriptive statistics rounded to two decimal spaces because we're dealing with money. For this, we needed to import numpy as np. 
```
df.describe().round(2)
```
I also generated value counts for two categorical data: packagelinelevel and type. 
I had issues with dealing with the second dataset. 

# SQLite Database Operations
We are dealing with sqlalchemy and sqlite3, so we imported both of those in. 
```
from sqlalchemy import create_engine
import sqlite3
```
Next, we create a temporary database file using SQLITE in our local machine.
```
conn = sqlite3.connect('health.db')
c = conn.cursor()
```
Then, I created a table called healthinfo within my database file.
```
c.execute('''
              CREATE TABLE healthinfo
                      (
                        bloodpressure integer,
                        heartreate integer,
                        height_inches real,
                        name text,
                        address text,
                        contact text
                        );
            ''')
conn.commit()
```
I used three different catagories. Integers are numbers. Real are float values. Text are string values. 
To insert data into the table, we use the following. 
```
sql_query = '''
INSERT INTO healthinfo (
  'bloodpressure',
  'heartreate',
  'height_inches',
  'name',
  'address',
  'contact'
  )
  values (
    90,
    85,
    57.25,
    'Jane Doe',
    '68 McDonalds Rd',
    'janedoe@gmail.com'
  );
'''
```
The values correspond with the new rows above. For example, 90 corresponds with bloodpressure. 
Now we can check if we inserted the data with pandas. 
```
pd.read_sql_query("select * from healthinfo;", conn)
```
We then added another row for practice. 
#Automatic Table Creation
First, we modified the Stonybrook CSV file into something we can work with. 
```
columnNames = list(df)
idVars = columnNames[:8]
valueVars = columnNames[8:]

stonybrook_modified = df.melt(id_vars=idVars, value_vars=valueVars)

stonybrook_modified.columns

stonybrook_modified.rename(columns={'HospCode':'hospital_name',
            'variable':'insurance_type',
            'Code':'code',
            'Description':'code_description',
             'value':'cost_negotiated',
             'Minimum Negotiated Charge':'cost_minimum',
             'Maximum Negotiated Charge': 'cost_maximum'}, inplace=True)

stonybrook_modified['insurance_type'].value_counts()

stonybrook_modified.sample(10)
```
Then, we inserted the entire table into our healthinfo table within the database. 
```
stonybrook_modified.to_sql('healthinfo', conn, if_exists='replace')
```
The if_exists='replace' code simply replaces the data within the healthinfo table. 
# Diving Deeper
We can then create a query to look for specific information. For example, I set my query to look for the CPT code of 92950, which translates to cardiac arrest.
```
query = '''
  select *
  from healthinfo
  where code = '92950'
'''

output = pd.read_sql(query, conn)
output
```
Next, I wanted to look at the pricing for each cardiac arrest instance for each insurance. I created a pivot table that took the average price of the cardiac arrest code for each insurance. 
```
pivot_data = output.pivot_table(index='insurance_type', columns='code', values='cost_negotiated', aggfunc='mean')
pivot_data
```
