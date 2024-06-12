# Authentication Config

The package is very loosly coupled with the code, providing ability to change low level functions to implement the authentication in the endpoints.

## Simple configuration

Below configuration is simple authentication mechanism enabled on endpoints with users table to store users.

```js
import app from 'power-of-api';

import path from 'path'
import inFileManager from 'power-of-api/plugins/in-file-manager.js';
import authenticationManager from 'power-of-api/plugins/authentication-manager.js'
const port = 1337
app.port(port)
    .use({
        dbManager: await inFileManager.getDbManager({
            folderPath: path.resolve('./db-data')
        }),
        authentication: await authenticationManager.getAuthentication({
            userTableName: 'users'
        })
    })
    .start(() => {
        console.log(`Server start at ::${port}`)
    }, (err) => {
        console.log(`Exception: `, err)
    });
```
Above command should bring up the endpoints with authentication.


```shell
curl --location 'http://localhost:1337?tablename=movies'
```

you will receive below response for any collection.

```json
{
    "error": {
        "code": "INVALID_ACCESS",
        "message": "Invalid access to this request"
    },
    "serverTime": "2024-06-12T03:07:33.582Z"
}
```

### Register new user in the system

Use below endpoint to register in the system

```shell
curl --location 'http://localhost:1337?action=register' \
--header 'Content-Type: application/json' \
--data '{
    "username": "murali",
    "password": "murali",
    "firstName": "Murali",
    "lastName": "Admin",
    "role": "ADMIN"
}'
```

After successfull registration, you should be able to login into system using below endpoint

```shell
curl --location 'http://localhost:1337?action=authenticate' \
--header 'Content-Type: application/json' \
--data '{
    "username": "murali",
    "password": "murali"
}'

```

You will get below response with correct username and password

```json
{
    "data": {
        "isValid": true,
        "token": "7b22757365726e616d65223a226***"
    },
    "serverTime": "2024-06-12T03:10:00.898Z"
}
```

You can use above token in Authorization header for all endpoints.

```shell
curl --location 'http://localhost:1337?tablename=movies' \
--header 'Authorization: Bearer 7b22757365726e616****'
```

## Advanced configurations

you can control the logic to encrypt/decrypt the token during authentication.

```js
import app from 'power-of-api';

import path from 'path'
import inFileManager from 'power-of-api/plugins/in-file-manager.js';  // plugin for storing data in file system
import authenticationManager from 'power-of-api/plugins/authentication-manager.js' // plugin for enabling authentication
const port = 1337
const tokenTimeoutMin = 20; // token has 20 min expiry time, Zero to create token without expiry time
app.port(port)
    .use({
        dbManager: await inFileManager.getDbManager({
            folderPath: path.resolve('./db-data')
        }),
        authentication: await authenticationManager.getAuthentication({
            userTableName: 'users',
            encryptTokenCallback(dataToEncrypt) {
                // Logic to generate the token based on data (username, role)
                let currentTime = new Date();
                // Below condition is to set the expire field on token if tokenTimeout is >0
                if(tokenTimeoutMin) {
                    currentTime.setMinutes(currentTime.getMinutes() + tokenTimeoutMin);
                    dataToEncrypt['tokenExpiredAt'] = currentTime.getTime();
                }
                const buffer = Buffer.from(JSON.stringify(dataToEncrypt), 'utf8');
                const hexString = buffer.toString('hex');
                return hexString;
            },
            decryptTokenCallback(token) {
                // Logic to decrypt/decode the token and return decrypted/decoded data
                try {
                    const buffer = Buffer.from(token, 'hex');
                    const tokenString = buffer.toString('utf8');
                    const tokenObject = JSON.parse(tokenString);
                    // We check the tokenExpiredAt property in token and check the current time, else we skip the token expiry validation
                    if(tokenObject['tokenExpiredAt']) {
                        let tokenExpiredAt = new Date(tokenObject['tokenExpiredAt']);
                        if (tokenExpiredAt < new Date()) {
                            return null;
                        }
                    }
                    return tokenObject;
                } catch (ex) {
                    return null;
                }
            }
        })
    })
    .start(() => {
        console.log(`Server start at ::${port}`)
    }, (err) => {
        console.log(`Exception: `, err)
    });

```

One more configuration you can set is token expiry time for simple configuration using **tokenTimeoutMin** as below.

```js
...
        authentication: await authenticationManager.getAuthentication({
            userTableName: 'users',
            tokenTimeoutMin: 20 // 20 min
        })
...

```