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