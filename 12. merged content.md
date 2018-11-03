## Editing merged sites

In the previous part, we made a merger site and a merged site (hub) for our new zite, *PostHere*.


## Creating directory for user content

First of all we have to create `data/users/content.json` in our hub. Nothing new here.

    {
        "ignore": ".*",
        "user_contents": {
            "cert_signers": {
                "zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]
            },
            "permission_rules": {
                ".*": {
                    "files_allowed": "data.json",
                    "max_size": 50000
                }
            },
            "permissions": {}
        }
    }

- `ignore` - don't add users to `files` property.
- `user_contents` - user-signed content.
- `user_contents.cert_signers` - allowed *Certificate Authorities*.
- `user_contents.permission_rules` - multiuser permissions.
- `user_contents.permissions` - single user permissions.

...And include it to root `content.json`:

    ...
    "ignore": "data/users/.*",
    "includes": {
        "data/users/content.json": {
            "signers": [],
            "signers_required": 1
        }
    },
    ...
    

Sign `content.json` and then `data/users/content.json`.


## Database schema

As usual, we will use SQLite. Add this `dbschema.json` to main site:

    {
        "db_name": "merger",
        "db_file": "merged-PostHere/merger.db",
        "version": 3,
        "maps": {
            ".+/data/users/.+/content.json": {
                "to_json_table": ["cert_user_id"]
            },
            ".+/data/users/.+/data.json": {
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
                    ["site", "TEXT"],
                    ["directory", "TEXT"],
                    ["file_name", "TEXT"],
                    ["cert_user_id", "TEXT"]
                ],
                "indexes": ["CREATE UNIQUE INDEX path ON json(directory, site, file_name)"],
                "schema_changed": 2
            },
            "posts": {
                "cols": [
                    ["id", "integer"],
                    ["title", "text"],
                    ["content", "text"],
                    ["date_added", "integer"],
                    ["json_id", "integer references json(json_id)"]
                ],
                "indexes": [
                    "CREATE UNIQUE INDEX post_id ON posts(id)"
                ],
                "schema_changed": 2
            }
        }
    }

- Have a look at `version: 3`. This means: `json` table will now have `site` column, too. For merger sites, `site` is hub address. We also use `merged-PostHere` directory for database. This will be described later.
- Look at `.+/data/users/.+/content.json` object. It has `to_json_table` array with `["cert_user_id"]`. `json` table also has `cert_user_id` column. `to_json_table` means: "take `cert_user_id` property from file and use it for `cert_user_id` column of `json` table for this json". `cert_user_id` is username.


### Getting username

So, sample content is:

    +-------------------------------------------------------------------------+
    |                                  json                                   |
    +---------+---------+-------------------+--------------+------------------+
    | json_id | site    | directory         | file_name    | cert_user_id     |
    +---------+---------+-------------------+--------------+------------------+
    | 1       | 1Red... | data/users/1Cv... | data.json    | NULL             |
    +---------+---------+-------------------+--------------+------------------+
    | 2       | 1Red... | data/users/1CV... | content.json | ivanq@zeroid.bit |
    +---------+---------+-------------------+--------------+------------------+
    +-------------------------------------------------------------------------+
    |                                  posts                                  |
    +---------+-------------+--------------------------+------------+---------+
    | id      | title       | content                  | date_added | json_id |
    +---------+-------------+--------------------------+------------+---------+
    | 1       | Hello world | My first post in this... | 1234567890 | 1       |
    +---------+-------------+--------------------------+------------+---------+

Everything is OK until you want to get post author. For each post, you have to take `json_id`, then get `directory` column of this `json_id` in `json` table, then find a row in `json` table which has `directory` = `directory` of old `json_id` and `file_name` = `content.json` and then get `cert_user_id`. So, simplest DB query is:

    SELECT posts.*, json2.cert_user_id as username FROM posts, json, json AS json2 WHERE json.directory = json2.directory AND json2.file_name = "content.json" AND posts.json_id = json.json_id

Hopefully, ZeroNet has another option for this.

    ...
    ".+/data/users/.+/content.json": {
        "to_json_table": ["cert_user_id"],
        "file_name": "data.json"
    },
    ...

`file_name` works like `JOIN`. For each user's `content.json`, ZeroNet find a row with same `site` and `directory` properties and with `file_name` = `data.json` and adds `cert_user_id` to this row. So, now structure is:

    +-------------------------------------------------------------------------+
    |                                  json                                   |
    +---------+---------+-------------------+--------------+------------------+
    | json_id | site    | directory         | file_name    | cert_user_id     |
    +---------+---------+-------------------+--------------+------------------+
    | 1       | 1Red... | data/users/1Cv... | data.json    | ivanq@zeroid.bit |
    +---------+---------+-------------------+--------------+------------------+
    +-------------------------------------------------------------------------+
    |                                  posts                                  |
    +---------+-------------+--------------------------+------------+---------+
    | id      | title       | content                  | date_added | json_id |
    +---------+-------------+--------------------------+------------+---------+
    | 1       | Hello world | My first post in this... | 1234567890 | 1       |
    +---------+-------------+--------------------------+------------+---------+

...and we can simplify our query:

    SELECT posts.*, json.cert_user_id AS username FROM posts, json WHERE posts.json_id = json.json_id


So, now you can use your memory or previous sections to write the rest of the code. As you remember, ZeroNet uses virtual directories, for merger sites, so there is nothing new here expect file we change: `merged-PostHere/{hub}/data/users/{address}/data.json`.


## Adding and removing hubs

We can show users a list of hubs they doesn't have with `mergerSiteList` command.

    +-------------------------------------------------------------------------+
    |                             mergerSiteList                              |
    +-------------------------------------------------------------------------+
    | Return merged sites.                                                    |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | query_site_info         | If True, then gives back detailed site info   |
    |                         | for merged sites                              |
    +-------------------------+-----------------------------------------------+
    | Return: List of merger sites as object                                  |
    +-------------------------------------------------------------------------+

Try to execute `zeroFrame.cmd("mergerSiteList", [false], console.log.bind(console));` You'll see `Object { 1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ: "PostHere" }` or something like that in the console.

These commands are also often used by merger sites:

    +-------------------------------------------------------------------------+
    |                              mergerSiteAdd                              |
    +-------------------------------------------------------------------------+
    | Start downloading new merger site (requires confirmation if called      |
    | twice in 10 seconds)                                                    |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | addresses               | Site address or list of site addresses        |
    +-------------------------+-----------------------------------------------+
    | Return: Always "ok" (even before site is added)                         |
    +-------------------------------------------------------------------------+
    
    +-------------------------------------------------------------------------+
    |                            mergerSiteDelete                             |
    +-------------------------------------------------------------------------+
    | Stop seeding and delete a merged site                                   |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | address                 | Site address                                  |
    +-------------------------+-----------------------------------------------+
    | Return: "ok" or object with "error" property                            |
    +-------------------------------------------------------------------------+


## Example

As usually, you can watch finished site [here](downloads/posthere.html) and hub [here](downloads/postherehub.html).
