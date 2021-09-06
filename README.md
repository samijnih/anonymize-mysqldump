# Anonymize Mysqldump Tool

[![Test Main Branch](https://github.com/DekodeInteraktiv/go-anonymize-mysqldump/actions/workflows/test-main.yml/badge.svg?branch=main)](https://github.com/DekodeInteraktiv/go-anonymize-mysqldump/actions/workflows/test-main.yml)

Allows you to pipe data from `mysqldump` or an SQL file and anonymize it:

```sh
mysqldump -u yada -pbadpass -h db | anonymize-mysqldump --config config.json > anonymized.sql
```

```
usage: anonymize-mysqldump [-h|--help] -c|--config "<value>"

                           Reads SQL from STDIN and replaces content for
                           anonymity based on the provided config.

Arguments:

  -h  --help    Print help information
  -c  --config  Path to config.json
```

## Installation

You can download the binary for your system from the [Releases](https://github.com/DekodeInteraktiv/go-anonymize-mysqldump/releases/) page. Once downloaded and unarchived, move it to a location in your path such as `/usr/local/bin` and make it executable. For instance, to download the MacOS binary for 64 bit platforms (this is most common):

```sh
curl -OL https://github.com/DekodeInteraktiv/go-anonymize-mysqldump/releases/download/latest/go-anonymize-mysqldump_darwin_amd64.gz
gunzip go-anonymize-mysqldump_darwin_amd64.gz
mv go-anonymize-mysqldump_darwin_amd64 /usr/local/bin/anonymize-mysqldump
chmod +x /usr/local/bin/anonymize-mysqldump
```

## Usage

This tool is designed to read a file stream over STDIN and produce an output over STDOUT. A config file is required and can be provided via the `-c` or `--config` flag. An example config for anonymizing a WordPress database is provided at [`config.example.json`](./config.example.json):

```sh
curl -LO https://raw.githubusercontent.com/DekodeInteraktiv/go-anonymize-mysqldump/main/config.example.json
```

Whenever the tool experiences an error, it will output a log to STDERR. If you wish to not see that output while the command is running, redirect it to some other file (or `/dev/null` if you don't care):

```sh
mysqldump -u yada -pbadpass -h db | anonymize-mysqldump --config config.json 2> path/to/errors.log > anonymized.sql
```

## Caveats

Important things to be aware of!

- Currently this only modifies `INSERT` statements. Should you wish to modify other fields, feel free to submit a PR.
- **Verify the output file has been modified.** This is a friendly reminder this tool is still in its early days and you should verify the output sql file before distributing it to ensure the desired modifications have been applied.

## Config File

An example config for anonymizing a WordPress database is provided at [`config.example.json`](./config.example.json).

The config is composed of many objects in the `patterns` array:

- `patterns`: an array of objects defining what modifications should be made.
  - `tableName`: the name of the table the data will be stored in (used to parse `INSERT` statements to d	etermine if the query should be modified.)
  - `tableNameRegex`: regex to identify the relevant tables, required for multisite compatibility e.g. `.*_comments`.
  - `fields`: an array of objects defining modifications to individual values' fields
    - `field`: a string representing the name of the field. Not currently used, but still required to work and useful for debugging.
    - `position`: the 1-based index of what number column this field represents. For instance, assuming a table with 3 columns `foo`, `bar`, and `baz`, and you wished to modify the `bar` column, this value would be `2`.
    - `type`: a string representing the type of data stored in this field. Read more about field types [here](#field-types).
    - `constraints`: an array of objects defining comparison rules used to determine if a value should be modified or not. Currently these are limited to a simple string equality comparison.
      - `field`: a string representing the name of the field.
      - `position`: the 1-based index of what number column this field represents. For instance, assuming a table with 3 columns `foo`, `bar`, and `baz`, and you wished to modify the `bar` column, this value would be `2`.
      - `value`: string value to match against.

### Constraints

Supposing you have a WordPress database and you need to modify certain meta, be it user meta, post meta, or comment meta. You can use `constraints` to update data only whenever a certain condition is matched. For instance, let's say you have a user meta key `last_ip_address`. If you wanted to change that value, you can use the following config in the `fields` array:

```
{
  "field": "meta_value",
  "position": 4,
  "type": "ipv4",
  "constraints": [
    {
      "field": "meta_key",
      "position": 3,
      "value": "last_ip_address"
    }
  ]
}

```

### Field Types

Each column stores a certain type of data, be it a name, username, email, etc. The `type` property in the config is used to define the type of data stored, and ultimately the type of random data to be inserted into the field. [https://github.com/dmgk/faker](https://github.com/dmgk/faker) is used for generating the fake data. These are the types currently supported:

- `username` - `micah_pfeffer`
- `password` - `5ccf672d5c73748146be6b37568efa57`
- `email` - `maritza@farrell.org`
- `url` - `http://sporerhamill.net/kyla.schmitt`
- `name` - `Natasha Hartmann`
- `firstName` - `Carolina`
- `lastName` - `Kohler`
- `phoneNumber` - `+49-131-0003060`
- `addressFull` - `6071 Heaney Island Suite 553, Ebbaville Texas 37307`
- `addressStreet` - `"586 Sylvester Turnpike"`
- `addressPostCode` - `31340`
- `addressCountry` - `Uruguay`
- `paragraph` - `Lorem ipsum dolor sit amet, fabulas nostrum recteque vel ea, sit ut nemore similique. Ad per dicam molestie, nostro constituto duo ad. Ex scripta impedit cum, vidisse feugiat vivendum cum ad, cu liber senserit mediocrem pro.`
- `shortString` - `wqFyJIrXYfVP7cLwqFyJIrXYfVP7cL`
- `ipv4` - `121.204.82.227`
- `companyName` - `Aufderhar LLC`
- `companyNumber` - `123456789`
- `creditCardNumber` - `1234-2121-1221-1211`
- `creditCardExpiryDate` - `2015-11-11`
- `creditCardType` - `mastercard`

If you need another type, please feel free to add support and file a PR!

## Credit

Many thanks to [`Automattic/go-search-replace`](https://github.com/Automattic/go-search-replace) for serving as the starting point for this tool, for [`HumanMade/go-anonymize-mysqldump`](https://github.com/DekodeInteraktiv/go-anonymize-mysqldump) their improvements. Also many thanks to [`xwb1989/sqlparser`](https://github.com/xwb1989/sqlparser) for the SQL parsing library. We wouldn't have been able to do this without them!
