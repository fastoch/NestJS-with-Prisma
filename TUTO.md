src = https://www.youtube.com/watch?v=skQXoZ8chxk
date = October 2023
length = 51 min

# Intro

Prisma is an **open-source ORM** for **Node**.js and **TypeScript**.  

## What we'll learn

- how to set up **Prisma** with **NestJS** in order to interact with a **SQL** database.  
- how the **PrismaClient** allows us to autogenerate **types** and have a **type-safe query builder** 
  for our application.  
- how Prisma's **built-in migration system** works
- how to define one-to-one, one-to-many and many-to-many relations in a `schema.prisma` file.  
**

# Getting started

- open a new terminal in VSCodium
- install NestJS CLI via `sudo npm i -g @nestjs/cli@latest`
- you can check installation via `nest -v`
- use the Nest CLI to initialize a new NestJS project: `nest new <project_name>`
  - choose your favorite package manager
- `cd` into the newly created project folder
- run `npm run start:dev` to start up the dev server 
- we can check that our server is up and running via a GET request at `http://localhost:3000`
  - for that, you can simply use your web browser, or any other HTTP client such as Postman or Bruno

After that, we need to install Prisma and save it as a dev dependency: `npm i prisma -D`  
- now, we can initialize Prisma in our NestJS project via `npx prisma init`

We now have a `prisma` folder with a `schema.prisma` file in it.  
A `.env` file has also been created for us, where we can set our database URL and other environment variables.  

# The `schema.prisma` file

It's the main config file for Prisma, and it consists of 3 main parts: 
- the **datasource**: specifies the details of the data source Prisma should connect to
- the **generator**: specifies which client should generate the types and where the output files should go
- the **data model definition**: specifies the entities in our app which will correspond to tables in our database

Prisma will generate types in our application code based on the schema in this `schema.prisma` file.  

**Note**: 
- to get syntax highlighting in our .prisma file, just install the Prisma extension for VSCodium.  

# Running a SQL database locally

## Setting up the docker-compose file

We will run a MariaDB container (database) and a phpMyAdmin (database client) container via Podman.  
See the `LOCAL_DB_SETUP.md` file for a step-by-step guide.  

- start the mariadb and phpmyadmin containers via `podman start mariadb phpmyadmin`
- open a web browser at http://localhost:8080 (change the port if using a different one) 
  - you should see the login page of phpMyAdmin
- enter the credentials you've set when creating the phpMyAdmin container (see DB_CLIENT.md)