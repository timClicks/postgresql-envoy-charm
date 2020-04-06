# Overview

> **Warning: prototype software**
>
> This charm is a prototype only. It is designed for testing the possibility of 
> interacting with software deployed outside of Juju only. 

This charm provides the `pgsql:db` interface, without installing the database. 
It's useful for connecting your model to a PostgreSQL database created outside of
Juju's control.

This charm is an example of the Envoy pattern. An _envoy charm_ does little work on its
own. Its role is to connect a Juju model to infrastructure that exists outside of Juju.
Envoy charms are also known as _proxy charms_ and _integration charms_.


# Usage


## Deployment

Unlike regular charms, `postgresql-envoy` relies on system administrators to perform
several setup tasks.

### Pre-deployment steps

1. Deploy a PostgreSQL database server. Note the `host` and `port` connection parameters. 
   These will be used as configuration parameters later.
2. Ensure that a user account (a "role" in PostgreSQL terminology) exists that has the 
   ability to create users and databases. This account becomes the `username` and 
   `password` credential pair used within the configuration steps later. 
 
   An example of a SQL command that creates such a user is:

           CREATE USER postgresql_envoy_charm
           WITH
            PASSWORD 'pass123'
            CREATEDB
            CREATEROLE;
3. (Optional) Add a small machine to the model. The default machine size is not required, 
   as this charm does little work. 

        juju add-machine --constraints 'mem=512M cores=1 root-disk=10G'

### Deploy

When deploying `decoy-postgresql`, you should use a placement directive (`--to`) to point
 to a machine that already exists.

    juju deploy cs:tim-clicks/postgresql-envoy  [--to <machine-id>] --config username=<admin-username> --config password=<admin-password> 


### Post-deployment steps

1. Ensure that remote connectivity is enabled PostgreSQL in pg_hba.conf
1. Enable firewall access between the machine hosting the `postgresql-envoy` application
1. If you omitted the `--config` parameters during `juju deploy`, you can add them now:

        juju config postgresql-envoy username=<admin-user> password=<password>


## Relating

Other charms should be able to interact with this charm as they are with `cs:postgresql`.

When a relation is broken, the user account is removed.

## Known Limitations and Issues

**This charm is not suitable for production**

Clean up:

- When the application is removed, data persists. If you require a full clean up, then delete
data manually.


**Blockers for recommended usage**

Security:

- Leaks secrets to log files.

Relations:

- `postgresql-envoy` does not understand the `pgsql:db-admin` relation endpoint. 
