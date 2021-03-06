[![][AMO_button]][AMO]

### 🔵 What is this?

An extension for Firefox that gives users a safe degree of control over CORS requests, with the specific goal of preventing the browser from leaking information to third parties.

### 🔵 What is CORS?

CORS stands for *Cross-Origin Resource Sharing*. In short, it is a mechanism used for bypassing the same-origin policy safely.

[Wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) ▪ [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) ▪ [W3C](https://w3c.github.io/webappsec-cors-for-developers/)

### 🔵 What is the same-origin policy?

It is a standard that has been widely adopted for many years. From the client's perspective, it denies access to resources when these are requested by other resources that were fetched from a different location. Such requests are known as cross-origin requests. 

The same-origin policy is an effective security measure against both [XSS][XSS] and [XSRF][XSRF].

[Wikipedia](https://en.wikipedia.org/wiki/Same-origin_policy) ▪ [MDN](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)

### 🔵 How does CORS work?

Every time the browser makes a cross-origin request, it adds an `Origin` HTTP header to it, which tells the server the location of the resource that triggered the request. After the server parses that header, it decides whether to allow or deny access to its resource from that location. If access is allowed, the sever adds an `Access-Control-Allow-Origin` header to the response, indicating so. The most common values are:

1. `<origin>`: this is the `scheme`+`hostname`+`port` (`https://www.example.org:8080`) of the resource that is allowed access. 
2. `*`: this means the resource is *public*. It can be accessed from anywhere as long as the request does not include credentials.
3. `null`: in practice, this denies access to the resource, but this way is discouraged. The recommended way is to not include an `Access-Control-Allow-Origin` header at all.
4. no header: access is denied.

When the client reads the response headers, the request succeeds or fails based on the presence or absence of the `Access-Control-Allow-Origin` header (and its value). If the request did not include credentials, it only succeeds if the value of that header corresponds to either #1 or #2 (as listed above). If it *did* include credentials, the value must correspond to #1.

### 🔵 How does this extension work?

It has two main modes of operation: aggressive and relaxed.

- The aggressive mode quite simply alters all `GET` requests that include an `Origin` header. This has the potential to break many websites, which is why the extension also allows more fine-grained control via other options like a whitelist and exclusions.
- The relaxed mode uses heuristics to guess which `GET` requests can include credentials, and excludes those automatically. This is the default mode because it is the easiest way to prevent breakage, but since it relies on heuristics, it is by no means perfect. I recommend you to try out the aggressive mode and whitelist sites when needed instead.

When this extension decides to alter a request (after passing it through all the filters), that request is modified as follows:
1. The `Origin` header is removed from it.
2. Since there is no `Origin` header, the server's response most likely does not include an `Access-Control-Allow-Origin` header either, which would normally cause it to fail. To prevent that, this extension injects an `Access-Control-Allow-Origin: *` header into that specific response.

### 🔵 How exactly does the relaxed mode work?

In relaxed mode, a request is excluded automatically when it fulfills any of the following conditions:
- it includes cookies.
- it includes an `Authorization` header.
- it includes the `username`, `password`, `query` or `hash` portions of the URL. `scheme://username:password@hostname:port/path/?query#hash`

### 🔵 What about preflight requests?

Preflight requests use the `OPTIONS` method instead of the `GET` method.

Up to version `1.2.1`, the extension was outright ignoring all non-`GET` requests, including those. However, since `1.3.0` the extension also alters preflight requests, but **only when it knows that the actual request(s) will use the `GET` method**. It does this by reading the `Access-Control-Request-Method` header in the preflight request. If it is found and the value is `GET`, the preflight request itself is altered too, otherwise it is ignored just like before `1.3.0`.

### 🔵 Is this extension *safe*?

Attentive readers shouldn't need me to explain this, but here I go anyway: Yes, this is safe. It will at worst break website functionality, but there are various built-in ways to circumvent that.

Why do I say this is safe? Because this only touches `GET` requests (and preflight requests for `GET` requests), and when it does, it always sets the `Access-Control-Allow-Origin` to `*`. When a request is altered this way, it only succeeds as long as it was not flagged as having credentials. Firefox aborts the request and throws a (healthy) yellow warning in the console otherwise.

Ideally, I would like professionals to let me know if there are any potential dangers I'm overlooking, but that would be quite a luxury. The only potential risks I can imagine are related to badly configured and/or outdated servers, but those risks are inherent to the servers themselves anyway. I suppose this extension would at worst aggravate those risks in some **very** specific (unlikely) scenarios.

If you want to minimize (or even eliminate) those theoretical risks (which would exist even without this extension), enable first-party isolation and/or use containers.

### 🔵 How come no one else made anything like this extension in all these years?

I can't really speak for others, but my guess is only a small subset of extension developers would be willing to hack a security mechanism (ethically).

Additionally, this extension relies on relatively new standards. The same-origin policy and CORS have existed for a long time, but they kept getting updates over the years. It was only a few years ago that [the W3C recommended][W3Creco] the introduction of a *supports credentials* flag and aborting requests flagged as such whenever the server responds with an `Access-Control-Allow-Origin: *`. Before that, the `*` was extremely permissive and risky, which means an extension like this one would've been a lot riskier in the past.

### 🔵 Can CORS leaks be avoided by any other (alternative) means?

The only alternative I know of is to block **all** cross-origin requests. Content blockers like **uBlock Origin** and **uMatrix** allow blocking *third-party* requests, but not all third-party requests are cross-origin requests (it is a broader group).

### 🔵 Why P.O.O.P.?

Because I'm but a lowly hacker-wannabe and I don't want to raise anyone's expectations if I can avoid it. Plus, it was easy to come up with, and it is just as easy to remember.

### 🔵 Dat icon is tacky AF

![Deal with it.][DWI]

Just pretend it's ice cream or something.

### 🔵 Privacy
This extension is meant to *protect* your privacy, not *just* respect it. 

Since you're on Firefox and you seem to care about your privacy, I might as well recommend you to take a good look at [this project](https://github.com/ghacksuserjs/ghacks-user.js), which is where this extension was first conceived.

### 🔵 Source code and changelog

See the [release notes][releases] in the project's [Github repository][repo].

### 🔵 Acknowledgments
- Big thanks to [crssi](https://github.com/crssi) for [bringing attention][issue] to this previously overlooked tracking vector, for all the help testing, and for all the feature suggestions and valuable feedback. If not for him, the extension would still be the half-assed solution I first came up with, because I'm quite the lazy bum.
- Other alpha/beta testers (in no particular order):
  - [StanGets](https://github.com/StanGets)
  - [KOLANICH](https://github.com/KOLANICH)
  - [atomGit](https://github.com/atomGit)

[AMO]: https://addons.mozilla.org/firefox/addon/privacy-oriented-origin-policy/
[AMO_button]: https://gist.github.com/claustromaniac/f054061826ac71bf9e122edb2a313bc0/raw/AMO-button_1.png
[releases]: https://github.com/claustromaniac/poop/releases
[repo]: https://github.com/claustromaniac/poop/
[XSS]: https://en.wikipedia.org/wiki/Cross-site_scripting
[XSRF]: https://en.wikipedia.org/wiki/Cross-site_request_forgery
[issue]: https://github.com/ghacksuserjs/ghacks-user.js/issues/509
[W3Creco]: https://www.w3.org/TR/cors/#supports-credentials
[DWI]: https://gist.githubusercontent.com/claustromaniac/f054061826ac71bf9e122edb2a313bc0/raw/dealwithit.gif
