# User content

You have probably visited *ZeroMe* or *ZeroTalk* - a social network and a forum. They are created with *SQLite* - the database we talked about in the previous parts of this tutorial.

In previous parts, we created a zite with content created by administrator (like posts). Remember, we had no votes. Today we are going to fix this. We'll create a voting zite.

First of all, let's make a new zite.


## Including `content.json`

We will have a directory `data/users`. It will have subdirectories (name is user's public key) with user files.

First of all, let's create a `content.json` for user data at `data/users/content.json` (ZeroNet will add other fields on signing):

    {
        "files": {}
    }

...and include it somewhere to root `content.json`:

    ...,
    "ignore": "data/users/.*",
    "includes": {
        "data/users/content.json": {
            "signers": [],
            "signers_required": 1
        }
    },
    ...

The first line with `ignore` property tells ZeroNet that there should be not info about files in `data/users` directory. The other lines say that `data/users/content.json` should also be used when downloading zite.

Sign `content.json` and then `data/users/content.json`.


## Creating correct `content.json`

We are going to use *ZeroID*. Just think: how can peers know that somebody can edit some folder? *ZeroID* and other *Certificate Authorities* prove that user can sign it.

Let's change `data/users/content.json`.

    {
        "ignore": ".*",
        "user_contents": {
            "cert_signers": {
                "zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]
            },
            "permission_rules": {
                ".*": {
                    "files_allowed": "data.json",
                    "max_size": 10000
                }
            },
            "permissions": {
                "ivanq@zeroid.bit": {
                    "max_size": 100000
                }
            }
        }
    }

- `ignore` means that files inside are not signed by administrator.
- `user_contents.cert_signers` is a list of *Certificate Authorities*. Here we have `zeroid.bit` name and `1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz` zite has to prove that user can sign some directory.
- `user_contents.permission_rules` shows user permissions based on regular expressions. The key (`.*`) means "do the following for all users". The string which is matched against this regexp is `{authtype}/{username}@{zite}`. `authtype` is set by *Certificate Authority* (usually `web` or `bitmsg`). `files_allowed` is a regular expression for files which user can have. `max_size` is size (in bytes) of allowed files.
- `user_contents.permissions` lists user permissions for a single user. The key is `{username}@{zite}`. The value is the same as in `permission_rules`.

Here we allow 10 KB for everybody and 100 KB for `ivanq@zeroid.bit` (That's me!).


## Creating `dbschema.json`

Let's create `dbschema.json` with these tables:

    +-------------------------------------------------------------------------+
    |                                questions                                |
    +-------------------------+-----------------------------------------------+
    | Row                     | Type                                          |
    +-------------------------+-----------------------------------------------+
    | id                      | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | question                | TEXT                                          |
    +-------------------------+-----------------------------------------------+
    | answers                 | TEXT                                          |
    +-------------------------+-----------------------------------------------+
    | date_added              | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | json_id                 | INTEGER REFERENCES json(json_id)              |
    +-------------------------------------------------------------------------+
    
    +-------------------------------------------------------------------------+
    |                                 answers                                 |
    +-------------------------+-----------------------------------------------+
    | Row                     | Type                                          |
    +-------------------------+-----------------------------------------------+
    | question_id             | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | answer_id               | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | json_id                 | INTEGER REFERENCES json(json_id)              |
    +-------------------------------------------------------------------------+


`dbschema.json`:

    {
        "db_name": "votes",
        "db_file": "data/votes.db",
        "version": 2,
        "maps": {
            "users/.*/data.json": {
                "to_table": [
                    {
                        "node": "questions",
                        "table": "questions"
                    },
                    {
                        "node": "answers",
                        "table": "answers",
                        "key_col": "question_id",
                        "val_col": "answer_id"
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
            "questions": {
                "cols": [
                    ["id", "integer"],
                    ["question", "text"],
                    ["answers", "text"],
                    ["date_added", "integer"],
                    ["json_id", "integer references json(json_id)"]
                ],
                "indexes": [
                    "CREATE UNIQUE INDEX question_id ON questions(id)"
                ],
                "schema_changed": 1
            },
            "answers": {
                "cols": [
                    ["question_id", "integer"],
                    ["answer_id", "integer"],
                    ["json_id", "integer references json(json_id)"]
                ],
                "indexes": [
                    "CREATE UNIQUE INDEX answer_value ON answers(question_id, json_id)"
                ],
                "schema_changed": 1
            }
        }
    }

The only new part is `users/.*/data.json`. It means: take all files from `data/users/{something}/data.json` and use it the following way: `...`.


## Authorizating via ZeroID

As I said before, we are going to use ZeroID. So, let's write code to do that.

First of all, create `js/index.js` and include it before other files:

    window.zeroFrame = new ZeroFrame();

Let's create a new file, `js/zeroid.js` and include it to `index.html`:

    function authAsZeroID(callback) {
        zeroFrame.cmd("siteInfo", [], function(siteInfo) {
            // If logged in, return object with username and public key (address)
            if(siteInfo.cert_user_id) {
                callback({
                    user: siteInfo.cert_user_id,
                    address: siteInfo.auth_address
                });
                
                return;
            }
            
            // Open authorization window and allow zeroid.bit
            zeroFrame.cmd("certSelect", {
                accepted_domains: ["zeroid.bit"]
            }, function() {
                zeroFrame.cmd("siteInfo", [], function(siteInfo) {
                    // If logged in, return object with username and public key (address),
                    // else return false
                    if(siteInfo.cert_user_id) {
                        callback({
                            user: siteInfo.cert_user_id,
                            address: siteInfo.auth_address
                        });
                    } else {
                        callback(false);
                    }
                });
            });
        });
    }


Now try to call `authAsZeroID(console.log.bind(console))` in console. A window will open. If you choose ZeroID, an object with `user` and `address` properties will appear in the console. If you choose `Unique to this site`, `false` will appear in the console.

We'll learn how to read and write user data in the following parts of the tutorial.
