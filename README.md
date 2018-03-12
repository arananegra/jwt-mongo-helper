# JWT helper for REST made with typescript for MongoDB 

[![NPM](https://nodei.co/npm/jwt-mongo-helper.png?compact=true)](https://npmjs.org/package/jwt-mongo-helper?compact=true)

## Install

```bash
npm i --save jwt-mongo-helper
```

## Usage
##### **Note**: *This implementation example uses the jsonwebtoken library to generate the tokens. You could use an alternative if you want to. You should change mySecret key.*
This project is made entirely with Typescript. In order to use the connector you should create/implement a register and login service. This example uses Express.

### Login service

- Create a Mongo connection object (Db) and a UserDTO.
```typescript
import {Db} from "mongodb"
import * as jsonwebtoken from "jsonwebtoken"
import {UserDTO, UserBS} from "jwt-mongo-helper"
let connection: Db ...
let userToLogin = new UserDTO();
```
Assign the values to the object fields from the POST body:
```typescript
userToLogin._username = req.body.username;
userToLogin._email = req.body.email;
userToLogin._password = req.body.password;
```
Create and instance of UserBS. If the loginUser method returns a UserDTO object (result different from null) it means that the user exists on the Db (and its credentials had been correctly verified). We can now proceed to create the token using jsonwebtoken library and send it back inside the header field 'token'.
```typescript
// "users" is the collection name used to store the users, connection is the Db Mongo object.
let userBS = new UserBS("users", connection);
let resultOfLoginUser : UserDTO = await userBS.loginUser(userToLogin);
        if (resultOfLoginUser !== null) {
            let token = jsonwebtoken.sign({"data": userToLogin._password}, 'mySecret', {
                    expiresIn: '30s'
                }
            );
            res.header("token", token);
            res.status(200).send();
        } else {
            res.status(401).send("Wrong credentials provided");
        }
```

### Register service

- Create a Mongo connection object (Db) and a UserDTO.
```typescript
import {Db} from "mongodb"
import * as jsonwebtoken from "jsonwebtoken"
import {UserDTO, UserBS} from "jwt-mongo-helper"
let connection: Db ...
let userToRegister = new UserDTO();
```
Assign the values to the object fields from the POST body:
```typescript
userToRegister._username = req.body.username;
userToRegister._email = req.body.email;
userToRegister._password = req.body.password;
```
Create and instance of UserBS. If the registerUser method returns a result different from null (void --> normal path), it means that the user doesn't exist in the collection and the new one will be inserted automatically in the system. We can now proceed to create the token using jsonwebtoken library and send it back inside the header field 'token'.

```typescript
// "users" is the collection name used to store the users, connection is the Db Mongo object.
let userBS = new UserBS("users", connection);
let resultOfRegisterUser = await userBS.registerNewUser(userToRegister);
        if (resultOfRegisterUser !== null) {
            let token = jsonwebtoken.sign({"data": userToRegister._password}, 'mySecret', {
                    expiresIn: '30s' //1h, 2d ...
                }
            );
            res.header(ServicesConstants.TOKEN_HEADER_NAME, token);
            res.status(201).send();
        } else {
            res.status(409).send("User already exists");
        }
```

## License

[MIT](http://vjpr.mit-license.org)