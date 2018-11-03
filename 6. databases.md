# Creating databases

Today we are going to write a blog. Your own blog with your own design.


## Creating zite

First of all, I recommend creating a new, empty site (in my case, `1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm`), because it can be a bit difficult to repair a site from incorrect database. As usual, I will give you the code in two formats: with plain ZeroFrame and with my library (I believe the latter is better).


## Overview

ZeroNet has builtin database, *SQLite*. Of course, you can create your own DB using files, but this is not serious. Let's begin with creating our database.


## Database source

In ZeroNet, only SQL *SELECT* is allowed. Database is created from JSON files. Let's create one of these files right now.


### `data.json`

By tradition, tables are held in `data/*/data.json` files. Let's create `data/admin/data.json` now. Create an array, `posts`, with rows inside it.

    {
        "posts": [
            {
                "id": 1,
                "title": "Post 1",
                "content": "Post 1"
            },
            {
                "id": 2,
                "title": "Post 2",
                "content": "Post 2"
            }
        ]
    }


## Dbschema format

`dbschema.json` is a simple JSON file. This is empty DB:

    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {},
        "tables": {}
    }

- `db_name` parameter is database name, but it is used only for debug. You cannot have more that one database per site.
- *SQLite* database is only one file, and `db_file` is its path.
- `version` is database, not *SQLite* version. We'll talk about it later.
- `maps` object sets relationship between database and files.
- `tables` object sets table structure.

Please copy this empty DB to `dbschema.json` file in the root of your site.


### `json` table

For each registered user (administrator or simple ZeroID user), a row is created in `json` table. Sadly, ZeroNet doesn't work without this table. We'll talk about it's contents later. Now, let's just add such table.

`dbschema.json`:

    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {},
        "tables": {
            "json": {
                "cols": [
                    ["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
                    ["directory", "TEXT"],
                    ["file_name", "TEXT"]
                ],
                "indexes": ["CREATE UNIQUE INDEX path ON json(directory, file_name)"],
                "schema_changed": 1
            }
        }
    }


### Adding tables

Let's add `posts` table from our `data/admin/data.json` file.

First, let's set table name.

    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {},
        "tables": {
            "json": {
                "cols": [
                    ["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
                    ["directory", "TEXT"],
                    ["file_name", "TEXT"]
                ],
                "indexes": ["CREATE UNIQUE INDEX path ON json(directory, file_name)"],
                "schema_changed": 1
            },
            "posts": {
                ...
            }
        }
    }


As you can see from `json` table specification, each table consists of `cols`, `indexes` and `schema_changed`. `schema_changed` is simply table version. Increment it once you change table structure, so that other peers recreate this table.

    "posts": {
        "cols": [
            ...
        ],
        "indexes": [
            ...
        ],
        "schema_changed": 1
    }


`cols` is column array. First entry of each column is column name (SQL keywords, such as `GROUP` and `UPDATE` are not allowed). Second entry of each column is *SQLite* column type - `integer`, `float` or `text`.

    "posts": {
        "cols": [
            ["id", "integer"],
            ["title", "text"],
            ["content", "text"],
            ["json_id", "integer references json(json_id)"]
        ],
        "indexes": [
            ...
        ],
        "schema_changed": 1
    }

See that `json_id` column? That's another strange thing in `dbschema.json`. Each table must have such column with such type.


`indexes` is index array. Any `CREATE INDEX` SQL command can be there. Index name doesn't mean anything.

    "posts": {
        "cols": [
            ["id", "integer"],
            ["title", "text"],
            ["content", "text"],
            ["json_id", "integer references json(json_id)"]
        ],
        "indexes": [
            "CREATE UNIQUE INDEX post_id ON posts(id)"
        ],
        "schema_changed": 1
    }


Now, we have a DB. Right now, sign `content.json`. You'll find a new file, `data/mydatabase.db` (from `db_file` property of `dbschema.json`). This is *SQLite* database. I recommend *SQLiteStudio* for browsing these files. Right now, you'll find empty `json` and `post` tables, and a special `keyvalue` table. This is ZeroNet internal table, usually we don't need to touch it.


### Mapping tables

#### `to_table`

We have an empty table with correct structure. But who needs empty tables? Let's fill `posts` table from `data/admin/content.json`.

Remember empty `maps` object from `dbschema.json`?

    ...
    "version": 2,
    "maps": {},
    "tables": {
        ...


Let's change it.

    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts"
                }
            ]
        }
    }

Object name `admin/data.json` is *relugar expression*. Each file that matches this expression is checked against object value. Right now you can see only `to_table` property. That means that each row from `posts` object in any file that matches *regular expression* (`admin/data.json`) is added to `posts` table.


Let's sign `content.json` and check database content again. See that `json_id` column in `posts` table? And that row in `json` table? That's OK. `json_id` in `posts` table maps to `json_id` in `json` table. And from `json` table we know, which file added a row. You probably think that's not necessary, but that's not necessary only right now.


## Read access to database

Let's start writing code. Create empty `js/index.js`. Include it and `js/ZeroFrame.js` to `index.html`.

Remember how to call ZeroFrame? Let's call `dbQuery`:

    +-------------------------------------------------------------------------+
    |                                 dbQuery                                 |
    +-------------------------------------------------------------------------+
    | Run a query on the sql cache                                            |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | query                   | Sql query command                             |
    +-------------------------+-----------------------------------------------+
    | Return: Result of query as array or object with "error" property        |
    +-------------------------------------------------------------------------+

    var zeroFrame = new ZeroFrame();
    
    zeroFrame.cmd("dbQuery", ["SELECT * FROM posts"], function(posts) {
        if(posts.error) {
            console.warn(posts.error);
            return;
        }
        
        console.log(posts);
    });

Open console, reload page, and you'll see the following:

    Array [ Object, Object ]
            Object {
                content: "Post 1",
                id: 1,
                json_id: 1,
                title: "Post 1"
            }


We'll cover dynamic data changing and design in the following parts. For now, you can directly change `data/admin/data.json`, sign `content.json` and reload zite to see what changes.
