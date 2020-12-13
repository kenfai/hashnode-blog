## How To Use JavaScript Fetch API For Modern AJAX Requests

In this article, we will look at the use of **Fetch API**, a modern AJAX request method in JavaScript, as well as the advantages and things to look out for when using it in your project.

## Prerequisites

It is recommended that you have some basic experience in using JavaScript and have implemented AJAX on a web project.

We will also touch on a few language features in JavaScript such as arrow functions and Promise. If you don't already know it, this article may serve as a good introduction to start using Promise in your next project.

## History

JavaScript has come a long way since its inception in 1995. The web back then was largely static, until **XMLHttpRequest**(XHR) came along around 2002 that really made the web come alive.

However, as with all new technologies, the adoption was slow until  [John Resig](https://twitter.com/jeresig)  wrote a wrapper around it in his **jQuery** library back in 2006 which made it easy to implement XHR on any website. That's when **AJAX**, which stands for "Asynchronous JavaScript and XML", becomes one of the most popular and widely used web technologies even until today.

As the internet progresses, platforms and protocols expand. The limitations and flaws of XHR soon surfaced and it deemed inadequate in many modern use cases.

## Introduction

In 2015, the Fetch API was introduced as a much-improved alternative to XHR. It uses some of the modern language features in JavaScripts such as **Promise**, which provides a much cleaner API and helps avoid callback hell.

In fact, it is so simple that you can make a `GET` request in a single line of code:

```javascript
fetch('https://example.com/books')
```

That's it. In contrast, you will need to call at least three methods on an XHR object just to make a network request. Not to mention the complexity of actually using XHR, such as error handling and multiple event-listeners.

Sure, you could use jQuery and make an AJAX call with a one-liner as well. But that will cost your webpage an extra 70~80kb of traffic and delays just to load the entire jQuery library.

## Promise

Let's look at one of the main advantages of using the Fetch API.

Every `fetch()` request will return a JavaScript Promise object, which is asynchronous by default. This means you could safely and confidently ensure that your code will only run AFTER a response is successfully returned from a network request.

Assuming we are returning some plain text over our fetch request, we can get the result by using `.then()` after the fetch:

```javascript
fetch('http://example.com/message')
    .then(response => response.text())
    .then(text => console.log(text))
```

As you can see, the code is so much simpler and concise compared to using XHR. Also, this is all ready to use in JavaScript without loading any external 3rd party libraries.

## Response

The `response` passed to the `.then()` function is actually a JavaScript Response object. This means that we could parse a JSON result directly on the Response object without having to read the result first.

Assuming the following request expects a response containing JSON data:

```javascript
fetch('https://example.com/users/1')
    .then(response => response.json())
    .then(user => console.log(user.name))
```

The response also contains other useful properties that you can read to check the response statuses. Some of the more common ones include:

a) `Response.ok`

Return a Boolean true or false to indicate the success of the response. It will return `true` for `200`-`299` range statuses, and `false` otherwise.

b) `Response.status`

The status code of the response itself. For example, `200`, `404`, `500` etc.

c) `Response.statusText`

The status message corresponding to the status code. For example, `OK` for `200` response.

Here's an example of how to use them:

```javascript
fetch('https://example.com/users/1')
    .then(response => {
        if (!response.ok && response.statusText != 'OK') {
            if (response.status == 404) {
                console.log('Resource not found!')
            }
        }
    })
```
## Error Handling

To handle any connection error, you simply need to add a `.catch()` method to the chain:

```javascript
fetch('https://invalidurl')
    .then(response => response.json())
    ... // other .then()
    .catch((error) => console.log(error))
```

>❗️ Fetch only throws an error when the network request itself fails for some reasons, such as unreachable host or broken connection. A `500` status response(or `200`, `400` etc.) is considered a successful request by Fetch, and it should be handled in your response instead.

## POST Requests

All the examples we have seen so far are making only GET requests. Even though we have not set any options for our fetch request, it has accomplished quite a few scenarios.

To send a POST request, we need to set a few options to our `fetch()` function in the second parameter. Let's look at an example of sending some JSON data over POST.

```javascript
fetch('https://example.com/posts', {
    method: 'POST',
    headers: {
        'Content-type': 'application/json; charset=UTF-8'
    },
    body: JSON.stringify({
        title: 'New Post',
        content: 'Hello world!'
    }),
})
.then((response) => response.json())
.then((json) => console.log(json));
```

Since the `Content-Type` header is set to `text/plain; charset=UTF-8` by default, you will most likely need to provide one according to the content of your request.

For example, for form data request:

```javascript
fetch('https://example.com/login', {
    method: 'POST',
    headers: {
      'Content-type': 'application/x-www-form-urlencoded; charset=UTF-8'
    },
    body: 'username=admin&password=secret'
})
.then(response => response.text())
.then(text => console.log(text))
```

## 🏁 Conclusion

🎉 That's all you need to know to get started on using Fetch API!

Certainly, there are many more features I have yet to touch on regarding Fetch API. Hopefully, this article will give you a glimpse about the powerful and modern approach towards making AJAX requests using Fetch API compared to XHR. 🚀

To explore further, you may check out the links I shared under the Resources section below.

🙏 Thank you for reading! If you find this article useful, please share it with your followers.

## 📚 Resources

- 🔗 https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- 🔗 https://javascript.info/fetch
