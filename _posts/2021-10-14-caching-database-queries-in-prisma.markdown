---
layout: post
title:  "Caching Database Queries In Prisma"
date:   2021-10-14 1A7:04:45 +0200
categories: jekyll update
---

### Table Of Contents
- Introduction
- Prerequisites
- Configuring the API application
    - Setup Database
    - Initialize App
    - Connect Prisma to DB
    - Implement Caching 
- Conclusion

### Introduction
It can be a bit intimidating to interact with databases using database drivers. You'll require a high level of understanding of the database you're working with; it's also not object oriented programming friendly. Object Relational Mappers make this easier by adding a layer of abstraction to the database, making it easier to deal with and speeding up the development process. 

Prisma is an ORM that is commonly used in Node.js and Typescript backend applications. It has a number of distinguishing characteristics that set it apart from other ORMs. It features a clean architecture and excellent documentation that aids new developers. It also facilitates dealing with a database with a robust api.

Caching is a technique used in high-demand applications to improve application performance. It works by storing frequently accessed data in a fast access temporary memory. This in turn reduces the workload on the database, ensuring a better optimized application. 

This article will cover how to achieve database caching with prisma.

### Prerequisites
- NodeJS
- MySQL

### Configuring the API application
#### Setup Database
For this project, we will be using a sample database provided by MySQLTutorial.org. To download the database, enter this command on your terminal.

```bash
curl -o db.zip [https://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip](https://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip)
```

Then, we unzip the file (as it's stored in a zip format)

```bash
unzip db.zip
```

And import it into our mysql server.

```bash
# For other sql files, you might have to create a database before importing
sudo mysql
# Within MySQL
mysql> source mysqlsampledatabase.sql;
mysql> exit;
```

We now have a database to work with 😄. Here are the tables in our database

<p align="center">
    <img src="https://res.cloudinary.com/erenaspire7/image/upload/v1634567401/prisma-assets/Screenshot_53_e3rcqb.png">
</p>

#### Setup Project
We create a project folder and initialize npm

```bash
mkdir my-api && cd my-api
npm init -y
```

Then, we install our api's dependencies

```bash
npm i -D nodemon
npm i @prisma/client dotenv express node-cache object-hash
```

Explain what each dependency does

We then create files 
```bash
touch app.js server.js .env 
mkdir prisma
touch prisma/schema.prisma 
```
Explain what you did

And edit the .env file with the port and the database connection string

```typescript
DATABASE_URL = "mysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DATABASE}?schema=public"
PORT = 5000
```

and setup a working server
```javascript
const express = require("express");
const dotenv = require("dotenv");

dotenv.config();

const app = express();

module.exports = app;

// server.js
const app = require("./app");
const dotenv = require("dotenv");

dotenv.config();

const port = process.env.PORT || 5000;

const server = app.listen(port, () => {
    console.log(`API running on port ${port}`);
});
```

And finally, a script to run our server, within `package.json`

```json
{
    "scripts": {
        "dev": "nodemon server.js"
    }
}
```
To run the application, enter `npm run dev` on your terminal

#### Connect Prisma to DB
In order to connect prisma to our mysql database, we need to create the schema file

```bash
mkdir prisma && cd prisma
touch schema.prisma
```

Then, we add this to our schema
```
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "mysql"
    url = env("DATABASE_URL")
}
```

Explain why

Now, our schema doesn't reflect what we have in our database. Luckily, prisma has a function which imports the current state of the database

```
npx prisma db deploy
```

Now, to make our app work with our db, we use the prisma dependency we installed

```
cd ../ && mkdir database
touch index.js
```

And we edit `index.js`

```
const { PrismaClient} = require("@prisma/client")

const prisma = new PrismaClient();

module.exports = prisma;
```

Nice, now, we create routes to test that our db setup is working, we will be working with the customers table

Edit `app.js`

```javascript
// Other imports above
const prisma = require('./database')

dotenv.config();

const app = express();

app.get("api/customers", (req, res) => {
    const customers = await prisma.customers.findMany();

    return res.status(200).json(customers);
})

app.get("/api/customers/:num", async (req, res) => {
    const customer = await prisma.customers.findUnique({
        where: {
            customerNumber: parseInt(req.params.num),
        },
    });

    return res.status(200).json(customer);
});

module.exports = app;
```

Now, we test our api using postman = Define postman briefly

![All Customers Route](https://res.cloudinary.com/erenaspire7/image/upload/v1634569043/prisma-assets/Screenshot_43_kfhbdi.png)
<p align="center">
    <em>All Customers</em>
</p>

![Customer Route](https://res.cloudinary.com/erenaspire7/image/upload/v1634569043/prisma-assets/Screenshot_44_vfm1pd.png)
<p align="center">
    <em>Unique Customer</em>
</p>

Now this looks fine, but imagine if you had a thousand requests for the same query, that would be massively slow down the application. That's where caching comes in

We are going to implement a simple cache using node-cache. In production environments, it's better to use a dedicated caching system like redis as node-cache uses the RAM of the node application. Prisma doesn't offer caching, however, it uses middlewares, so we are going to use the prisma middleware function to cache our requests.

```JavaScript
// index.js
const { PrismaClient } = require("@prisma/client");
const NodeCache = require("node-cache");
const hash = require("object-hash");

const prisma = new PrismaClient();
const app_cache = new NodeCache({ stdTTL: 24 * 60 * 60 }); // In Seconds

prisma.$use(async (params, next) => {
  if (
    params.action == "findMany" ||
    params.action == "findUnique" ||
    params.action == "findFirst"
  ) {
    const queryKey = hash(params);

    const savedCache = app_cache.get(`${queryKey}`);

    if (savedCache == undefined) {
      const cache = await next(params);

      app_cache.set(`${queryKey}`, cache);

      return cache;
    }

    return savedCache;
  }

  return await next(params);
});

module.exports = prisma;
```
Explain Code

Now, we test the same routes again

![All Customers Route](https://res.cloudinary.com/erenaspire7/image/upload/v1634569043/prisma-assets/Screenshot_46_iaoflr.png)
<p align="center">
    <em>All Customers</em>
</p>

![Customer Route](https://res.cloudinary.com/erenaspire7/image/upload/v1634569043/prisma-assets/Screenshot_45_pohhi9.png)
<p align="center">
    <em>Unique Customer</em>
</p>

There has been a massive improvement from 137ms to 8ms for the customers route, also, for the unique customer route, we have seen an improvement from 24ms to 5ms. Massive ...
### Conclusion