# Musicbrainz Data

Goal: Load subsets of Musicbrainz data into a generic Postgres (without all the custom extensions)

This is a quick and rough hack. No support will be provided.

## Instructions

* Obtain a dump file
  * Use `musicbrainz-docker` and follow the DB-only steps
  * In that folder, run: `docker-compose exec db pg_dump -U musicbrainz -d musicbrainz_db --if-exists --no-owner --no-privileges -x -c -t release -t release_country -t artist -t area -n musicbrainz > dump.sql`
* `git clone https://github.com/chendo/musicbrainz-data`
* `cd musicbrainz-data`
* `docker-compose up -d`
* Load the data: `cat preamble.sql <path to dump> postamble.sql | grep -v "musicbrainz_unaccent" | grep -v mb_simple | grep -v COLLATE | grep -v "CREATE TRIGGER" | grep -v "CREATE CONSTRAINT" | docker-compose exec -T db psql -U postgres -v ON_ERROR_STOP=1`
* Get a shell: `docker-compose exec db psql -U postgres`

## Example queries

```
SELECT
	release.id,
  release.name,
  artist.name artist,
  release_group.type release_type,
  date_year,
  date_month,
  date_day
FROM
	release_group
  INNER JOIN release ON release_group.id = release.id
	INNER JOIN release_country ON release.id = release_country.release
  INNER JOIN artist ON release.artist_credit = artist.id
WHERE date_month = 3 AND date_day = 1;
```

## ChatGPT prompt

Use at your own risk etc.

I've left out some of the columns.

```
For a Postgres database with the following tables:

                                          Table "public.area"
      Column      |           Type           | Collation | Nullable |             Default
------------------+--------------------------+-----------+----------+----------------------------------
 id               | integer                  |           | not null | nextval('area_id_seq'::regclass)
 gid              | uuid                     |           | not null |
 name             | character varying        |           | not null |
 type             | integer                  |           |          |
 edits_pending    | integer                  |           | not null | 0
 last_updated     | timestamp with time zone |           |          | now()
 begin_date_year  | smallint                 |           |          |
 begin_date_month | smallint                 |           |          |
 begin_date_day   | smallint                 |           |          |
 end_date_year    | smallint                 |           |          |
 end_date_month   | smallint                 |           |          |
 end_date_day     | smallint                 |           |          |
 ended            | boolean                  |           | not null | false
 comment          | character varying(255)   |           | not null | ''::character varying
Indexes:
    "area_pkey" PRIMARY KEY, btree (id)
    "area_idx_gid" UNIQUE, btree (gid)
    "area_idx_name" btree (name)
Check constraints:
    "area_check" CHECK ((end_date_year IS NOT NULL OR end_date_month IS NOT NULL OR end_date_day IS NOT NULL) AND ended = true OR end_date_year IS NULL AND end_date_month IS NULL AND end_date_day IS NULL)
    "area_edits_pending_check" CHECK (edits_pending >= 0)

                                          Table "public.artist"
      Column      |           Type           | Collation | Nullable |              Default
------------------+--------------------------+-----------+----------+------------------------------------
 id               | integer                  |           | not null | nextval('artist_id_seq'::regclass)
 gid              | uuid                     |           | not null |
 name             | character varying        |           | not null |
 sort_name        | character varying        |           | not null |
 begin_date_year  | smallint                 |           |          |
 begin_date_month | smallint                 |           |          |
 begin_date_day   | smallint                 |           |          |
 end_date_year    | smallint                 |           |          |
 end_date_month   | smallint                 |           |          |
 end_date_day     | smallint                 |           |          |
 type             | integer                  |           |          |
 area             | integer                  |           |          |
 gender           | integer                  |           |          |
 comment          | character varying(255)   |           | not null | ''::character varying
 edits_pending    | integer                  |           | not null | 0
 last_updated     | timestamp with time zone |           |          | now()
 ended            | boolean                  |           | not null | false

                                         Table "public.release"
    Column     |           Type           | Collation | Nullable |               Default
---------------+--------------------------+-----------+----------+-------------------------------------
 id            | integer                  |           | not null | nextval('release_id_seq'::regclass)
 gid           | uuid                     |           | not null |
 name          | character varying        |           | not null |
 artist_credit | integer                  |           | not null |
 release_group | integer                  |           | not null |
 status        | integer                  |           |          |
 packaging     | integer                  |           |          |
 language      | integer                  |           |          |
 script        | integer                  |           |          |
 barcode       | character varying(255)   |           |          |
 comment       | character varying(255)   |           | not null | ''::character varying
 edits_pending | integer                  |           | not null | 0
 quality       | smallint                 |           | not null | '-1'::integer


              Table "public.release_country"
   Column   |   Type   | Collation | Nullable | Default
------------+----------+-----------+----------+---------
 release    | integer  |           | not null |
 country    | integer  |           | not null |
 date_year  | smallint |           |          |
 date_month | smallint |           |          |
 date_day   | smallint |           |          |

                                         Table "public.release_group"
    Column     |           Type           | Collation | Nullable |                  Default
---------------+--------------------------+-----------+----------+-------------------------------------------
 id            | integer                  |           | not null | nextval('release_group_id_seq'::regclass)
 gid           | uuid                     |           | not null |
 name          | character varying        |           | not null |
 artist_credit | integer                  |           | not null |
 type          | integer                  |           |          |
 comment       | character varying(255)   |           | not null | ''::character varying
 edits_pending | integer                  |           | not null | 0



 Please my questions about writing SQL queries for the above tables.

```