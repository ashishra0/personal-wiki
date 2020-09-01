+++
title = "Backup / Restore postgres data on Docker"
tags =["docker"]

date = "2020-01-10"
+++

Postgres comes with handy tools such as **pg_dump** and **psql** which
lets us take backups and restore them with ease.

To back up data, we can use `pg_dump`:

```bash
docker exec <postgres_container_name> pg_dump -U postgres <database_name> > backup.sql
```

This command will create a text file named `backup.sql` containing all
the data and schema of your database.

To import the data back into Postgres, we use `psql`:

```bash
cat backup.sql | docker exec -i <postgres_container_name> psql -U postgres -d <database_name> < backup.sql
```

> Note:
> The above command assumes you have postgres as the default user
