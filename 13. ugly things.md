# Ugly things

This part is for developers who use ZeroNet and ZeroFrame API often. Here I'll talk about some strange things in ZeroFrame API which I know only because I read source code.


## `dbQuery`

### `?` placeholder

From ZeroMe source code:

    return Page.cmd("dbQuery", [
        "SELECT * FROM json WHERE ?", {
            directory: directory
        }
    ], function(subject_rows) {

That is equal to (See note 1):

    SELECT * FROM json WHERE
        directory = ${directory}

You can also use multiple key/value pairs, that means:

    SELECT * FROM json WHERE
        key1 = ${value1} AND
        key2 = ${value2} AND
        key3 = ${value3}


If you want to make some condition negative, prepend key with `not__`:

    Page.cmd("dbQuery", [
        "SELECT * FROM json WHERE ?", {
            not__json_id: 2,
            file_name: "data.json"
        }
    ], handler);

    SELECT * FROM json WHERE
        json_id != 2 AND
        file_name = "data.json"


You can also use array as value: (See note 2)

    Page.cmd("dbQuery", [
        "SELECT * FROM json WHERE ?", {
            not__json_id: [2, 3],
            file_name: ["content.json", "data.json"]
        }
    ], handler);

    SELECT * FROM json WHERE
        json_id NOT IN (2, 3) AND
        file_name IN ("content.json", "data.json")


### `:` placeholder

But what if you want to use some special rules (like `A=a and (B=b or C=c)`) but still want to use placeholders? Use `:`!

From ZeroMe source code:

    return Page.cmd("dbQuery", ["SELECT * FROM json WHERE site = :site AND directory = :directory LIMIT 1", params], (function(_this) {

Pass parameters as usual (object as second argument) and then use `:{key}` placeholder to get that value!


## User content

Remember this `data/users/content.json` we often use?

    {
        "files": {},
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

Currently only site admin can sign user content (eg. delete posts). And what if we want to add moderators?

    ...
    "permission_rules": {
        ".*": {
            "files_allowed": "data.json",
            "max_size": 50000,
            "signers": ["1Abc"]
        }
    },
    ...

We add `signers` to `permission_rules` items! Now, `1Abc` can sign all users' content. And if you move `signers` to a new item, `.*bot@.*`, `1Abc` will be able to sign only `...bot` user content.


## Notes

### Note 1

From `Db/DbCursor.py` of ZeroNet source code:

    def execute(self, query, params=None):
        if isinstance(params, dict) and "?" in query:  # Make easier select and insert by allowing dict params
            if query.startswith("SELECT") or query.startswith("DELETE") or query.startswith("UPDATE"):
                # Convert param dict to SELECT * FROM table WHERE key = ? AND key2 = ? format
                query_wheres = []

### Note 2

From `Db/DbCursor.py` of ZeroNet source code:

    if type(value) is list:
        if key.startswith("not__"):
            query_wheres.append(key.replace("not__", "") + " NOT IN (" + ",".join(["?"] * len(value)) + ")")
        else:
            query_wheres.append(key + " IN (" + ",".join(["?"] * len(value)) + ")")
