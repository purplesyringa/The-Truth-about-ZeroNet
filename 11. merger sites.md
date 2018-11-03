# Merger sites

Let's have a look at ZeroMe. It's a social network with about 10 000 users. And each user can write a lot of data. So, we would have to let ZeroMe use around 600 MB of storage. Really bad. Hopefully, ZeroNet introduces merger sites to solve this problem.


## Structure

Imagine the following structure:

    ZeroMe = 500 MB
    |
    +-- data = 500 MB
    |   |
    |   +-- users = 500 MB
    |       |
    |       +-- user1 = 50 KB
    |       +-- user2 = 50 KB
    |       +-- user3 = 50 KB
    |       +-- user4 = 50 KB
    |       +-- user5 = 50 KB
    |       +-- user...
    |       +-- user10000 = 50 KB
    +-- index.js = 250 KB
    +-- index.css = 100 KB
    +-- index.html = 50 KB

Here, users have to download all other users' data (`userN` directories).


And the imagine this:

    ZeroMe = 400 KB
    |
    +-- index.js = 250 KB
    +-- index.css = 100 KB
    +-- index.html = 50 KB
    
    UserDB = 10 MB
    |
    +-- data = 10 MB
    |   |
    |   +-- userinfo = 10 MB
    |       |
    |       +-- user1 = 1 KB
    |       +-- user2 = 1 KB
    |       +-- user3 = 1 KB
    |       +-- user4 = 1 KB
    |       +-- user5 = 1 KB
    |       +-- user...
    |       +-- user10000 = 1 KB
    +-- index.html = 5 KB
    
    Hub1 = 167 MB
    |
    +-- data = 167 MB
    |   |
    |   +-- users = 167 MB
    |       |
    |       +-- user1 = 50KB
    |       +-- user2 = 50KB
    |       +-- user...
    |       +-- user3333 = 50KB
    +-- index.html = 5 KB
    
    Hub2 = 167 MB
    |
    +-- data = 167 MB
    |   |
    |   +-- users = 167 MB
    |       |
    |       +-- user3334 = 50KB
    |       +-- user3335 = 50KB
    |       +-- user...
    |       +-- user6666 = 50KB
    +-- index.html = 5 KB
    
    Hub3 = 167 MB
    |
    +-- data = 167 MB
    |   |
    |   +-- users = 167 MB
    |       |
    |       +-- user6667 = 50KB
    |       +-- user6668 = 50KB
    |       +-- user...
    |       +-- user10000 = 50KB
    +-- index.html = 5 KB

So, we have splitted data to different zites. That's how merger sites work. *ZeroMe* (`1MeFqFfFFGQfa1J3gJyYYUvb5Lksczq7nH` in reality) has only layout and code, *UserDB* (`1UDbADib99KE9d3qZ87NqJF2QLTHmMkoV`) user metadata, while other zites - *hubs* have other data. Of course, downloading these hubs is optional.

For example, if you download `ZeroMe`, `UserDB` and `Hub1`, you can communicate with `user878` and `user678` and learn that there are also `user5412`, `user3453` and `user6789` and that they are assigned to `Hub2` and `Hub3` zites. If you want to say *Hello!* to `user6789` or read his posts, you have to download `Hub3`. But usually you want to read *something*, not *everything*, and you download only 1-2 hubs.


## Making merger site

First of all, let's make a new, empty zite. Open sidebar and name it `PostHere`. Remove all content from `<body>` of `index.html`. We will have data in hubs, and code in main `PostHere` zite.


## Making hub

Let's make a new zite for content. This site address is `1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ` for me.

Change `index.html` to:

    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <meta http-equiv="content-type" content="text/html; charset=utf-8" />
            <base href="" target="_top" id="base">
            <script>
                base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")
            </script>
        </head>
        <body>
            Use PostHere to watch content of this site.
        </body>
    </html>

Change zite title if you want to `Red Hub` and sign `content.json`.


## Configurating merger sites

Merger site (`1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm`, in my case) and merged site (`1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`) must agree to make connection.


So, let's add the following to root `content.json` our merged site (`1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`):

    "merged_type": "PostHere",


Now create a directory `merged-PostHere` in the root of merger site. We want users to sign this folder, so add:

    "ignore": "merged-.*",

...somewhere to `content.json` of main site. To virtually fill it with merger site content, we request `Merger:PostHere` permission:

    +-------------------------------------------------------------------------+
    |                          wrapperPermissionAdd                           |
    +-------------------------------------------------------------------------+
    | Request new permission for site                                         |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | permission              | Name of permission                            |
    +-------------------------+-----------------------------------------------+
    | Return: "Granted" if allowed, not send when disallowed                  |
    +-------------------------------------------------------------------------+


So, add the following to `js/index.js` file and add it to `index.html` of PostHere:

    window.zeroFrame = new ZeroFrame();
    
    function requestPermission(permission, callback) {
        zeroFrame.cmd("siteInfo", [], function(siteInfo) {
            // Already have permission
            if(siteInfo.settings.permissions.indexOf(permission) > -1) {
                callback();
                return;
            }
            
            zeroFrame.cmd("wrapperPermissionAdd", [permission], callback);
        });
    }
    
    requestPermission("Merger:PostHere", function() {
        // TODO
    });


## Accessing merged sites

ZeroNet doesn't provide us a way to read and write other site's content, so MergerSite plugin makes all `merged-...` directories virtual. So to ZeroNet `merged-PostHere` structure is:

    merged-PostHere
    |
    +-- merger.db
    +-- 1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ
        |
        +-- js
        |   |
        |   +-- ZeroFrame.js
        +-- data
        |   |
        |   +-- users
        |       |
        |       +-- user1
        |       |   |
        |       |   +-- data.json
        |       |   +-- content.json
        |       +-- content.json
        +-- index.html

We have set `db_path` to `merged-PostHere/merger.db` because ZeroNet can gather data only from files which are in database directory, which is in this case `merged-PostHere`.

So, if we want to access `data/users/{address}/data.json` of `1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ`, we have to access `merged-PostHere/1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ/data/users/{address}/data.json`.

In the following part of the tutorial, we will fill our `PostHere` zite with user content.
