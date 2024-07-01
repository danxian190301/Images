# Perform NL-to-SQL using Dataherald AI with PostgreSQL （Email_Compagin5.0)

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
