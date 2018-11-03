# Dbschema. We need to go deeper

Today let's speak about some improvements in `dbschema.json`.

Let's use this `dbschema.json`:

    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {
            "admin/data.json": {
                "to_table": [
                    {
                        "node": "posts",
                        "table": "posts"
                    }
                ]
            }
        },
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
                "cols": [
                    ["id", "integer"],
                    ["content", "text"],
                    ["json_id", "integer references json(json_id)"]
                ],
                "indexes": [
                    "CREATE UNIQUE INDEX post_id ON posts(id)"
                ],
                "schema_changed": 1
            }
        }
    }

So, we have a `posts` table with `id` and `content` columns. And we get it from `posts` table in `data/admin/data.json`.


## `key_col`

Think about unique columns. Like votes - the pair `id` and `author_id` is unique. Or simple `id` in our `posts` table. But how can we add unique columns if data is loaded from JSON arrays, and there is no unique data in JSON? Or is there?..

Yes, there is. Remember objects? Keys are unique. So, let's make a unique column a key. Post ID, for example.


In `data/admin/data.json`, we change array to object and move ID to property key.

    {
        "posts": {
            "1": {
                "content": "Post 1"
            },
            "2": {
                "content": "Post 2"
            }
        }
    }


In `dbschema.json`, we added `key_col` property. We told ZeroNet that property key should be `id` column.

    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts",
                    "key_col": "id"
                }
            ]
        }
    }


Sign `content.json` and open `data/mydatabase.db`. Nothing was changed, but now `id` column is always unique - simply because of JSON.

...Well, in fact not always. If there are two equal `admin/data.json` files (e.g. `data/a/admin/data.json` and `data/b/admin/data.json`), there may be two equal IDs. But admin won't do this, and we'll talk about user limitations later.


## `val_col`

But object with one property doesn't look good. It would be nice if we could change `data.json` to:

    {
        "posts": {
            "1": "Post 1",
            "2": "Post 2"
        }
    }


And we can! That's the magic of `val_col`. First of all, let's change our `data/admin/data.json`. Then, let's change `dbschema.json`:

    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts",
                    "key_col": "id",
                    "val_col": "content"
                }
            ]
        }
    }

We added `val_col` property. We told ZeroNet that property value is `content` column. Let's sign `content.json` and see what changed. You can add other posts to check that everything works.


That's why I told that practically any JSON structure is OK. Simple `to_table` and advanced `key_col` and `val_col` can do much.


## `version`

Let's check our `json` table. Currently it has 3 columns: `json_id`, `directory` and `file_name`. That's because of `version: 2`. Let's speak about this `version` property:

- `version: 1`
    1. `json_id` - ID of JSON file
    2. `path` - Path to JSON file
    
    **What problems to expect**: None

- `version: 2`
    1. `json_id` - ID of JSON file
    2. `directory` - Directory of JSON file
    3. `file_name` - Name of JSON file

    **What problems to expect**:
    `directory` is relative to database file, so if file path is `data/admin/data.json`, and database path is `data/mydatabase.db`, then `directory` is `admin`. So, you cannot have a file in the same directory as database.

- `version: 3`
    1. `json_id` - ID of JSON file
    2. `site` - Site of JSON file
    3. `directory` - Directory of JSON file
    4. `file_name` - Name of JSON file
    
    **What problems to expect**:
    The same as `version: 2`, but `site` is the part before the first slash in `directory`, and `directory` is the path after the first slash. So, you cannot have a file in the same directory as database and in a subdirectory of database (but subsubdirectory is OK).
    
    You may ask: "Why `site`?". That's because of merger sites. We'll talk about them later.
