+++
date = 2020-02-12T00:00:00Z
description = ""
tags = ["Rails", "hasura"]
title = "Using Rails migrations with Hasura"

+++
This guide assumes you have a Rails application setup with postgres database and a Hasura instance running.

# Introduction

Hasura comes with a migration tool out of the box to help you manage your Postgres schema, So if
we create a table on the console then hasura automatically tracks the table and then that
information is added to the metadata which indicates the newly created table should be exposed
via GraphQL engine.
In our case we have a Postgres database which is managed by the Active Record Migration system,
hence we don’t need to depend on Hasura migrations. Whenever we make any changes to our
schema like creating a new table Hasura won’t automatically track the table and expose it to
GraphQL engine, this has to be done manually.
In this guide we will explore how to manually track tables.

# Prerequisites

If you don’t have the CLI installed then follow the installation docs ​[here](https://hasura.io/docs/1.0/graphql/manual/migrations/existing-database.html).

# Set up a project directory

Inside your rails application directory

```bash
$ hasura init --directory hasura --endpoint http://mygraphql-endpoint.com
```

```bash
$ cd hasura
```

The project directory should look more or less the same depending on your application use-case.

![file-structure](/static/image_preview.png)

# Adding tables

Let’s create few tables

```bash
$ rails generate model User name:string
$ rails generate model Post user_id:integer:index body:text
```

Run the migration

```bash
$ rails db:migrate
```

Now head over to hasura console and you should see the newly created tables ready to track, we
won’t be using the console to track each table.

![console-img ><](/static/image_preview_1.png)

Instead, we will export the metadata from the console and update the metadata each time we create
a new table in rails. The hasura CLI has few commands which will let us simplify this task.

# The CLI

Inside the hasura directory type in the following commands

```bash
$ hasura metadata export
```

This command will export Hasura metadata and save it in the migrations/metadata.yaml file. This
file includes all the information about tables that are tracked, permission rules, relationships and
event triggers that are defined on those tables. We will be editing this file manually to indicate the
server to track the tables.
Open up the metadata.yaml file and edit the file by adding the table names you want to track, in our
case posts and users

![metadata.yml ><](/static/image_preview_2.png)

We will have to apply the changes back to the server, type in the following

```bash
$ hasura metadata apply
```

Navigate back to the console and you should see both the users and posts table now being tracked
by the server.

![console-image ><](/static/image_preview_4.png)

Perfect, we have successfully tracked tables created by rails. Hence, every time you do a migration in
your rails app update the metadata.yaml file and apply it back to the server.