# Perform NL-to-SQL using Dataherald AI with PostgreSQL （Email_Compagin5.0)
<details>
  <summary>Tables of Contents</summary>
  
  1. Overview
  2. Use the Constant Contact API to extract the required data
  3. Load the data into PostgreSQL
  4. NL-to-SQL with Dataherald AI and PostgreSQL

</details>

## Overview
This project was done by grabbing data through the Constantcontact API and then collating the data afterward. Connect to PostgreSQL using the Dataherald API and execute NL-to-SQL.

## Use the Constant Contact API to extract the required data（Email_Compagin5.0)
- Extract data using API.
- Set the URL.
- Set the authorization token. replace 'your_token' with the actual token.
- Send the get request.
- Raise an exception in case the get request was unsuccessful.
- Load the response as a JSON
- Convert the JSON data into a pandas data frame.
- Save the data frame to a CSV file.
- Confirm the operation.

The following screenshot shows the code used, and the results of running it.
```python
import requests
import pandas as pd
import json
```
The above shows the Python libraries to be used for this example.
```python
url = 'https://api.constantcontact.com/v2/emailmarketing/campaigns/1140300363415?api_key=mqn6hbrtymaa6a9c756xgaww'
headers = {'Authorization': 'Bearer 7ae092c5-6217-4a20-acba-f804f35ec6cb'}
response = requests.get(url, headers=headers)
response.raise_for_status()
data = response.json()
df = pd.json_normalize(data)
df.to_csv('email_compaign5.0.csv', index=False)
print("Data exported to email_compaign5.0.csv")
print(df.head())
```
The following is the result of the code run, which obtained the required data.
![screenshot](https://github.com/DanXian01/Images-1/blob/patch-1/Email_Compaign1.png)

## Load the data into PostgreSQL
This is the preparation step in the project, we need to import the obtained data into PostgreSQL, you can prepare a new schema in advance for the data import.

- Prepare the Python libraries.
```python
!pip install psycopg2-binary
import csv
import psycopg2
```
- Setup database connection.
```python
conn = psycopg2.connect(host='73.222.84.110', dbname='Dan', user='postgres', password='postgres')
cur = conn.cursor()
```
- Create the column and the SQL query without specifying column types in the INSERT statement.
```python
sql_query = """
INSERT INTO V_DS_ESL_AGG_SUMMARY (
    id,
    name,
    subject,
    status,
    from_name,
    from_email,
    reply_to_email,
    template_type,
    created_date,
    modified_date,
    last_run_date,
    permalink_url,
    is_permission_reminder_enabled,
    permission_reminder_text,
    is_view_as_webpage_enabled,
    view_as_web_page_text,
    view_as_web_page_link_text,
    greeting_salutations,
    greeting_name,
    greeting_string,
    sent_to_contact_lists,
    click_through_details,
    message_footer.city,
    message_footer.state,
    message_footer.country,
    message_footer.organization_name,
    message_footer.address_line_1,
    message_footer.address_line_2,
    message_footer.address_line_3,
    message_footer.international_state,
    message_footer.postal_code,
    message_footer.include_forward_email,
    message_footer.include_subscribe_link,
    tracking_summary.sends,
    tracking_summary.opens,
    tracking_summary.clicks,
    tracking_summary.forwards,
    tracking_summary.unsubscribes,
    tracking_summary.bounces,
    tracking_summary.spam_count
    # Include DUMMY_DATA if it's supposed to receive a value
) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
"""
```
- Open the CSV file and import the data into the table.
```python
with open('/content/email_compaign5.0.csv', 'r') as f:
    reader = csv.reader(f)
    next(reader)  # Skip the header row
    for row in reader:
        if len(row) == 9:  # Ensure there are exactly 9 columns as expected
            try:
                cur.execute(sql_query, row)
            except psycopg2.Error as e:
                print(f"Error inserting data: {e}, row: {row}")
                conn.rollback()
            else:
                conn.commit()
        else:
            print(f"Skipped malformed row: {row}")
```
## NL-to-SQL with Dataherald AI and PostgreSQL
Having completed the data preparation described above, we can next use obtaining the data for our purposes. To use DataheraldAI, you first need to connect DataheraldAI to PostgreSQL via an API. you can then use Dataherald to allow the application to ask questions in natural language, thus removing the technical barriers to getting insights from the data.

Step 1: Install and Instantiate Dataherald
- You can interact with the Dataherald API through HTTP requests from any language or through our official Python SDK package. To install the official Python SDK package, run the following command:
```python
!pip install dataherald
```
- Authentication

The Dataherald API uses API keys for authentication. These API keys are at the organizational level. You can access the API Keys page in the Management Console and retrieve keys that can be used to make API calls. You should store API keys securely and not expose them to client code. All API requests should include your API key in an X-API-Key HTTP header as follows:
```python
'X-API-Key: dh-2d3b40……………………………………………………………………60af'
```
For example, to make a call to get a list of database connections for your organization the call will be as follows:
```python
!curl -X 'GET' \
  'https://stage.api.dataherald.ai/api/database-connections' \
  -H 'accept: application/json' \
  -H 'X-API-Key: dh-a8fd5ab7d44ad6439ee2fec40fbe7b4e086134ff05578aff6eca7b1edfcf898e'
```

- The API key is passed as a keyword argument to the Dataherald Client object.
```python
dh_client = Dataherald(base_url = 'https://api.dataherald.ai',
                       api_key='dh-2d3b40……………………………………………………………………60af' ,
                       timeout=360)
```
- Assumes the API key is saved as an environment variable.
```python
import os
from dataherald import Dataherald
API_KEY = os.environ['API_KEY']
dh_client = Dataherald(api_key=API_KEY)
```
Step 2: Create a Database Connection

The next step is to give the tool access to your relational data warehouse. Dataherald uses the standard ODBC protocol to connect to the relational data warehouse of your choice. To create a database connection, you need a connection URI for the database.

- Establish a Connection to PostgreSQL

You’ll need to generate a connection URI for your data warehouse instance, which can be done either from the admin console’s Databases tab or through the SDK. Otherwise, you can use the/table-descriptions POST endpoint.
```python
alias = "PostgreSQL_RealEstate"
use_ssh = False
connection_uri = "postgresql+psycopg2://<user>:<password>@<host>:<port>/<db-name>" 

db_connection = dh_client.database_connections.create(
  alias=alias,
  use_ssh=use_ssh,
  connection_uri=connection_uri
)
```
As an example of the API on display, here is a complete process for connecting DataheraldAI to PostgreSQL.
```python
from dataherald import Dataherald
# Create a Dataherald client
dh_client = Dataherald(api_key='dh-23195cc8c49adb0b9b2f09ccb49824ee2968308e28f7cfcd42a2d4354601a0fd')
alias = "<Email_Compaign>"
use_ssh = False # Dataherald provides support for SSH connections
connection_uri = "postgresql+psycopg2://postgres:postgres@34.121.125.111/Dan"
db_connection = dh_client.database_connections.create(
  alias=alias,
  use_ssh=use_ssh,
  connection_uri=connection_uri
)
```
Step 3: Scan Tables

Next, Dataherald AI will now scan all of the specified tables, performing tasks to better understand how to use the data. The tool will:
- Read the schema to infer data types and the type of content contained in the table.
- Extract identified categorical variable values.
- Determine relationships between tables.

Use the /table-descriptions/sync-schemas POST endpoint to start the scan. You may specify tables in the endpoint, or just have the tool scan all available tables. The SDK code would look like this:
```python
db_connection_id = db_connection.id
table_names = [ "table_name" ]

dh_client.table_descriptions.sync_schemas(
  db_connection_id=db_connection_id,
  table_names=table_names
)
```
The form was successfully scanned and on DataheraldAI's operating platform, you can see that "scanned" is displayed. This is shown in the figure below.

![shootscreen](https://github.com/DanXian01/Images-1/blob/patch-1/Email_compaign2.png)

Step 4: Querying your Data in Natural Language

DHAI can now be leveraged to answer questions. The tool can generate the SQL to answer the question, extract the relevant slice of data, and then convert the response back to natural language for the user. It utilizes the following API endpoints:

- /prompts creates a prompt, without trying to generate any SQL code or natural language response
- /prompts/sql-generations creates the prompt and generates a valid SQL query in response
- /prompts/sql-generations/nl-generations creates the prompt, generates a valid SQL query, and generates a natural language response from the results of the query
The below SDK code makes use of the above endpoints:

```python
sql_generation = {
  "prompt": {
    "text": "The content of questions",
    "db_connection_id": db_connection.id
  }
}

response = dh_client.nl_generations.create(sql_generation=sql_generation)
nl_generation_id = response.id
sql_generation_id = response.sql_generation_id
prompt_id = dh_client.sql_generations.retrieve(id=sql_generation_id).prompt_id

print(f"Question: {dh_client.prompts.retrieve(id=prompt_id).text}")
print(f"NL response: {dh_client.nl_generations.retrieve(id=nl_generation_id).text}")
print()
print(f"Generated SQL query: \n{dh_client.sql_generations.retrieve(id=sql_generation_id).sql}")
```
Example: Querying the Data in Natural Language as an example of the data used in this project and show the results.
```python
from dataherald.types.sql_generation_create_params import Prompt
prompt = Prompt(text="<That subject has the highest open count and sorts the different subjects according to the value of the open count.>",
db_connection_id="665e44b26160e4adb2d8bbac")
response = dh_client.sql_generations.create(prompt=prompt)
print(response.sql)
```
The following screenshot shows the results of the above code run.

![shootscreen](https://github.com/DanXian01/Images-1/blob/patch-1/E3.png)
```python
# In the 14th cell, replace 'db_connection.id' with 'db_connection_id'
sql_generation = {
  "prompt": {
    "text": "That subject has the highest open count and sorts the different subjects according to the value of the open count.",
    "db_connection_id": db_connection_id
  }
}

response = dh_client.nl_generations.create(sql_generation=sql_generation)
nl_generation_id = response.id
sql_generation_id = response.sql_generation_id
prompt_id = dh_client.sql_generations.retrieve(id=sql_generation_id).prompt_id

print(f"Question: {dh_client.prompts.retrieve(id=prompt_id).text}")
print(f"NL response: {dh_client.nl_generations.retrieve(id=nl_generation_id).text}")
print()
print(f"Generated SQL query: \n{dh_client.sql_generations.retrieve(id=sql_generation_id).sql}")
```
The following screenshot shows the results of the above code run.

![shootscreen](https://github.com/DanXian01/Images-1/blob/patch-1/E4.png)

The results from the presentation show that it consists of two parts. The first part is the answer to the question, which answers the question posed and gives an explanation. The second part is the Generated SQL Query, which can be copied directly for use in the query tool in PostgreSQL. Also, the console in DataheraldAI gives the results simultaneously, as shown in the figure below.

![shootscreen](https://github.com/DanXian01/Images-1/blob/patch-1/E5.png)
With Dataherald, you can eliminate the technical barriers to deriving insights from your data by enabling applications to ask questions in natural language.
