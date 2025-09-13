src = https://www.youtube.com/watch?v=skQXoZ8chxk
date = October 2023
length = 51 min

# Intro

Prisma is an **open-source ORM** for **Node**.js and **TypeScript**.  

## What we'll learn

- how to set up **Prisma** with **NestJS** in order to interact with a **SQL** database
- how the **PrismaClient** allows us to autogenerate **types** and have a **type-safe query builder** for our application
- how Prisma's **built-in migration system** works
- how to define one-to-one, one-to-many and many-to-many relations in a `schema.prisma` file.  

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

# Configuring the `schema.prisma` file

It's the main config file for Prisma, and it consists of 3 main parts: 
- the **datasource**: specifies the details of the data source Prisma should connect to
- the **generator**: specifies which client should generate the types and where the output files should go
- the **data model definition**: specifies the entities in our app which will correspond to tables in our database

Prisma will generate types in our application code based on the schema in this `schema.prisma` file.  

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model Product {
  id Int @default(autoincrement()) @id  
  name String @unique 
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  price Float
  onSale Boolean @default(false)
  availability Availability
}

enum Availability {
  IN_STORE
  ONLINE
}
```

**Note**: 
- to get syntax highlighting in our .prisma file, just install the Prisma extension for VSCodium.  

# Configuring the `docker-compose.yaml` file

- create a docker-compose.yaml file at the root of our project, it should contain the following:
```yaml
services:
  mysql:
    image: mysql
    env_file:
      - .env
    ports:
      - "3306:3306"
```

# Configuring the `.env` file

```bash
DATABASE_URL="mysql://root:password@127.0.0.1:3306/nestjs_prisma"

MYSQL_DATABASE=nestjs_prisma
MYSQL_ROOT_PASSWORD=password
```

# Running our MySQL container

- open a terminal and run `docker-compose up --detach`  
  - The --detach option allows us to keep using the same terminal.

# Using a local database client to connect to our database


