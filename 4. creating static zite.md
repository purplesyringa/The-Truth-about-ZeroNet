# Creating static zite

In this part of the tutorial, we are going to create an "About me" zite.


## Creating empty zites

Open *ZeroHello* by starting ZeroNet. Press the epsilon to the right of `Hello ZeroNet_` and choose `Create new, empty site`. Wait a few seconds and a new zite will open.

You will find something like that:

    Page address: 1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm
    - Peers: 1
    - Size: 5380
    - Modified: Mon Aug 14 2017 08:14:58 GMT+0300

- *Page address* is the site's public key.
- *Peers* is the number of people which your local ZeroNet knows and have your zite. *Peers* should not be used as number of people which visit your zite.
- *Size* is the size of your zite in bytes.


## What's in `index.html`

Let's open `index.html` of your zite. You can easily access it by visiting your zite, dragging the 0 button and clicking to open zite directory. If you want to do it manually, let's open ZeroNet home 
directory:

- For macOS, that's `~/Application Support/ZeroNet/` or the place where `ZeroNet.app` lives.
- For Windows, that's the place where ZeroNet was installed (e.g. `C:\Program Files\ZeroNet\`).
- If you used GIT to install ZeroNet, that's the directory of git repo.

Now, open `data` directory and then the directory of your zite address (from the page ZeroNet opened, e.g. `1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm`). All the other directories are your local copies of other zites.

Let's open `index.html`:

    <!DOCTYPE html>
    
    <html>
    <head>
     <title>New ZeroNet site!</title>
     <meta charset="utf-8">
     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
     <base href="" target="_top" id="base">
     <script>base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")</script>
    </head>
    <body>
        
    <div id="out"></div>
    
    <script type="text/javascript" src="js/ZeroFrame.js"></script>
    <script>
        
    class Page extends ZeroFrame {
        setSiteInfo(site_info) {
            var out = document.getElementById("out")
            out.innerHTML =
                "Page address: " + site_info.address +
                "<br>- Peers: " + site_info.peers +
                "<br>- Size: " + site_info.settings.size +
                "<br>- Modified: " + (new Date(site_info.content.modified*1000))
        }
        
        onOpenWebsocket() {
            this.cmd("siteInfo", [], function(site_info) {
                page.setSiteInfo(site_info)
            })
        }
        
        onRequest(cmd, message) {
            if (cmd == "setSiteInfo")
                this.setSiteInfo(message.params)
            else
                this.log("Unknown incoming message:", cmd)
        }
    }
    page = new Page()
    </script>
    
    </body>
    </html>

It's not easy to understand what's happening here, but that's not needed. Let's delete something and show what happens.


    <!DOCTYPE html>
    
    <html>
    <head>
     <title>New ZeroNet site!</title>
     <meta charset="utf-8">
     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
     <base href="" target="_top" id="base">
     <script>base.href = document.location.href.replace("/media", "").replace("index.html", "").replace(/[&?]wrapper=False/, "").replace(/[&?]wrapper_nonce=[A-Za-z0-9]+/, "")</script>
    </head>
    <body>
    </body>
    </html>

That's much better. We removed a useless (for us) script. Now let's reindent this.


    <!DOCTYPE html>
    <html>
        <head>
            <title>New ZeroNet site!</title>
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

Even better. **You may change anything espect `<base>` and the following `<script>` tag.** Each zite you visit runs in a restricted `<iframe>`, so some things like *localStorage* are not possible. I'll tell you how to bypass that in the following part of the tutorial.

You shouldn't also remove `js/ZeroFrame.js` file. It may not be needed for this zite, but practically all other ZeroNet sites use it.


## Tips

### Disabling caching

When you edit files, you may not notice updates in browser. That is because of caching. There should be an option to disable this while *DevTools* is open. *DevTools* has to be open while zite developing then.

- In Firefox, open *DevTools* by pressing F12, click Settings icon in the top right corner of the window, and choose `Disable HTTP cache (while DevTools is open)`.
- In Chrome, open *DevTools* by pressing F12, now press F1, click *Preferences* in the left navigation bar and choose `Disable cache (while DevTools is open)` from *Network* menu.


### Absolute paths don't work

Let's make some `<div>`s and style them.

First, add `<div>1</div>` to `index.html`. A `1` appears.


Now, let's create `css/index.css` with the following content:

    div {
        color: red;
    }

...and include it in `index.html`:

    <link rel="stylesheet" href="/css/index.css" type="text/css">

Nothing changes. But why?.. Because of zite address: `http://127.0.0.1:43110/1CyNApZ4zp7k3SSXsrW54vEFMHHBpDy3nm/`. So `/css/index.css` gets calculated as `http://127.0.0.1:43110/css/index.css`. So, using absolute paths is not allowed in ZeroNet. So, we change `<link>` to:

    <link rel="stylesheet" href="css/index.css" type="text/css">


### `<iframe>`s don't work

Each page you open in ZeroNet has the following code:

    <script>
    // If we are inside iframe escape from it
    if (window.self !== window.top) window.open(window.location.toString(), "_top");
    if (window.self !== window.top) window.stop();
    if (window.self !== window.top && document.execCommand) document.execCommand("Stop", false)
    </script>

So, you cannot `<iframe>` other ZeroNet pages, including other HTML files of your site.


## Design

I'll leave design to you. You can download the final version of my site from [downloads](downloads/porfolio.zip).


## Site settings

To change site description and title, we have to use sidebar. Have you noticed `0` button in the top right corner? Clicking it returns you to *ZeroHello*. Now drag it to the left:

      _____              _____  
     /     \            /     \ 
    |   0   | <------- |   0   |
     \_____/            \_____/ 


The sidebar will appear. You can see connected peers, files, database info (we'll cover it later), your unique address (it changes when you login *ZeroID* or *KaffieID* or *CryptoID*). You can also change site info if you are the owner.

Scroll to the bottom and change `Site title` and `Site description` parameters. Press `Save site settings`. Now the settings are saved in `content.json`. Have you noticed that file in the root of your zite? It holds all information about your zite, including files and settings.

Now let's publish your zite. First, press `Sign` button in the very bottom of the sidebar. Your `content.json` now holds updated information about files and is signed correctly. Now press `Publish`. You will probably see `Content publish failed` message. That's OK, as no other peers have downloaded your zite.

Now, if you have a static IP or Tor mode is on, your zite can be accessed from other peers while your computer is on. If you want to turn your computer down, visit `ZeroGate`: `http://zerogate.tk/{zite public key}/` (You can get public key from address bar of from sidebar).


## .bit address

If you really want to have a nice domain like `ivanq.bit`, use *Namecoin*. I won't cover this, but there are many tutorials in Internet and ZeroNet about it.
