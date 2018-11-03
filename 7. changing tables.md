# Changing tables

In this section, we will cover write access to *SQLite*.


## Auto increment

Now we want to add a new post. But where can we take ID from? There is a nice answer. Let's add a new property to `data/admin/data.json`:

    {
        "next_post_id": 3,
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

Most zites use this way: they add `next_..._id`. We'll do the same. When we add new post, we will take id from `next_post_id` and increment the latter.


## Write access to database

As I told you before, only *SELECT* is supported. Remember where we edited posts? Yes, in `data/admin/data.json` file. So, let's take our `readFile()` and `writeFile()` functions from the previous lesson:

    function readFile(file, callback) {
        zeroFrame.cmd("fileGet", [file, false], callback);
    }
    
    function writeFile(file, content, callback) {
        zeroFrame.cmd("fileWrite", [file, base64Encode(content)], callback);
    }
    
    function base64Encode(content) {
        content = encodeURIComponent(content); // Split to bytes in % notation
        content = unescape(content); // Join % notation as bytes (not as chars)
        return btoa(content);
    }

...and put them to `js/files.js`. Don't forget to include this file to `index.html` before `index.js`:

    <script type="text/javascript" src="js/files.js"></script>


First of all, let's read `data/admin/data.json` and parse it. Try to do that yourself.

Answer:

    function addPost(title, postContent) {
        readFile("data/admin/data.json", function(content) {
            content = content || ""; // Convert undefined and null to ""
            
            // Parse JSON
            try {
                content = JSON.parse(content);
            } catch(e) {
                content = {
                    posts: [],
                    next_post_id: 0
                };
            }
            
            console.log(content);
        });
    }
    
    addPost("test", "test content");

Open console and reload page. You'll see an object with `posts` and `next_post_id` properties.


Now, we can simply take `next_post_id` and change `posts` array:

    function addPost(title, postContent, callback) { // Notice new callback argument
        readFile("data/admin/data.json", function(content) {
            ...
            // Parse JSON
            ...
            
            content.posts.push({
                id: content.next_post_id++, // Use next_post_id and then increment it
                title: title,
                content: postContent
            });
            
            content = JSON.stringify(content, null, 4); // Make content string
            
            writeFile("data/admin/data.json", content, function(result) {
                callback(result == "ok");
            });
        });
    }
    
    addPost("test", "test content", function(result) {
        console.log(result ? "OK" : "Error");
    });

Try to run this code and see how `data/admin/data.json` changes. Now open `data/mydatabase.db`. Only old posts, right? That's because we have to sign and publish our changes after changing data. Here are API commands we need:

    +-------------------------------------------------------------------------+
    |                                siteSign                                 |
    +-------------------------------------------------------------------------+
    | Sign a content.json of the site                                         |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | privatekey (optional)   | Private key used for signing (default:        |
    |                         | current user's privatekey). "stored" means    |
    |                         | saved site private key                        |
    +-------------------------+-----------------------------------------------+
    | inner_path (optional)   | Inner path of the content json you want to    |
    |                         | sign (default: content.json)                  |
    +-------------------------+-----------------------------------------------+
    | Return: "ok" on success else the error message                          |
    +-------------------------------------------------------------------------+
    
    +-------------------------------------------------------------------------+
    |                               sitePublish                               |
    +-------------------------------------------------------------------------+
    | Publish a content.json of the site                                      |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | privatekey (optional)   | Private key used for signing (default:        |
    |                         | current user's privatekey). "stored" means    |
    |                         | saved zite private key                        |
    +-------------------------+-----------------------------------------------+
    | inner_path (optional)   | Inner path of the content json you want to    |
    |                         | publish (default: content.json)               |
    +-------------------------+-----------------------------------------------+
    | sign (optional)         | If True then sign the content.json before     |
    |                         | publish (default: True)                       |
    +-------------------------+-----------------------------------------------+
    | Return: "ok" on success else the error message                          |
    +-------------------------------------------------------------------------+

We want to both sign and publish zite, so we can use `sitePublish` with `sign` option:

    writeFile("data/admin/data.json", content, function(result) {
        if(result != "ok") {
            callback(false);
            return;
        }
        
        zeroFrame.cmd("sitePublish", {
            privatekey: "stored"
        }, function(result) {
            callback(result == "ok");
        });
    });

Try to reload page multiple times and open `data/mydatabase.db`.


## Private key

The main thing in `siteSign` and `sitePublish` commands is `privatekey` option. It is not logical at all, but as it is used frequently, you have to remember its meaning.

- `privatekey: null` means "use private key which ZeroNet created especially for this zite and especially for this user". Some sites (like ZeroTalk) have user content (posts and votes). To sign this data with user's private key, they don't pass `privatekey` option at all or set it to `null`.
- `privatekey: "stored"` means "use private key of zite". This can be used only if user is zite administrator.
- `privatekey: "..."` means "use this private key".


## Permissions

We also often want to know if user is administrator. For example, for a blog, we should show "Add" button only if user can write to `data/admin/data.json`. ZeroFrame to rescue! We can use `siteInfo` command. It returns a JSON object with `privatekey` boolean - it is true if current user is administrator.

    function isAdmin(callback) {
        zeroFrame.cmd("siteInfo", [], function(info) {
            callback(!!info.privatekey);
        });
    }


## Example

You can watch blog example [here](downloads/blog.html).

Some ideas for improvements:

- Add option to delete post.
- More beautiful editing page.
- Delete created post when `Cancel` is clicked on `Add post` page.
- Use markup language different from HTML.
