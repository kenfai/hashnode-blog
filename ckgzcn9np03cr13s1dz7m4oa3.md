## Understanding NGINX Location Block Modifiers

🛠 [NGINX](http://nginx.org/)  has become the preferred default web server in recent years. Although it has many benefits over other popular web servers, a poorly configured web server may bring more harm than the benefits that it supposed to provide.

This guide assumes that you have some experience in setting up a web server, understand the purpose of URL rewrites, and basic knowledge of regular expression (RegEx). We will go over all the options available when using the `location` directive, as well as some practical examples to highlight common use cases.

You may scroll to the bottom for a quick **TL;DR** summary. 📘

## 📬 The `location` block

One of the most widely used configuration modules within NGINX is the `location` block. It is responsible for matching the Request URI and to subsequently passes it on to the correct block for further processing.

The `location` directive only concern about the path after the domain name and before any query strings. For example, if we have the following URL:

`https://www.example.com/user/posts?post_id=45#top`

The `location` directive will only use the path `/user/posts` to perform any matching.

### 1. Match any URLs

Let's start with a common example that you have likely encountered in many default configurations and tutorials elsewhere:

```
location / {
    try_files $uri $uri/ =404;
}
```

The above rule will match with any URLs, and try to serve them. For example, it will match with `https://www.example.com/homepage.html` and try to serve the page if it exists. Otherwise, it will return a *404 not found* response.

Just to be clear, the `/` after `location` is not a special symbol. It is actually telling NGINX to match with any URIs that **begins** with `/`, which literally means to match with any URLs since that's the first character after a domain name.

This specific rule usually serves as a fallback that matches last when all other `location` blocks failed to match.

> ❗️ Please note that the keyword here is **begins**. NGINX will always match from the beginning of a URI no matter which rule is being applied.

### 2. Match exact URL

Sometimes, we would like to match an exact URL for NGINX to process separately. For example, redirecting legacy links after a migration or a marketing landing page such as `https://www.example.com/deals`.

To do that, the `location` directive offers us a handy `=` modifier to be used as such:

```
location = /deals {
    return 301 https://$host/marketing/deal/2020;
}
```

With the `=` modifier set, NGINX will only redirect to `https://example.com/marketing/deal/2020` if the request URL is exactly `https://example.com/deals`.

❗️ *It is possible to achieve the same outcome with `location = deals { ... }` but it's better to express your intent in a clear manner.*

### 3. Matching URL prefix

Continuing on with the example before, what if the `=` modifier was not set?

```
location /deals {
    return 301 https://$host/marketing/deal/2020;
}
```

In such configuration, NGINX will redirect to `https://example.com/marketing/deal/2020` as long as the URL **begins** with `/deals`. The following URLs will be considered a match:

- `https://example.com/deals`
- `https://example.com/deals/today`
- `https://example.com/deals/today/12`
- `https://example.com/dealsoftheday`
- `https://example.com/dealsoftheday/latest`
- `https://example.com/dealsoftheday/latest?deal_id=12`

A prefix match is more useful when you have a dynamic page that you want to serve via preprocessing engine such as PHP:

```
location /post {
    rewrite ^/post/(.*)$ /post.php?post_id=$1;
}
```

With that in place, the `post.php` page can be accessed from the URL `https://example.com/post/23` where the value `23` will be used as the `post_id` variable for PHP to serve the correct post content.

Another practical use case would be to deny access to a certain directory on your server. For example, you wish to deny access to a directory that contains your users' uploaded profile photos in `/upload/` directory. Here's how you can apply such restriction:

```
location /upload/ {
    deny all;
}
```

This may do the job well for now until you have more matching cases to deal with. We will reexamine this further down with a more suitable modifier.

### 4. Match URLs using regular expression

Another common use case is when you want to serve static assets such as images to your visitors directly without having to route the request through your web application. NGINX is best known for serving static content efficiently. Therefore, it is wise to take advantage of such a feature.

By using the `~` modifier, it will allow us to define a configuration with regular expression to serve a collection of asset types, and handle it properly when they are not found.

```
location ~ \.(ico|gif|jpe?g|png|js|css)$ {
    try_files $uri =404;
}
```

As you can see from above, having the full power of regular expression makes matching URL pattern very convenient.

❗️ *However, I want to stress that the above configuration is only beneficial if you have another **match all** `location` block that matches all URIs. If your site only serves static content, there's no need for such configuration.*

### 5. Case-insensitivity in regular expression matching

The previous example works well if all your assets have a lower-case file extension. To cover both lower-case and upper-case file extensions, we can use the `~*` modifier for case-insensitive matching.

```
location ~* \.(ico|gif|jpe?g|png|js|css)$ {
    try_files $uri =404;
}
```

This will match `https://www.example.com/images/teapot.jpg` as well as `https://www.example.com/images/teapot.JPG`.

>❗️ Please note that this does not mean that a file named 'teapot.jpg' can be accessed by the URL request `https://www.example.com/images/teapot.JPG`. Instead, it is the underlying OS filesystem that decides whether a file can be served with a case-insensitive request or not, not NGINX.

### 6. Prioritising prefix matching

With multiple `location` blocks in place, how would NGINX decide which block gets matched with the request URI? Here's the order with what we've learned so far:

1. NGINX will always look for an exact match that uses the `=` modifier.
2. If there is no exact match, NGINX will try to match a RegEx `location` block that uses the `~` & `~*` modifiers, in top-down order.
3. If there is no RegEx block match, NGINX will try to match the longest prefix `location` block that doesn't contain any modifiers.

Now, what if you want to match a prefix `location` block(step 3) **first** with the presence of other RegEx blocks(step 2)?

Remember the previous prefix match example that was used to deny access to a directory? Here's what we have before:

```
location /upload/ {
    deny all;
}
```

And here's the previous RegEx `location` block example for serving static assets:

```
location ~* \.(ico|gif|jpe?g|png|js|css)$ {
    try_files $uri =404;
}
```

However, since RegEx `location` blocks will be matched before URL prefix matches, the URL prefix block will never be matched. Therefore, anyone will be able to access all resources in the `/upload/` directory.

To solve this issue, that's where you can use the `^~` modifier to prioritise a `location` block to be matched before other potentially matching RegEx blocks. In fact, NGINX will not perform any further RegEx `location` block matching checks if a `^~` modifier `location` block is found to be a match.

Here's how you can redefine the URL prefix block that imposes access restriction to a specific directory path:

```
location ^~ /upload/ {
    deny all;
}
```

With the above configuration, any request to the resources in `/upload/` will be denied with a 403 Forbidden response, such as `https://www.example.com/upload/profile23.jpg`.

### 7. Determining the order of location matching

That concludes the fourth and last modifier available for use in a `location` directive. Therefore, the final matching order is as follows:

1. NGINX will look for an exact match that uses the `=` modifier.
2. If there is no exact match, NGINX will try to match the longest prefix `location` block that uses the `^~` modifier.
3. If there is no prefix match, NGINX will try to match a RegEx `location` block that uses the `~` & `~*` modifiers, in top-down order.
4. If there is no RegEx block match, NGINX will try to match the longest prefix `location` block that doesn't contain any modifiers.

❗️ *In reality, NGINX will first scan all non-RegEx based prefix `location` blocks(including `^~`) and remember them to be used later if no RegEx `location` blocks are matched. But in my opinion, it's easier to visualize them in just four steps as described above.*

<hr>

## 🎁 Summary

That's all to it about NGINX `location` directive and all **four** of its modifiers. Here's a table to summarize each modifier's purpose and effect.

| Modifier | Purpose | Order | Example | Matching Request URI |
|---|---|---|---|---|
| `=` | For matching an exact URI | First | `location = /deals { ... }` | `https://ex.com/deals` |
| `^~` | For matching URL prefix | Second | `location ^~ /upload/ { ... }` | `https://ex.com/upload/profile34.jpg` `https://ex.com/upload/photos/andy2020.jpg` |
| `~` | For matching URL with RegEx (case-sensitive) | Third | `location ~ \.php$ { ... }` | `https://ex.com/index.php` `https://ex.com/index.php?route=home` `https://ex.com/blog/post.php?post_id=45` |
| `~*` | For matching URL with RegEx (case-insensitive) | Fourth | <code>location ~* \.(gif&#124;jpe?g&#124;png&#124;css&#124;js)$ { ... }</code> | `https://ex.com/images/banner.jpg` `https://ex.com/gallery/cars/ROADSTER.JPEG` `https://ex.com/static/css/styles.css` `https://ex.com/scripts/jquery.js` `https://ex.com/favicon.PNG` |
| *none* | For matching URL prefix | Last | `location / { ... }` | `https://ex.com/` `https://ex.com/index.html` `https://ex.com/homepage.html` `https://ex.com/img/background.png` `https://ex.com/styles/footer.css` `https://ex.com/marketing/subscribe.php` |

❗️Thank you for reading! If you find this useful, feel free to share it with your followers. 🦊

## 📙 References

To find out more about NGINX `location` directive and its technical details, you may read further from the following great resources.

- http://nginx.org/en/docs/http/ngx_http_core_module.html#location
- https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms
- https://www.keycdn.com/support/nginx-location-directive