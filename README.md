# ZUPC

This project contains the list of all French ZUPC (zone unique de prise en
charge), i.e. an administrative zone to allow taxis from a wider range to pick
up customers in that zone.

The default rule being that basically, taxi drivers can only accept customer
hails when driving around in the city that delivered their licence, unless they
specifically come to pick up a customer who booked that ride.

It is used by [APITaxi](https://github.com/openmaraude/APITaxi) to initialize
the PostgreSQL database, or update it on a yearly basis to take administrative
changes into account.

Please note that it is currently unknown whether this list of ZUPC is exhaustive
or accurate. A zone could be updated or cancelled by a further administrative
notice. And we could miss new zones being created.

Extra note if you're not familiar with the French administration: each commune
is assigned a unique code, called "code commune" or "code INSEE" (after the
national statistics institute managing these codes).

ZUPC themselves don't have a unique identifier, sometimes not even a designated
name.


## Goal

This information is required to know which taxis to show to customers at a given
location.


## Structure

Each directory is a zone, where the name follows the pattern:

  {departement number}\_{type}\_{main city}

where _type_ can be of:

  - `ZUPC` (unique hail zone across a list of towns)
  - `GareTGV` (high-speed train station, outside of downtown)
  - `Aeroport` or `Aerodrome` (big or small airport)

Inside is found copies of the decrees that create or update the ZUPC, and a
description file that is detailed below.

Some decrees may also not create a ZUPC, or just change a rule that doesn't
affect our project, but we keep them anyway to remember we considered them.

When decrees are revised, we only keep the latest copy. When a decree amends a
past one, we try to keep both.


## Steps

The first step is to download the list of French communes (from small towns to
big cities) and their geography from an OpenStreetMap contributor:
http://osm13.openstreetmap.fr/~cquest/openfla/export/communes-20200101-shp.zip

This file is expected to be updated every year.

The ZUPC themselves come from the list of city and department decrees to create
these zones.

Each decree is manually read to produce a description of the zone in a YAML
format. The most common example is a "block" of towns grouping together to have
a single service of taxis, or an airport opening its taxi stations to a
selection of drivers only, based on the town that delivered their licence.

The YAML format was chosen over JSON to allow for comments.

An extra file is read to take town merges into account, typically smaller
villages grouping together in a single town. Some INSEE codes are no longer
used, and taxis being registered there need to be reassigned to the new town
(which is reusing one of the previous INSEE code).


## YAML format

The file describing zones uses the following fields:

- `name`: the name found in the decrees, or made up if not explicitly mentioned.
- `allowed`: a list of INSEE codes, taxis from these communes are allowed (the name of the commune is also used for
  readability and foolproof purposes).
- `feature` (optional): a filename found in the same directory, the file contains a GeoJSON feature, like a polygon.
  If missing, the feature is the merge of all allowed towns geography.
- `include` (optional): extra GeoJSON file to merge with the feature, i.e. airport.
- `exclude` (optional): extra GeoJSON file to exclude from the feature, i.e. where hail is not allowed.


## Usage

This YAML file will be read by the ``import_zupc`` command found in the
[APITaxi](https://github.com/openmaraude/APITaxi/blob/master/APITaxi2/commands/zupc.py)
repository.

At the time of this writing, the big picture is:
- create a temporary table to import the list of communes and their geography;
- add an extra line in this table for each ZUPC found in this repo;
- merge this temporary table into the real one, and update relationships.

The table itself is pretty simple: name, INSEE code, geography, and a
relationship to a parent in the same table. The primary key is left managed by
PostgreSQL, we don't use the INSEE code for that purpose.


## Improvements

The first way to improve this repository is ideally to have French
administrations to contribute when they create or change a ZUPC.

The import script could also be improved for performance and to reduce stress on
the production database.
