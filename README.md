# Parking Dashboard 

## 🚀 Content

- (✅) Data [dat](#📊-data) acquisition from the application
- (✅) [Algorithm](#algorithm) for available spots
- (✅) Parking lot [database](#📦-database)
- (✅) [Web output](#🖥-website) of the dashboard

# 📊 Data


The AXIS camera is running an application for car classification. This application returns data in the JSON format. This JSON contains information about individual parking spots. The main value is the state which is true if the spot is occupied and false if the spot is available.

```json
{
  "01": {
    "id": "01",
    "state": "true"
  }
}
```

You can simply get the data from the application using a simple [fetch](https://javascript.info/fetch). However, it is necessary to verify the identity using Digest authentication. For this, the special [digest-fetch](https://www.npmjs.com/package/digest-fetch) package is used.

```js
const digestFetch = require("digest-fetch");
const digclient = new digestFetch(username, password, { algorithm: "MD5" });

await digclient.fetch(url)
    .then ...
```

# Algorithm

To obtain the value of how many spots are occupied, it is necessary to create an algorithm. Simply loop through each object and determine if its `state` variable is equal to `true`. If so, add +1. The result is then saved in a parking lot array.

```js
for (const key in json) {
  if (json[key].state) {
    places[x]++;
  }
}
```

The maximum number of spots can be easily obtained by the length of the JSON object.

```js
max[x] = Object.keys(json).length;
```

But it is also necessary to determine how many cars park on the entire parking lot each day. Therefore, we save our `Current` value of occupied spots into a variable, which we then compare with the new `Calculated` value from the previous algorithm. If the `Current` value is less than the `Calculated` value, the difference between `Current` and `Calculated` is added to the daily count. Every day exactly at 23:59:59, this value is sent to the database and reset.

```js
if(Current < Calculated) {
  daily[x] += Calculated - Current;
}
```

# 📦 Database

The end-of-day values are stored in a database created using [MongoDB](https://www.mongodb.com/) and hosted on [Atlas](https://www.mongodb.com/cloud/atlas), as mentioned at the end of the [Algorithm](#algorithm) section. The [Mongoose](https://mongoosejs.com/) library is used to simplify and enhance the visibility of the database operations. The schema stores the current date and the values for each parking lot.

## Getting data from database

The date sent to MongoDB must be in a specific format. Therefore, it needs to be modified before sending. The formatDate function is used for this, which converts the date to the format YYYY-MM-DD. If the month or day is single-digit, a 0 is added before it (padTo2Digits function). The modified date can then be sent.

When retrieving data from the database for display in the statistics on the webpage, the format is unreadable because Mongo adds the `time` 00-00-00, the `time zone`, etc. So, it is necessary to adjust the format to be readable. The formatDate function is used again (everything is separated by "-"), but to have the first year, the date needs to be rotated and concatenated with dots. Similarly, the data of each parking lot needed to be adjusted.

# 🖥 Website

The [Express](https://expressjs.com/) framework is used to display the data on the page. It is a very simple framework that allows you to create endpoints that return different data, in this case HTML structure. Simply return the values from the array at the specified endpoint index. For example, Parking Lot 04 returns all data from the arrays at index 4. The page is refreshed every second and a half. [Font Awesome](https://fontawesome.com/) is used to spice up the page for icons.

On the left side, there is a list of all parking lots, next to which is a number representing the `maximum` number of parking spaces. All values are `dynamic`, so in case of adding a new parking space to any parking lot, the maximum value is automatically recalculated.
