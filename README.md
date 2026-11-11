# Welcome to FixMyStreet Platform

FixMyStreet Platform is an open source project to help people run websites for
reporting common street problems such as potholes and broken street lights to
the appropriate authority.

Users locate problems using a combination of address and sticking a pin
in a map without worrying about the correct authority to report it to.
FixMyStreet then works out the correct authority using the problem location and
type and sends a report, by email or using a web service such as Open311.
Reported problems are visible to everyone so they can see if something has
already been reported and leave updates. Users can also subscribe to email or
RSS alerts of problems in their area.

It was created in 2007 by [mySociety](https://www.mysociety.org/) for reporting
problems to UK councils and has been copied around the world. The FixMyStreet
Platform is now at version 2.6; see CHANGELOG.md for a version history.

## Installation

We've been working hard to make the FixMyStreet Platform easy to install and
re-use in other countries - please see our site at <https://fixmystreet.org/>
for help and documentation in installing the FixMyStreet Platform.

For development, if you have Vagrant installed, you can clone the repo and run
'vagrant up'. We use [Scripts to Rule Them All](https://githubengineering.com/scripts-to-rule-them-all/)
so `script/update` will update your checkout, `script/server` will run a dev
server, and `script/test` will run the tests.

## Running this fork locally (Group 8)

`docker-compose.yml` runs the FixMyStreet v2.6 image, which is the version this
fork is based on, and loads the AduanKu (Baiki) files from this repo. It also adds
MailHog, which catches every email the site sends. Open Docker Desktop, then run
this inside this folder:

```
docker compose up -d
```

The first start downloads the images and sets up the database, so give it a few
minutes. The site shows 502 until it's ready.

| What | Where |
|---|---|
| Site | <http://localhost:8000> |
| Emails (MailHog inbox) | <http://localhost:8025> |
| Admin | <http://localhost:8000/admin> |

To get into the admin, go to `/auth`, enter `superuser@example.org`, click **Email
me a link to sign in**, and open the link in MailHog. The password box doesn't show,
so use the email link.

`docker compose down` stops everything and keeps your data. Adding `-v` also
deletes the database.

### Database

The database is PostgreSQL 9.6 in its own container, `fms26-postgres-1`. Everyone
gets their own copy: it's created on the first start and kept in the Docker volume
`fms26_fixmystreet-pgdata`.

To open it in the terminal (`\dt` lists the tables, `\q` quits):

```
docker exec -it fms26-postgres-1 psql -U fms fixmystreet
```

To use a tool like pgAdmin or DBeaver:

| Host | Port | Database | User | Password |
|---|---|---|---|---|
| localhost | 5434 | fixmystreet | fms | fms |

To give your data to someone else, export it:

```
docker exec fms26-postgres-1 pg_dump -U fms -f /tmp/fixmystreet.sql fixmystreet
docker cp fms26-postgres-1:/tmp/fixmystreet.sql .
```

The other person puts `fixmystreet.sql` in this folder and replaces their database
with it:

```
docker compose stop fixmystreet nginx
docker exec fms26-postgres-1 dropdb -U postgres fixmystreet
docker exec fms26-postgres-1 createdb -U postgres -O fms fixmystreet
docker cp fixmystreet.sql fms26-postgres-1:/tmp/fixmystreet.sql
docker exec fms26-postgres-1 psql -U postgres -q -f /tmp/fixmystreet.sql fixmystreet
docker compose up -d
```

### Emails

All emails land in MailHog, so any made-up address works. Alerts and report emails
are sent on a timer on a real server. There's no timer here, so run them yourself:

```
docker exec fms26-fixmystreet-1 su fms -c "/var/www/fixmystreet/fixmystreet/bin/send-alerts"
docker exec fms26-fixmystreet-1 su fms -c "/var/www/fixmystreet/fixmystreet/bin/send-reports"
```

`send-reports` only sends a report if a council covers it. To add one, go to
**Bodies** in the admin and add a name with **Everywhere** as the area. Then open it
and add a category with any email, set to **Confirmed**, and fill in **Summarise
your changes**. This is a staging setup, so the report email goes to the person who
reported it.

## Contribution Guidelines

Whilst many contributions come as part of people setting up their own
installation for their area, we of course welcome stand-alone contributions as
well. The [*Suitable for
Volunteers*](https://github.com/mysociety/fixmystreet/labels/Suitable%20for%20Volunteers)
label in our GitHub issues hopefully labels some potential tasks that might be
suitable for that situation, though please do search through the other issues
to see if what you're after has been suggested or discussed - or feel free to
add your own issue if not.

## Mobile apps

We've extracted all of the mobile apps from this repository into the
[fixmystreet-mobile repository](https://github.com/mysociety/fixmystreet-mobile).

## Acknowledgements

Thanks to [Browserstack](https://www.browserstack.com/) who let us use their
web-based cross-browser testing tools for this project.

## Examples

* <https://www.fixmystreet.com/>
* <http://www.fiksgatami.no/>
* <http://fixmystreet.ie/>
* <https://www.zueriwieneu.ch/>
