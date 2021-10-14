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
It can be intimidating to interact with databases using database drivers. You'll require a high level of understanding of the database you're working with; it's also not object oriented programming friendly. Object Relational Mappers make this easier by adding a layer of abstraction to the database, making it easier to deal with and speeding up the development process. 

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
### Conclusion