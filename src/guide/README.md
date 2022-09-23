---
# next: configuration-walkthrough.md
sidebar: auto
---

# Hyperchalk Quickstart

Hyperchalk is an online whiteboard to be used educational research. It provides a useful tool for
collaborative assignments and supports you in collecting data for learning analytics.

If you use Hyperchalk in your scientific work, please cite this paper ([see biblatex citation][bib]):

> Lukas Menzel, Sebastian Gombert, Daniele Di Mitri and Hendrik Drachsler. "Superpowers in the
> Classroom: Hyperchalk is an Online Whiteboard for Learning Analytics Data Collection". In:
> Educating for a new future: Making sense of technology-enhanced learning adoption. Sep 2022.

[bib]: https://github.com/Hyperchalk/Hyperchalk/blob/main/citation.bib

## Installation and Setup

### Introduction

It is highly recommended to use Docker and docker-compose for deploying Hyperchalk. We offer a pre-made container which is kept updated by us:

```sh
docker pull ghcr.io/hyperchalk/hyperchalk
```

 We provide an example [docker-compose.example.yml](https://github.com/Hyperchalk/Hyperchalk/blob/main/docker-compose.example.yml) file which also contains explanatory comments. Hyperchalk relies on [Redis](https://redis.io/) for caching and fast inter-process communication. Inter-process communication is needed as Hyperchalk uses WebSockets to communicate with the client-side and runs on multiple processes for better scalability. When users connect to the same room but are assigned to different processes by the server, this can lead to synchronization issues which can be prevented via inter-process communication. Moreover, the caching offered by Redis decreases the load on the database, and, through this, helps to better scale the app. As suggested by the previous sentence, Hyperchalk also requires an SQL database, namely either [MariaDB/MySQL](https://mariadb.org/) or [PostgreSQL](https://www.postgresql.org/), for data persistence. We are working on supporting more database servers such as MS-SQL and Oracle, but we want to conduct further testing before we provide official support. The compose file also includes basic configurations for Redis and MariaDB containers, but for production, we recommend you to further configure these to your needs. If you intend to use Hyperchalk with existing SQL or Redis instances you have already set up, just remove the respective sections from the docker-compose file. You don't need to worry about setting up any tables. Hyperchalk will conduct this automatically for you. In case an update changes the database scheme, migrations are conducted automatically, as well.

### Configuring Hyperchalk

You should configure Hyperchalk to your needs via the docker-compose file. This is conducted with the helpt of environment variables. The environment variables which are used to configure Hyperchalk are explained in the following table. Adapt them to your needs:

| Variable                         | Description                                                                                                                                                                                                                                                        |
|----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DATABASE_URL                     | The URI of the SQL server (e.g., "mysql://user:password@host:3306/database").                                                                                                                                                                                      |
| HC_REDIS_URL                     | The URI of the Redis server (e.g., "redis://host:6379").                                                                                                                                                                                                           |
| HC_SECRET_KEY                    | The secret key is required to encrypt cookies. We recommend using the command ```sh /dev/urandom tr -dc 'A-Za-z0-9!#$%&()*+,-./:;<=>?@[\]^_`{\|}~' \| head -c64; echo ``` or any other secure string generator.                                                    |
| HC_ALLOWED_HOSTS                 | This variable declares which hosts are allowed and should be set to the domain of your application as well as localhost. If you want to use multiple domains, separate them by comma.                                                                              |
| HC_LINK_BASE                     | This variable contains the URL which is used as the basis for LTI registration links. It should be set to your domain.                                                                                                                                             |
| HC_TIME_ZONE                     | This variable is used to set the time zone of your server.                                                                                                                                                                                                         |
| HC_ALLOW_AUTOMATIC_ROOM_CREATION | If this variable is set to true, new rooms are created when users visit the base URL. This should be enabled if you want to use Hyperchalk outside of an LTI consumer.                                                                                             |
| HC_ALLOW_ANONYMOUS_VISITS        | If this variable is set to true, anyone who visits the base URL of Hyperchalk is presented with a new room. This is useful if you want to use Hyperchalk outside of an LTI consumer and don't want to require people to login before they can access Hyperchalk.   |
| HC_PUBLIC_ROOMS                  | If HC_ALLOW_ANONYMOUS_VISITS is set to false, you can use this variable to create exemptions for particular rooms which should be accessible by all users who are not logged in. You can find the room ids in the admin interface.                                 |
| HC_DEBUG                         | This variable activates the debug mode. In debug mode, static files are delivered directly by the Hyperchalk server which is not recommended for production. Moreover, additional files for debugging are delivered which increases data consumption drastically.  |
| HC_SERVE_FILES                   | If this variable is set to true, static files are delivered directly by the Hyperchalk server. This is only recommended for local testing.                                                                                                                         |
| HC_ADMIN_MAIL                    | Hyperchalk offers the feature to send you mails in cases of critical errors. For this purpose, you can set the E-Mail these mails should be sent to. Important: the mails are only sent it if you have configured an E-Mail server.                                |
| HC_ADMIN_NAME                    | The name by which you want to be addressed in these mails.                                                                                                                                                                                                         |
| HC_EMAIL_HOST                    | (optional) URI of an SMTP server. Hyperchalk can send out mails in case critical events occur.                                                                                                                                                                     |
| HC_EMAIL_PORT                    | (optional) The port of the SMTP server.                                                                                                                                                                                                                            |
| HC_EMAIL_HOST_USER               | (optional) The SMTP user.                                                                                                                                                                                                                                          |
| HC_EMAIL_HOST_PASSWORD           | (optional) The password of the SMTP user.                                                                                                                                                                                                                          |
| HC_EMAIL_USE_TLS                 | (optional) Set to true if TLS should be used for the SMTP connection. This variable is mutually exclusive with HC_EMAIL_USE_SSL.                                                                                                                                   |
| HC_EMAIL_USE_SSL                 | (optional) Set to true if SSL should be used for the SMTP connection. This variable is mutually exclusive with HC_EMAIL_USE_TLS.                                                                                                                                   |
| HC_EMAIL_SUBJECT_PREFIX          | (optional) You can use this variable to set a prefix for the E-Mail titles. This is handy if you want to search all mails sent out by Hyperchalk etc.                                                                                                              |

### Making Hyperchalk ready for Production via Static File Deployment

After you have configured the docker-compose file, the next step is to configure your web server to deploy the static files of Hyperchalk.
These files are not deployed by the Hyperchalk container itself by default unless you activate the debug mode or static file deployment. The reason for this is that 
Django, the web framework we used to implement Hyperchalk, is not optimized for the deployment of static files. Static file deployment might lead
to performance problems in cases where many users try to access the app at the same time. Nonetheless, the container provides all static files needed in two directories (namely ``/srv/static_copy`` and ``/srv/media``) which you should make available via a volumes. In our example docker-compose file, this is already the case. On the other hand, if you just want to test Hyperchalk locally and don't want to deploy it for production, it is the easiest to just activate the debug mode via the HC_DEBUG environment variable or static file deployment via HC_SERVE_FILES.

For production deployment, the next step is to make the files in these volumes available via a web server. From our experience, many apps are deployed behind an [Nginx](https://www.nginx.com/) reverse proxy. As [Nginx](https://www.nginx.com/) is also optimized for static file deployment, we intend Nginx to be used as deployment server for the static files (of course, you can also use other servers such as [Apache](https://httpd.apache.org/), but we don't offer official support and torubleshooting for these). We provided an [example configuration file](https://github.com/Hyperchalk/Hyperchalk/blob/main/nginx-site.example.conf) for configuring Hyperchalk with the Nginx proxy. You should rename this file to the domain you are using for Hyperchalk and copy it to the sites-available directory of your Nginx reverse proxy (usually /etc/nginx/sites-available if you are running Linux). You should then edit it. In particular, replace all occurences of the ``your.site`` placeholder with your actual domain. In addition to this, you should change line 43 to the directory you mounted ``/srv/static_copy`` to and line 48 to the directory you mounted ``/srv/media`` to. 

Now you are good to go. First, start Hyperchalk via:

```sh
docker-compose up -d
```

After this, create a soft-link to the sites-enabled directory to activate Hyperchalk on your Nginx proxy:

```sh
ln -s /etc/nginx/sites-available/<your_domain>.conf /etc/nginx/sites-enabled/
```

Now, you need to reload your nginx proxy via:

```sh
systemctl reload nginx
```

Finally, you should run certbot to acquire an SSH certificate for Hyperchalk (see [this link](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/) for a more detailed description): 

```sh
certbot --nginx -d <your_domain>
```

### Creating a superuser

From here on, you are almost done. The last step which is left is to create a superuser which is needed to access the admin interface (your_domain/admin). This can be done via the following command. You will be asked to assign a username and a password. If you need further superusers, just rerun this command in the future.

```sh
# create an admin user for logging in to the admin backened
docker-compose run --rm hyperchalk manage createsuperuser
```

We wish you a pleasant experience using Hyperchalk 🥳.

## Useful Configuration Documentation Links

You are not restricted to just configuring the things in the example files. For further options,
check out the following sources:

- [Django Deployment Guide] (especially the section where [Gunicorn deployment] is explained.)
- [Channel Layers configuration guide]
- [Gunicorn Configuration](https://docs.gunicorn.org/en/latest/configure.html)
- [Uvicorn Settings Documentation](https://www.uvicorn.org/settings/)
- [Uvicorn Deployment Guide](https://www.uvicorn.org/deployment/)

[channel layers configuration guide]: https://channels.readthedocs.io/en/stable/topics/channel_layers.html#configuration
[django deployment guide]: https://docs.djangoproject.com/en/3.2/howto/deployment/
[gunicorn deployment]: https://docs.djangoproject.com/en/3.2/howto/deployment/wsgi/gunicorn/
[django settings guide]: https://docs.djangoproject.com/en/3.2/topics/settings/
[settings reference]: https://docs.djangoproject.com/en/3.2/ref/settings/

## Creating LTI registration links

Hyperchalk supports LTI 1.3 Advantage auto-configuration. For this purpose, your consumer app will ask you for a link. You can create such a link via the following script:

```sh
# create a registration link
$ docker-compose run --rm hyperchalk manage makeconsumerlink
```

Alternatively, you can use the admin interface for this purpose.

## Supported Data Storage Options

Databases (see [Django configuration guide]):

- Postgres
- MySQL
- SQLite (not recommended for production)

## Recommended Moodle Plugin

Moodle has a strange way to decide the iframe size for LTI apps. To circumvent this, Hyperchalk
supports the [LTI Message Handler](https://moodle.org/plugins/ltisource_message_handler) which can
resize and configure the iframe of the LTI app via cross window messaging. It is advised to install
this when using the app with Moodle.
