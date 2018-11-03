# Security

This section is about security in ZeroNet. If you are not interested in this or don't understand much, just remember a few things and skip this part.

- Each zite has a public key and a private key. You access *ZeroHello* by visiting `http://127.0.0.1:43110/1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D/` - here, `1HeLLo4uzjaLetFx6NH3PMwFP3qbRbTf3D` is a public key. Only site owner has private key, and he publishes changes using it.
- When you want to download a zite, you connect to other peers you know and download zite content from them.
- No one can change any data on zite without a private key for that zite.


## Wrong SHA512

In ZeroNet, everybody hosts zites, but not all - only the zites they want. For example, if `Windows` hosts `ZeroMe`, and `iPad` wants to download it, `iPad` connects to `Windows` *directly* and says:

- **iPad**: Hey, *Windows*, give me `content.json` of `ZeroMe`.
- **Windows**: Here it is: ...
- **iPad**: Thanks! And I also need `index.html`, `js/all.js` and `css/all.css` of `ZeroMe`.
- **Windows**: Here is `index.html`: ... But I have no `js/all.js` and `css/all.css`, sorry.

*Windows* gave *iPad* `index.html`, but where can *iPad* find `js/all.js` and `css/all.css`? From other peers *Windows* knows. But the problem is, if *Windows* is malicious, he can only recommend peers that also didn't have those files.

- **iPad**: Good bye, *Windows*.
- **iPad**: ...Searching peers...
- **iPad**: Hey, *MacOS*, you have `js/all.js` and `css/all.css` of `ZeroMe`?
- **MacOS**: Of course I have! Here they are: ...

But *MacOS* is a hacker! It gives wrong `js/all.js`, with an exploit. Boom! `Windows` has an infected `js/all.js`!

...In fact, no. Each zite has its public key and private key. Public key example is: `1BewKAyyiMZHY3AjQn65J6f6Rcb9p1h64K`. Public key is zite address.

In `content.json` of any zite, there is an SHA512 of each files and size of each file. **MacOS** gave **iPad** a file with other SHA512, so **iPad** says:

- **iPad**: `css/all.css` is OK. `js/all.js`... isn't.
- **iPad**: ...Adds *MacOS* to black list...
- **iPad**: Good bye, *MacOS*.
- **iPad**: ...Searching peers...
- **iPad**: Hey, *Linux*, you have `js/all.js` of `ZeroMe`?
- **Linux**: Yes. Here it is: ...
- **iPad**: Thanks. Good bye, *Linux*.

Now *iPad* has full *ZeroMe* and can show it to you.

That's how ZeroNet makes data secure with SHA512.


## Wrong `content.json`

Now *Linux* needs *ZeroTalk* (its public key is `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`):

- **Linux**: You have `content.json` of `1TaLkFrMwvbNsooF4ioKAY9EuxTBTjipT`?
- **MacOS**: Yes. Take it, please: ...

As we know, *MacOS* is a hacker. It gives wrong `content.json`, with wrong SHA512. *Linux* would ask *MacOS* for `js/all.js`, and *MacOS* would give *Linux* file with exploit.

- **Linux**: Thanks! ...Wait a minute. `content.json` is signed with wrong key.
- **Linux**: ...Adds *MacOS* to black list...
- **Linux**: Good bye, *MacOS*.

*Linux* is clever. `content.json` is signed with private key. *MacOS* would have to steal private key or bruteforce it. But the latter is practically impossible. And you shouldn't let anybody steal zite's private key.


## Signing

Now, we know how data is signed. Only `content.json` is signed, and all other files are signed using `content.json` and its SHA512. It means that nothing can be changed without private key, and usual rainbow tables are useless.


## Restricted policy

As ZeroNet cannot create a new server for every zite you visit, it creates a single server at `http://127.0.0.1:43110/`. So, some more security is needed. ZeroNet opens every zite in a restricted `<iframe>`, so *localStorage* and some other things can not be used (because some zite would be able to access other zite's data).

ZeroNet also does not let zites escape from `<iframe>` - a special token is passed to `<iframe>`, zites use it to communicate with ZeroNet server for some low-level commands. As it changes when you reload the page, bruteforcing `wrapper_nonce` (the token) is the only way to escape `<iframe>`.

But how can data be passed to `<iframe>`? Only with query string. So, there is always a `wrapper_nonce` parameter in query string. So, if you expect query string be `?a=2`, in fact it is `?a=2&wrapper_nonce=e0ccdebc9a804cfd8ac9eaf78a4a03054acbd400ce981219037647922219fbcd`.
