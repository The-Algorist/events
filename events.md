
---

# Structuring InfluxDB Schema to Record Customer Journey Events

To structure your InfluxDB schema for recording customer journey events, you need to organize the data efficiently to store and query time-series data. Here’s a detailed approach to setting up the schema:

## Measurement

A measurement in InfluxDB is similar to a table in a relational database. For user events, you can name your measurement `events`.

## Tags

Tags in InfluxDB are indexed, making them useful for filtering and grouping data. Relevant tags for tracking user events include:

- `event_type`: Type of event (e.g., 'pageview', 'interaction', 'transaction', 'button_click').
- `content_id`: Unique identifier for the content (e.g., movie ID).
- `user_id`: Unique identifier for the user.
- `device_type`: Type of device used (e.g., 'desktop', 'mobile').
- `location`: User's location (optional).
- `referral_source`: Source of the referral traffic (optional).

## Fields

Fields are not indexed but store the actual data values. For user events, fields include:

- `timestamp`: Timestamp of the event.
- `page_url`: URL of the page (for pageview events).
- `scroll_depth`: Depth of scrolling (for scroll depth events).
- `interaction_type`: Type of interaction (e.g., 'click', 'search') (for interaction events).
- `purchase_value`: Value of the purchase (for purchase events).
- `rental_duration`: Duration of the rental (for rental events).
- `rental_popularity`: Popularity score of the rented movie (for rental events).
- `time_lag_from_transaction`: Time lag from the transaction (for first rental view events).
- `button_clicked`: Type of button clicked (e.g., 'add_to_collection', 'set_reminder', 'share') (for button click events).

## Example Schema

Here’s an example of how you might structure the data in InfluxDB:

### Measurement: `events`

- **Tags:**
  - `event_type` (string)
  - `content_id` (string)
  - `user_id` (string)
  - `device_type` (string)
  - `location` (string, optional)
  - `referral_source` (string, optional)
  
- **Fields:**
  - `timestamp` (timestamp)
  - `page_url` (string, optional)
  - `scroll_depth` (float, optional)
  - `interaction_type` (string, optional)
  - `purchase_value` (float, optional)
  - `rental_duration` (integer, optional)
  - `rental_popularity` (integer, optional)
  - `time_lag_from_transaction` (integer, optional)
  - `button_clicked` (string, optional)

## Example Data Points

### Example 1: Pageview Event

```plaintext
events,event_type=pageview,content_id=12345,user_id=67890,device_type=desktop,page_url="https://example.com/movie/12345" timestamp=1622499200
```

### Example 2: Purchase Event

```plaintext
events,event_type=transaction,content_id=12345,user_id=67890,device_type=mobile,purchase_value=19.99 timestamp=1622499200
```

## Implementation in Code

To implement this schema, you can use a Python script with the InfluxDB client library to send data points to your InfluxDB instance. Here’s how:

### Python Code Example

```python
from influxdb import InfluxDBClient
from datetime import datetime

# Initialize InfluxDB client
client = InfluxDBClient(host='localhost', port=8086)
client.switch_database('my_database')

def write_event(event_type, content_id, user_id, device_type, timestamp, page_url=None, scroll_depth=None, interaction_type=None, purchase_value=None, rental_duration=None, rental_popularity=None, time_lag_from_transaction=None, button_clicked=None, location=None, referral_source=None):
    json_body = [
        {
            "measurement": "events",
            "tags": {
                "event_type": event_type,
                "content_id": content_id,
                "user_id": user_id,
                "device_type": device_type,
                "location": location,
                "referral_source": referral_source
            },
            "fields": {
                "timestamp": timestamp,
                "page_url": page_url,
                "scroll_depth": scroll_depth,
                "interaction_type": interaction_type,
                "purchase_value": purchase_value,
                "rental_duration": rental_duration,
                "rental_popularity": rental_popularity,
                "time_lag_from_transaction": time_lag_from_transaction,
                "button_clicked": button_clicked
            },
            "time": datetime.utcnow().isoformat()
        }
    ]
    
    client.write_points(json_body)

# Example usage
write_event(
    event_type="transaction",
    content_id="12345",
    user_id="67890",
    device_type="mobile",
    timestamp=1622499200,
    purchase_value=19.99
)
```

## Using Cloudflare Workers to Send Data to InfluxDB

To send data to InfluxDB using Cloudflare Workers, you can set up an HTTP endpoint that processes incoming requests and forwards them to InfluxDB.

### Cloudflare Worker Script Example

