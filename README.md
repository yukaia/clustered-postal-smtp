# Clustered Postal SMTP Environment.

The following is my attempt at making a guide for a clustered postal smtp server environment following their documented best practices using docker compose.

I've ran in to a bug in `Postal 2.1.4` where you're unable to create the initial admin user, so for this we're starting with `2.1.2` After setup you're welcome to decide whether or not if you'd like to upgrade to `2.1.3` or `2.1.4`.

In here you'll find two `docker-compose.yml` files, one is for the postal application stack and the other is for the mariadb and rabbitmq backend. 

----

## Configuration

First we start with the DB and RMQ storage for the app, then we'll move to getting the postal Web, Cron, Queuer, and Runner containers up and running, you're welcome to run them on the same host as the DB and SQL containers, I run them on a separate box, and then finally on a separate server we'll get our first SMTP and Worker containers running, we can run as many of these as we need/like.

### Postal DB and RMQ Server.

First things first we'll want to get our database and rmq servers configured and ready to bring up. For the time being I'm choosing to run the `mariadb:lts` container and `rabbitmq:management`. 

You'll wan to rename `docker-compose.yml-db_rmq-sample` to `docker-compose.yml` and `.env_db_rmq` to `.env`. Don't touch the postal .env and docker-compose samples.

Edit the .env file, define the IP address you want it to listen on, or you can use `0.0.0.0` to have it listen on all available addresses.

Create your db and rqm usernames and passwords, I'd recommend leaving the RMQ VHOST set to postal unless you have a reason to change it.

#### Postal Web, Queue, Cron, and Runner.

You'll wan to rename `docker-compose.yml-postal-sample` to `docker-compose.yml`. Don't touch the db_rmq .env and docker-compose samples.

#### Postal SMTP and Worker.

----

## Postal Config

You can find a walkthrough of the postal config in [config/postal_config_outline.md](config/postal_config_outline.md)

In order to simplify things you'll want to edit one of these `postal.yml` configs and then copy it to all of your servers, it does lead to some problems managing any changes you might make in your environment, however I make all changes on one box, then manually propagate them to the rest of my containers.

----

## Installation.

The following are the installation steps.

#### Postal Initialization.

After you've finished editing the `postal.yml`, `.env` and `docker-compose.yml` files on all of your hosts it's time to initialize your postal environment. On the host that's running the `runner` container run the following commands.

1. First run `docker compose pull` to grab all of the relevant containers.
2. The run `docker compose up run runner postal initialize`
3. Now we can create our initial admin user by running this next. `docker compose up run runner postal make-user`
4. Next we need to generate the dkim key used to sign your mail. You'll need to copy this key to all of your SMTP hosts. `docker compose up run runner postal default-dkim-record`
5. 

----

```bash
    upgrade-db)
        run-docker-compose "run runner postal upgrade"
        ;;

    console)
        run-docker-compose "run runner postal console"
        ;;

    make-user)
        run-docker-compose "run runner postal make-user"
        ;;

    default-dkim-record)
        run-docker-compose "run runner postal default-dkim-record"
        ;;

    test-app-smtp)
        run-docker-compose "run runner postal test-app-smtp"
        ;;
```