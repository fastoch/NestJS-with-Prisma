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
  - The `--detach` option allows us to keep using the same terminal.

# Using a local database client to connect to our database

To install **Beekeeper Studio** via Flatpak on my Fedora 42:
- ensure flatpak is installed via `sudo dnf install flatpak`
- add the Flathub repo via `flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
- install Beekeeper Studio via `flatpak install flathub io.beekeeperstudio.Studio`
- You can then run Beekeeper Studio with `flatpak run io.beekeeperstudio.Studio`

In Beekeeper Studio, create a new MySQL connection, name it as you like, leave the host and port as localhost:3306, and set the username and password 
to `root` and `password` respectively (as set in our .env file).  

Once connected to our database, we can see there is the default database named `nestjs_prisma` that was created as specified in our .env file.  
Of course, we have no tables right now, because we haven't generated any migrations against our schema yet.  

# Executing our first Prisma migration

This migration is going to synchronize our Prisma schema with our current running database.  
- open up a terminal window and run `npx prisma migrate dev --name init`  

![prisma migrate command output](image.png)

![prisma migrate command output 2](image-1.png)

On success, this command returns "Your database in now in sync with your schema".  

## About the `Product` table

And now, in Beekeeper Studio, we should see the `Product` table in our `nestjs_prisma` database.  
In this table, we can see all of the **properties** that we defined in our **model** as **columns**.  

If we look at the **indexes** for this table, we'll see that the `id` is the **primary key**, and that there is a **unique** 
index for the Product `name`.  
These two indexes ensure that no two products will have the same id or name. 

We can also see a `_prisma_migrations` table. Let's explain why this is important.

## About the `_prisma_migrations` table

The `_prisma_migrations` table is created and managed by Prisma to track which database migrations have been applied to a given database instance.  

Each time a migration is executed using Prisma Migrate (with commands like `prisma migrate dev` or `prisma migrate deploy`), a record is added to 
this table containing **metadata** such as the migration name, the timestamp, and a hash of the migration script.   

Prisma uses the data in this table to determine which migrations are already applied and which are still pending, ensuring the database schema stays 
in sync with the migration history in your project.

## What else happens during the migration?

In VSCodium, we should see a new `migrations` folder inside the `prisma` folder.  
This `migrations` folder contains a folder which name is formed of a **timestamp** + underscore + the **migration name**.  
Inside this last folder, we have a `migration.sql` file which contains the **SQL statements** that were used by Prisma to apply the migration.  

These SQL statements are required to get the state of our connected database equivalent to that of our Prisma schema.  

The previous command also regenerates the **Prisma client**.  
This operation generates **TypeScript types** for us based off of our schema, so that we can use these types in our application code.  

# CRUD

Prisma has taken care of everything necessary to get our database ready to work with our NestJS application.  
So now we can start utilizing the types that Prisma Client has generated for us in our application.  

## Creating a DatabaseService

- in the `src` directory, let's create a `database` folder
- inside this `database` folder, let's generate a `database.module` and a `database.service` 
  - for that, let's use the Nest CLI: `nest g module database` and `nest g service database`

We will use that database.service to connect our app to our database through Prisma.  
- let's make our DatabaseService **extend** PrismaClient and **implement** OnModuleInit

### Explanation

- The `@Injectable` decorator allows us to inject DatabaseService and make use of it anywhere in our app 
- Extending `PrismaClient` allows this service to access the types that PrismaClient generates for us (based off of our schema)
- Implementing `OnModuleInit` allows us to connect our app to our database as soon as the application starts up

```ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client/extension';

@Injectable()
export class DatabaseService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
} {}
```

## Building a CRUD API around our model

- run `nest g resource products`
- select 'REST API' as the transport layer
- say Yes to generate CRUD entry points

This will create a products folder inside our src folder with a products.module, a products.controller, a products.service, and DTO files.  

The products.controller has all 