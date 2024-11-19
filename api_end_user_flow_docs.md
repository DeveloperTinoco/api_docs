# FTR Intel - API Testing Documentation

## Current available datasets

* IU
* RU
* SU
* TU
* TTO

## Endpoints

There are currently 7 different endpoints for testing in our v1 API.

* __POST__ - /api/v1/token
* __GET__ - /api/v1/standard-products/monthly-data
* __GET__ - /api/v1/standard-products/quarterly-data
* __GET__ - /api/v1/standard-products/annual-data
* __GET__ - /api/v1/standard-products/graphs/<span style="color: #e30b5d;">{dataset}</span>/<span style="color: #e30b5d;">{date}</span>
* __GET__ - /api/v1/standard-products/tables/<span style="color: #e30b5d;">{dataset}</span>/<span style="color: #e30b5d;">{date}</span>
* __GET__ - /api/v1/standard-products/database/<span style="color: #e30b5d;">{dataset}</span>/<span style="color: #e30b5d;">{date}</span>

#

### /api/v1/token - __POST__

This <span style="color: #e30b5d;">__/token__</span> endpoint returns a JWT bearer token to the user that needs to be used in every request moving forward so that the user can remain authenticated for the given timeframe that we allocate to each token. After the given timeframe runs out (1 hour), the token expires and a new token has to be requested.

The following __Python__ code block will provide an example of how to ping the endpoint to retrieve a JWT bearer token for authorization.

```python
from requests_html import HTMLSession
import json

# User Credentials
USERNAME = "your_username"
PASSWORD = "your_password"

# Create session
session = HTMLSession()

# Endpoint to retrieve Token with headers
get_token_url = "https://h1wh682ob0.execute-api.us-east-1.amazonaws.com/api/v1/token"
headers = {
    "accept": "application/json",
    "Content-Type": "application/x-www-form-urlencoded",
}

# Payload that holds credentials
payload = f"grant_type=password&username={USERNAME}&password={PASSWORD}"

# Send the POST request with the headers & the payload to receive the response
response = session.post(get_token_url, headers=headers, data=payload)
```

A <span style="color: #e30b5d;">__successful__</span> attempt will provide the user with the following JSON data in the <span style="color: #e30b5d;">__response.json()__</span>:

```python
{
    "access_token": "unique_authenticated_token_string",
    "token_type": "bearer"
}
```

An <span style="color: #e30b5d;">__unsuccessful__</span> attempt would return the following message to the user:

```python
{
    "detail": "Incorrect username or password."
}
```

Once a user is properly authenticated and they receive a JWT bearer token, they must use that token in ongoing requests until that token expires.

It is best practice to store the JWT token once you request it and implement logic to re-use that JWT token. Once the token expires, which you will know when that is given a specific error message, you should then request a new one.

#

### /api/v1/standard-products/monthly-data - __GET__

Continuing from the code above, the following code block is how to retrieve the bearer token, setup following requests & ping the <span style="color: #e30b5d;">__/monthly-data__</span> endpoint:

```python
# Pull the JWT bearer authentication token from the response
bearer_token = response.json()["access_token"]

# New headers that contains our authorization token & encoding setting
updated_headers = {
    "accept": "application/json",
    "Authorization": f"Bearer {bearer_token}",
    "Accept-Encoding": "gzip"
}

# Query the datasets we want to pull the data for
# Note that the 'datasets' parameter accepts a Python list of strings
querystring = {'datasets': ['tto', 'tu', 'iu']}

# Send a GET request to the monthly-data endpoint with the headers and querystring
response_two = session.get(url="https://h1wh682ob0.execute-api.us-east-1.amazonaws.com/api/v1/standard-products/monthly-data", headers=updated_headers, params=querystring)

# If the response is successful, decode/load the JSON data otherwise check the error
if response_two.status_code = 200:
    monthly_data_json = json.loads(response_two.content.decode('utf-8'))
    # Continue logic here to handle data as needed
else:
    print(resposne_two.json())
```

With an active JWT token, the user above is pinging the <span style="color: #e30b5d;">__/monthly-data__</span> endpoint for the specific datasets that are listed in the <span style="color: #e30b5d;">__querystring__</span> variable.

If the request is successful, the data that was requested would be returned to the user in a compressed byte-like format. The user must handle this by ensuring the <span style="color: #e30b5d;">__"Accept-Encoding": "gzip"__</span> header is present in their request and they are decoding/loading the byte-like object appropriately as shown above.

