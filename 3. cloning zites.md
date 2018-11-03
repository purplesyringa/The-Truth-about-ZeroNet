# Creating zites

## Cloning zites

Let's start with creating a blog for you. Visit *ZeroHello*, download *ZeroBlog* if you haven't downloaded it yet, hover it, and press the epsilon. A menu appears:

- To favourites
- Update
- Check files
- Pause
- Clone
- Delete

Press *Clone*. Your new blog appears. That's practically all about cloning.


### Changing ZeroBlog

You can change blog title, description, etc. by hovering them and pressing a pencil to the left.

Now, let's post something. Press *Add new post*. Edit post title, content and press *Sign & Publish new content* button in the bottom. When you sign data, `content.json` is changed. When you publish content, you send other peers that have your zite a new version, they send them to other peers, etc.

But now nobody except you have this zite. So, things become difficult. 


## Downloading and updating zites

When you download a zite, ZeroNet opens your peer list and chooses those which already have this zite. For example, your second computer, which doesn't know about your first computer, wants to download your zite. How to connect these computers?

...Via *trackers*. All peers often connect to trackers in *Internet* and tell them: we are on this IP and have these zites: `...`. So, the first computer tells some tracker about a new zite, and the second computer asks this tracker about anybody having this zite. Easy-peasy.

One more word about trackers. You need a static IP or Tor connection to share zites with other peers if these peers don't accept your zite update (for example, they don't have this zite at all).


## Proxies

Anybody can connect to your blog while there is a running computer (serving your zite). But you probably want to shutdown it at night. So it's a good idea to have a computer always working.

Fortunately, there are such computers: *ZeroNet proxies*. [bit.surf](http://bit.surf:43110/) is one of them. You should use them as you use local ZeroNet. On proxies, *ZeroTalk* address is `http://bit.surf:43110/1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`

When your blog is ready, visit `http://bit.surf:43110/{zite public key}/`. You can get public key from address bar: `http://127.0.0.1:43110/{zite public key}/`.

*Note*: bit.surf is no longer working.


## .bit addresses

Every zite can be accessed via public key (e.g. `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT` for *ZeroTalk*). Hosting such zites takes no price. But you probably don't think `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT` is a nice address what if you could have `zerotalk.com`?

...In fact, you can do that via *Namecoin*. *Namecoin* is based on blockchain. If you want to register `.bit` address (for price), you should download a *Namecoin* client and make a `.bit` address there (only `.bit` suffix is supported).

If you don't want to host a *Namecoin* client and host the whole blockchain, you can use online services to host domains, such as Dotbit.me and Peername.

As majority has *Namecoin* plugin enabled, you can access *ZeroTalk* via `http://127.0.0.1:43110/Talk.ZeroNetwork.bit/`.

You can check registered `.bit` addresses at [ZeroName](http://127.0.0.1:43110/zeroname.bit/`).


## Homework

Try to clone *ZeroTalk* and make your own zite.

Todo:
technical clonning explanation
