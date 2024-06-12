# Getting Started with power-of-api

power-of-api is an easy-to-use, lightweight JavaScript library that provides a simple and efficient way to perform CRUD (Create, Read, Update, Delete) operations on various data storage options, including file storage, MongoDB, and in-memory storage.

## Features

* Small JavaScript footprint with no internal dependencies
* Supports multiple data storage options:
	+ [File storage: Store and retrieve data from files](#in-file-storage)
	+ [MongoDB: Interact with a MongoDB database](#mongodb)
	+ [In-memory storage: Temporarily store data while the API is running, clearing when the API stops](#in-memory)
* Easy configuration to get started quickly
* Simple CRUD operations using curl commands
* [Authentication](Authentication-config.md)

## Getting Started

To start using power-of-api, simply install it via npm:

```shell
npm install --save power-of-api
```

## How to run

### In Memory

Use below command to import the package and running with simple configuration 

```js
// File: index.js
import app from 'power-of-api';

// In Memory configuration
const port = 1337; // port can be configurable
app.port(port)
    .start(() => {
        console.log(`Server start at ::${port}`)
    }, (err) => {
        console.log(`Exception: `, err)
    });


```

### In File storage

Create *db-data* folder in root path and provide the path to app as mentioned below to utlize the folder to store the data.

```js
// File: index.js
import app from 'power-of-api';

import path from 'path'
import inFileManager from 'power-of-api/plugins/in-file-manager.js';

const port = 1337
app.port(port)
    .use({
        dbManager: await inFileManager.getDbManager({
            folderPath: path.resolve('./db-data')
        })
    })
    .start(() => {
        console.log(`Server start at ::${port}`)
    }, (err) => {
        console.log(`Exception: `, err)
    });

```

### MongoDB

You need to install **mongodb** package in your project, to utilize mongodb data store.

```shell
npm install mongodb@6.7.0
```

Use below code, to configure the app to utilize mongodb data store.

```js
// File: index.js
import app from 'power-of-api';

import { MongoClient, ObjectId } from 'mongodb'; 
import mongodbManager from 'power-of-api/plugins/mongodb-manager.js';

const port = 1337
app.port(port)
    .use({
        dbManager: await mongodbManager.getDbManager({ MongoClient, ObjectId }, {
            url: 'mongodb://localhost:27017',
            dbName: 'powerofapi'
        })
    })
    .start(() => {
        console.log(`Server start at ::${port}`)
    }, (err) => {
        console.log(`Exception: `, err)
    });
```


To run the API, simply execute `node index.js` in your terminal.

Once installed, you can use the following curl commands for CRUD operations:

### Create

Below endpoint can be used to insert multiple records into the system.

```shell
curl --location 'http://localhost:1337?tablename=movies' \
--header 'Content-Type: application/json' \
--data '{
    "items": [
        {
            "name": "Jai Hanuman",
            "actors": [
                {
                    "name": "Teja",
                    "characterName": "Hanumanthu",
                    "castType": "Protagonist"
                }
            ],
            "technicians": [
                {
                    "name": "Prashanth Varma",
                    "technicianType": "Director"
                }
            ]
        },
        {
            "name": "Wall.e",
            "technicians": [
                {
                    "name": "Andrew Stanton",
                    "technicianType": "Director"
                }
            ]
        }
    ]
}'
```

### Read

For now, we are able load all the records for the table.

```shell
curl --location 'http://localhost:1337?tablename=movies'

```

### Update

With the below endpoint and data format, we can update multiple records with **_id** as primary key in the table.

```shell
curl --location 'http://localhost:1337?tablename=books&method=update' \
--header 'Content-Type: application/json' \
--data '{
    "items": [
        {
            "_id": "ffdb2dd18ced25d90118304518fff7e1178",
            "fieldsToUpdate": {
                "authors": [
                    {
                        "name": "Space Ranger"
                    },
                    {
                        "name": "Green star"
                    }
                ]
            }
        }
    ]
}'

```

### Delete
We can delete multiple records with below endpoint, using **_id** as primary key.

```shell
curl --location --request DELETE 'http://localhost:1337?tablename=movies' \
--header 'Content-Type: application/json' \
--data '{
    "items": [
        {
            "_id": "bfbd53acaf873e384814f5dc18fff7d05ed"
        }
    ]
}'
```

### Advanced Configruations

* [Authentication](Authentication-config.md)

### License

[MIT](LICENSE)

### Contributing

We welcome contributions from the open-source community! If you'd like to contribute to power-of-api, please follow best practices and submit a pull request.

### Thank you for using power-of-api!