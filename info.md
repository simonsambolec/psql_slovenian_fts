# Implement Slovenian search in PSQL using FTS

First some explanation, implementation is below.

You can also make this work for other languages if you read the first part.

## Explanation

This folder contains PSQL full text search related information.

File: `slovenian.stop`
Contains stop words. These are words which are considred unnecessary for the search.

This file was found [Here](https://sparknlp.org/2022/03/07/stopwords_iso_sl_3_0.html)

Then the file was parsed to be complient with PSQL requirements (each stop word on its own line).

File: `slovenian.affix` and `slovenian.dict`
These files contain info about Slovenian dictionary.

These two files were found in an OpenOffice Slovenian language extension,
which can be found [Here](https://extensions.openoffice.org/en/project/slovenian-dictionaries-spell-checker-hyphenation-thesaurus-openoffice-3x)

And then I converted the files from ISO-8859-2 to UTF-8 format

```
iconv -f ISO-8859-2 -t UTF-8 /path/to/slovenian.dic -o /path/to/slovenian.dict
iconv -f ISO-8859-2 -t UTF-8 /path/to/slovenian.aff -o /path/to/slovenian.affix
```

Notice that I also renamed them

The files in this directory are now ready-for-postgres!

## Implementation

In order to make use of this:

#### 1. Copy the files `slovenian.stop`, `slovenian.affix` and `slovenian.dict` to `/usr/share/postgresql/YOUR_VERSION/tsearch_data/`

##

(Or find the directory with this command on Linux:
`find / -type d -name "tsearch_data"`)

Docker:
If you are using docker you can copy the files with this commands:

Use `docker ps`to find the ID of your PG container

Copy the files:

```
docker cp slovenian.affix YOUR_CONTAINER_ID:/tmp/
docker cp slovenian.dict YOUR_CONTAINER_ID:/tmp/
docker cp slovenian.stop YOUR_CONTAINER_ID:/tmp/
```

Enter the containers terminal:
`docker exec -it YOUR_CONTAINER_ID bash`

Move the files from tmp folder to tsearch_data folder:

```
mv /tmp/slovenian.afix /usr/share/postgresql/YOUR_VERSION/tsearch_data/
mv /tmp/slovenian.dict /usr/share/postgresql/YOUR_VERSION/tsearch_data/
mv /tmp/slovenian.stop /usr/share/postgresql/YOUR_VERSION/tsearch_data/
```

After we have the files ready, we can move on.

#### 2. Now we can create our new [Ispell](https://www.postgresql.org/docs/current/textsearch-dictionaries.html#TEXTSEARCH-ISPELL-DICTIONARY) dictionary in PSQL:

```
CREATE TEXT SEARCH DICTIONARY slovenian_ispell (
    TEMPLATE = ispell,
    DictFile = slovenian,
    AffFile = slovenian,
    StopWords = slovenian
);
```

Now we need to create a new configuration:

```
CREATE TEXT SEARCH CONFIGURATION slovenian (
    COPY = simple
);
```

And we need to update the configuration

```
ALTER TEXT SEARCH CONFIGURATION slovenian
    ALTER MAPPING FOR asciiword, asciihword, hword_asciipart
    WITH slovenian_ispell;
```

### Testing

Now we can test the dictionary with:
`SELECT ts_lexize('slovenian_ispell', 'teniški');`

And we get two lexems from this word:
`{teniški,teniškega}`

Now you can create tsvector using `'slovenian'` as language parameter:
`to_tsvector('slovenian', title)`

I found this video very helpful [Youtube Link - Rob Conery](https://youtu.be/cN3h7bgCg_s?si=cZIHCjIun9DEuRy5) for helping me implement FTS

### Disclaimer

I am no specialist in PostgreSQL
