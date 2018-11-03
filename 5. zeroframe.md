# ZeroFrame

In the previous part, I told you that *localStorage* and some other features are disabled. In this section, I will tell you how to bypass it.


## WebSockets

To let sites read and write files, change *localStorage*, sign data, etc. ZeroNet uses WebSockets. *ZeroFrame* does all dirty work for you. Remember `js/ZeroFrame.js` I told you not to remove in the previous part? That's API interface.


## Installing ZeroFrame

First, let's create a new, really empty zite. Use *ZeroHello* to do this. Now open ZeroNet home directory, then data, now zite public key.

Change `index.html` to:

    <!DOCTYPE html>
    <html>
        <head>
            <title></title>
            <meta charset="utf-8">
            <meta http-equiv="content-type" content="text/html; charset=utf-8" />
            <base href="" target="_top" id="base">
            <script>
                base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")
            </script>
        </head>
        <body>
        </body>
    </html>

I have just removed useless (now) `<script>` from `<body>` and indented code.


Let's include `js/ZeroFrame.js` and create empty `js/index.js`:

    <!DOCTYPE html>
    <html>
        <head>
            <title></title>
            <meta charset="utf-8">
            <meta http-equiv="content-type" content="text/html; charset=utf-8" />
            <base href="" target="_top" id="base">
            <script>
                base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")
            </script>
        </head>
        <body>
            <script type="text/javascript" src="js/ZeroFrame.js"></script>
            <script type="text/javascript" src="js/index.js"></script>
        </body>
    </html>


Open and edit `js/index.js`:

    var zeroFrame = new ZeroFrame();
    console.log(zeroFrame);


Open *DevTools Console* and reload page. You'll notice:

    Object { url: undefined, waiting_cb: Object, wrapper_nonce: "45be457a20334e09dbda338264976328d1f…", target: Object → 1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm, next_message_id: 1 }

...or something like that. That's *ZeroFrame* - a receiver and a transmitter, the thing all ZeroNet was created of.


## First API call

The main function of ZeroFrame is `ZeroFrame.prototype.cmd()`. Let's try to get site info.

    var zeroFrame = new ZeroFrame();
    zeroFrame.cmd("siteInfo", [], function(info) {
        console.log(info);
    });

We already know first line. By the way, you must not create two ZeroFrame's. The rest of file call ZeroNet API. The first argument is API method - here it's `siteInfo`. The second argument is parameters - it is optional, so we could remove `, []` from this code. The third argument is callback.

Reload page and watch console. You'll find something like:

    Object { tasks: 0, size_limit: 10, address: "1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm", next_size_limit: 10, auth_address: "1E3sYXHq99vXW5hS3M9PhQzQhpmWx1paHu", feed_follow_num: null, content: Object, peers: 1, auth_key: "db87a590a113015ddf1b2c73f96baacaa097eee1747d71bdbe5748073bc6131b", settings: Object, 6 more... }


One of the problems of ZeroFrame is its callback nature which results in callback hell. I think that promise hell is better, moreover, promises are compatible with async/await. If you want to use callbacks, use ZeroFrame. If you want to use promises or async/await (the latter is not supported by ZeroNet, but you can use Babel), use my *ZeroPage library* (you can find documentation in extra lessons and download it [here](downloads/ZeroPage.js). I will show how to do that with ZeroFrame, but you can find both versions of resulting zite [here](downloads/notepad.html).

*UPD. In new versions of ZeroHello (sites are in fact created by it) `ZeroFrame.prototype.cmdp()` function was added. It returns a promise, which rejects if result has `error` property and resolves . However, I can still recommend my library because you don't have to write much code for simple things.*


## File system

ZeroFrame allows you to read and write files inside zite directory. The API methods are:

    +-------------------------------------------------------------------------+
    |                                 fileGet                                 |
    +-------------------------------------------------------------------------+
    | Get file content                                                        |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | inner_path              | The file you want to get                      |
    | required (optional)     | Try and wait for the file if it's not exists. |
    |                         | (default: True)                               |
    | format (optional)       | Encoding of returned data. (text or base64)   |
    |                         | (default: text)                               |
    | timeout (optional)      | Maximum wait time to data arrive              |
    |                         | (default: 300)                                |
    +-------------------------+-----------------------------------------------+
    | Return: The content of the file or null or undefined (I don't           |
    | understand the difference between the latter two)                       |
    +-------------------------------------------------------------------------+
    
    +-------------------------------------------------------------------------+
    |                                fileWrite                                |
    +-------------------------------------------------------------------------+
    | Write file content                                                      |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | inner_path              | Inner path of the file you want to write      |
    | content_base64          | Content you want to write to file (base64     |
    |                         | encoded)                                      |
    +-------------------------+-----------------------------------------------+
    | Return: "ok" or error message                                           |
    +-------------------------------------------------------------------------+

If you choosed to use my *ZeroPage* library, I also have *ZeroFS*. You can download it [here](downloads/ZeroFS.js).


Let's change `index.js` content to:

    var zeroFrame = new ZeroFrame();
    
    function readFile(file, callback) {
        ...
    }
    function writeFile(file, content, callback) {
        ...
    }


### Reading files

Before reading the answer, try to write `readFile()` yourself. Ready?

    function readFile(file, callback) {
        zeroFrame.cmd("fileGet", [file, false], callback);
    }


By the way, you could write:

    function readFile(file, callback) {
        zeroFrame.cmd("fileGet", {
            inner_path: file,
            required: false
        }, callback);
    }

...because parameters may be either ommited or array or object (notice parameter names in method documentation).

If you set `required` to `true`, then everything will still work, but a warning will appear.


### Writing files

`writeFile()` is complex. I would say, complicated. Input should be Base64, so you could write:

    function writeFile(file, content, callback) {
        zeroFrame.cmd("fileWrite", [file, btoa(content)], callback);
    }

But `btoa()` doesn't work with UTF8. So we will write another function:

    function base64Encode(content) {
        content = encodeURIComponent(content); // Split to bytes in % notation
        content = unescape(content); // Join % notation as bytes (not as chars)
        return btoa(content);
    }

...and use it instead of `btoa()` in `writeFile()`.


So, we convert UTF8 to bytes. But what if `content` is just byte string? Then, `encodeURIComponent` - `unescape` pair will break bytes because Unicode has multiple codes for several chars. So, we will add another function which uses `btoa()` instead of `base64Encode` (take `writeFile()` and code `writeBinaryFile()` yourself).

Our filesystem library is finished. Well, only file part, but that's enough for now. You can either read my *ZeroFS* library to understand directory part or read documentation here or at [this mirror](/1HP65eGEEMPbyzH3mkaoQ7eCMKd9P8G61W/).


## Notepad

The simpliest program which uses file API is probably text editor. Try to write it yourself. Then try to understand my code from [download section](download/notepad.html).


## Tips

You *can* write `animate()` function each time you need it, but you shouldn't do that. You should save it to some file and copy-paste it when you need it.

The same is true with ZeroNet. There are always problems with it (it's Beta). And sometimes you don't understand why something doesn't work (it took 2 months for me to make databases work - because of lack of documentation I had to read source code). So, feel free to contact me here or at [ZeroMail](/1MaiL5gfBM1cyb4a8e3iiL8L5gXmoAJu27/?to=ivanq).