```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  if (request.method !== 'POST') {
    return new Response('Method Not Allowed', { status: 405 })
  }

  const data = await request.json()
  const event_type = data.event_type
  const content_id = data.content_id
  const user_id = data.user_id
  const device_type = data.device_type
  const timestamp = data.timestamp
  const page_url = data.page_url || null
  const scroll_depth = data.scroll_depth || null
  const interaction_type = data.interaction_type || null
  const purchase_value = data.purchase_value || null
  const rental_duration = data.rental_duration || null
  const rental_popularity = data.rental_popularity || null
  const time_lag_from_transaction = data.time_lag_from_transaction || null
  const button_clicked = data.button_clicked || null
  const location = data.location || null
  const referral_source = data.referral_source || null

  let line = `events,event_type=${event_type},content_id=${content_id},user_id=${user_id},device_type=${device_type}`
  if (location) {
    line += `,location=${location}`
  }
  if (referral_source) {
    line += `,referral_source=${referral_source}`
  }
  line += ` timestamp=${timestamp}`
  if (page_url !== null) {
    line += `,page_url="${page_url}"`
  }
  if (scroll_depth !== null) {
    line += `,scroll_depth=${scroll_depth}`
  }
  if (interaction_type !== null) {
    line += `,interaction_type="${interaction_type}"`
  }
  if (purchase_value !== null) {
    line += `,purchase_value=${purchase_value}`
  }
  if (rental_duration !== null) {
    line += `,rental_duration=${rental_duration}`
  }
  if (rental_popularity !== null) {
    line += `,rental_popularity=${rental_popularity}`
  }
  if (time_lag_from_transaction !== null) {
    line += `,time_lag_from_transaction=${time_lag_from_transaction}`
  }
  if (button_clicked !== null) {
    line += `,button_clicked="${button_clicked}"`
  }

  const influxDBUrl = 'https://your-influxdb-instance.com/api/v2/write?org=your-org&bucket=your-bucket&precision=s'
  const influxDBToken = 'your-influxdb-auth-token'

  const influxResponse = await fetch(influxDBUrl, {
    method: 'POST',
    headers: {
      'Authorization': `Token ${influxDBToken}`,
      'Content-Type': 'text/plain'
    },
    body: line
  })

  if (influxResponse.ok) {
    return new Response('Data written to InfluxDB', { status: 200 })
  } else {
    const errorText = await influxResponse.text()
    return new Response(`Error writing to InfluxDB: ${errorText}`, { status: influxResponse.status })
  }
}
```

### Deploy the Worker

Deploy the Cloudflare Worker through the Cloudflare dashboard or using the `wrangler` CLI.

## Summary

1. **InfluxDB Schema**: Organize data with a measurement (`events`), tags (`event_type`, `content_id`, `user_id`, `device_type`, `location`, `referral_source`), and fields (`timestamp`, `page_url`, `scroll_depth`, `interaction_type`, `purchase_value`, `rental_duration`, `rental_popularity`, `time_lag_from_transaction`, `button_clicked`).
2. **Python Script**: Use a Python script to send data to InfluxDB.
3. **Cloudflare Worker**: Set up a Cloudflare Worker to forward data to InfluxDB via HTTP POST.

This setup ensures that you can track user events efficiently and store them in a time-series database for detailed analysis.

## Example Data Table

Below is a example showing how the data might look in the `events` measurement:

```html
<!DOCTYPE html>
<html>
<head>
    <title>InfluxDB Events Data</title>
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
        }
        th {
            background-color: #f2f2f2;
            text-align: left;
        }
    </style>
</head>
<body>

<h2>InfluxDB Events Data</h2>

<table>
    <tr>
        <th>time</th>
        <th>event_type</th>
        <th>content_id</th>
        <th>user_id</th>
        <th>device_type</th>
        <th>location</th>
        <th>referral_source</th>
        <th>timestamp</th>
        <th>page_url</th>
        <th>scroll_depth</th>
        <th>interaction_type</th>
        <th>purchase_value</th>
        <th>rental_duration</th>
        <th>rental_popularity</th>
        <th>time_lag_from_transaction</th>
        <th>button_clicked</th>
    </tr>
    <tr>
        <td>2023-05-17T00:00:00Z</td>
        <td>pageview</td>
        <td>12345</td>
        <td>67890</td>
        <td>desktop</td>
        <td></td>
        <td></td>
        <td>1622499200</td>
        <td>https://example.com/movie/12345</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>2023-05-17T00:05:00Z</td>
        <td>transaction</td>
        <td>12345</td>
        <td>67890</td>
        <td>mobile</td>
        <td></td>
        <td></td>
        <td>1622499200</td>
        <td></td>
        <td></td>
        <td></td>
        <td>19.99</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>2023-05-17T00:10:00Z</td>
        <td>interaction</td>
        <td>12345</td>
        <td>67890</td>
        <td>desktop</td>
        <td>New York</td>
        <td>Google</td>
        <td>1622499300</td>
        <td>https://example.com/search</td>
        <td></td>
        <td>search</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>2023-05-17T00:15:00Z</td>
        <td>button_click</td>
        <td>12345</td>
        <td>67890</td>
        <td>mobile</td>
        <td>San Diego</td>
        <td></td>
        <td>1622499400</td>
        <td>https://example.com</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>add_to_collection</td>
    </tr>
</table>

</body>
</html>


```


This table represents a mix of different events recorded with their respective tags and fields, allowing for detailed tracking and analysis of user behavior.

---
