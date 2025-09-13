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
- how to define one-to-one, one-to-many and many-to-many relations in a **prisma.schema**

# Getting started

- open a new terminal in VSCodium
- install NestJS via `sudo npm i -g @nestjs/cli@latest`
- you can check installation via `nest -v`
- use the Nest CLI to initialize a new NestJS project: `nest new <project_name>`
  - choose your favorite package manager
- `cd` into the newly created project folder
- run `npm run start:dev` to start up the dev server 