The following JSON output is an example of what the raw data will look like when returned to the user.

```python
{
    "tto": [
        {
            "date": "2000-01-01",
            "data_series": "U.S. Freight Tonnage (millions)",
            "data_label": "Total U.S. Tonnage (SA)",
            "source": "FTR",
            "history_forecast_status": "History",
            "value": 1570.0
        },
        ...
    ]
    ...
```

The image below is what the original file looks like and where the raw data is coming from.

![Test Image](https://i.postimg.cc/Jz7fmQRx/Image-example.png)

- <span style="color: #969696;"> __date__ </span>
- <span style="color: #ff2e31;"> __data_series__ </span>
- <span style="color: #4dff4b;"> __data_label__ </span>
- <span style="color: #feff14;"> __history_forecast_status__ </span>
- <span style="color: #2d2cfc;"> __source__ </span>
- <span style="color: #fe14ff;"> __value__ </span>

The color coded break down above reflects what each item in our raw response represents and how our raw data is structured.

Although the current user in our example above is authenticated by our system, this user may not be subscribed to a product/service that offers one of the datasets they requested.

In the example above, the user is requesting the TTO, TU & IU monthly data. If the user is not subscribed to a product/service that provides the TTO dataset, they will only receive the TU and IU monthly data.

The process above is the same for the <span style="color: #e30b5d;">__/annual-data__</span> and the <span style="color: #e30b5d;">__/quarterly-data__</span> endpoints.

#

### /api/v1/standard-products/graphs/{dataset}/{date} - __GET__

The <span style="color: #e30b5d;">__/graphs/{dataset}/{date}__</span> endpoint allows an authenticated user to download a zip file that contains two files. The files are the .pptx and the .pdf versions of the monthly graphs for each respective dataset. The endpoint takes one dataset and one date at a time. 

The following code block is continued from the code block above for the <span style="color: #e30b5d;">__/token__</span> endpoint and provides an example of how to ping this endpoint to download the zip file. 

```python
# Pull the JWT bearer authentication token from the response
bearer_token = response.json()["access_token"]

# New headers that contain our authorization token
updated_headers = {
    "accept": "application/json",
    "Authorization": f"Bearer {bearer_token}"
}

# Send a GET request to the /graphs endpoint using the dataset and date you want the zip file for
response_two = session.get(url="https://h1wh682ob0.execute-api.us-east-1.amazonaws.com/api/v1/graphs/tto/2024-09", headers=updated_headers)

# If the response is successful download the file, otherwise check the error
if response_two.status_code == 200:
    with open(f'This will be your zip filename.zip', 'wb') as f:
        f.write(response_two.content)
else:
    print(response_two.json())
```

A successful ping to that endpoint used in the code block above will download the zip file as "This will be your zip filename.zip" in the same directory that the script is being executed from. The same authentication logic applies here. If the user does not have access to the TTO dataset, they can still ping the endpoint however their request will fail.

Please note that the date in the endpoint in the code block above uses a <span style="color: #e30b5d;">__YYYY-MM__</span> format. Ensure you are using this format when pinging these endpoints to successfully interact with our API.

The <span style="color: #e30b5d;">__/database__</span> and the <span style="color: #e30b5d;">__/tables__</span> endpoints work exactly the same as the <span style="color: #e30b5d;">__/graphs__</span> endpoint, the only change you will need to make is to your filename structure when downloading the file to ensure the file is downloaded appropriately. Both the <span style="color: #e30b5d;">__/database__</span> and the <span style="color: #e30b5d;">__/tables__</span> endpoints return Excel files so your file will need to be saved with the <span style="color: #e30b5d;">__.xlsx__</span> file extension as shown below.

```python
# Pull the JWT bearer authentication token from the response
bearer_token = response.json()["access_token"]

# New headers that contain our authorization token
updated_headers = {
    "accept": "application/json",
    "Authorization": f"Bearer {bearer_token}"
}

# Send a GET request to the /tables or /database endpoint using the dataset and date you want the file for
response_two = session.get(url="https://h1wh682ob0.execute-api.us-east-1.amazonaws.com/api/v1/tables/tto/2024-09", headers=updated_headers)

# If the response is successful, download the file otherwise check the error
if response_two.status_code == 200:
    with open(f'This will be your excel filename.xlsx', 'wb') as f:
        f.write(response_two.content)
else:
    print(response_two.json())
```