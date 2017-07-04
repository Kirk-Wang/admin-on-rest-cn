# REST 客户端

Admin-on-rest可以与任何rest服务器通信，而不考虑它使用的 REST dialect。 无论是[JSON API](http://jsonapi.org/), [HAL](http://stateless.co/hal_specification.html), [OData](http://www.odata.org/)或者一个自定义dialect，Admin-on-rest唯一需要的就是一个 REST client 函数。这个地方是来转换REST请求到HTTP请求和HTTP响应到REST响应。

![REST client architecture](https://marmelab.com/admin-on-rest/img/rest-client.png)

`<Admin>`组件的`restClient`参数必须是一个具有如下签名的函数：

```jsx
/**
 * Execute the REST request and return a promise for a REST response
 *
 * @example
 * restClient(GET_ONE, 'posts', { id: 123 })
 *  => Promise.resolve({ data: { id: 123, title: "hello, world" } })
 *
 * @param {string} type Request type, e.g GET_LIST
 * @param {string} resource Resource name, e.g. "posts"
 * @param {Object} payload Request parameters. Depends on the action type
 * @returns {Promise} the Promise for a REST response
 */
const restClient = (type, resource, params) => new Promise();
```

你会发现一个实现了REST client的例子在[`src/rest/simple.js`](https://github.com/marmelab/admin-on-rest/blob/master/src/rest/simple.js)；

`restClient`也是在这个理想的地方来添加HTTP头，身份验证，等等。

## 可用客户端{#AvailableClients}

Admin-on-rest默认提供了两个REST客户端：

* [simpleRestClient](#simple-rest)主要服务作为一个例子。顺便说一下，它是与[FakeRest](https://github.com/marmelab/FakeRest)API兼容。
* **[JSON server](https://github.com/typicode/json-server)**：[jsonServerRestClient](#json-server-rest)

你可以在第三方的仓库中为admin-on-rest找到更多REST客户端：

* **[Feathersjs](http://www.feathersjs.com/)**：[josx/aor-feathers-client](https://github.com/josx/aor-feathers-client)
* **[Firebase](https://firebase.google.com/)**：[sidferreira/aor-firebase-client](https://github.com/sidferreira/aor-firebase-client)
* **[GraphQL](http://graphql.org/)**：[marmelab/aor-simple-graphql-client](https://github.com/marmelab/aor-simple-graphql-client) (使用[Apollo](http://www.apollodata.com/))
* **[JSON API](http://jsonapi.org/)**：[moonlight-labs/aor-jsonapi-client](https://github.com/moonlight-labs/aor-jsonapi-client)
* Local JSON：[marmelab/aor-json-rest-client](https://github.com/marmelab/aor-json-rest-client)。它甚至不使用HTTP。 用于测试目的。
* **[Loopback](http://loopback.io/)**: [kimkha/aor-loopback](https://github.com/kimkha/aor-loopback)
* **[Parse Server](https://github.com/ParsePlatform/parse-server)**: [leperone/aor-parseserver-client](https://github.com/leperone/aor-parseserver-client)
* **[PostgREST](http://postgrest.com/en/v0.4/)**: [tomberek/aor-postgrest-client](https://github.com/tomberek/aor-postgrest-client)

如果您为另一个后端编写了REST客户端，并开放源代码，请完善此列表具有您的软件包。

### 简易的REST客户端{#SimpleREST}

这个REST客户端适合API使用简单 GET 参数进行筛选和排序。这个dialect适用于例如在[FakeRest](https://github.com/marmelab/FakeRest)中。

| REST verb            | API calls
|----------------------|----------------------------------------------------------------
| `GET_LIST`           | `GET http://my.api.url/posts?sort=['title','ASC']&range=[0, 24]&filter={title:'bar'}`
| `GET_ONE`            | `GET http://my.api.url/posts/123`
| `CREATE`             | `POST http://my.api.url/posts/123`
| `UPDATE`             | `PUT http://my.api.url/posts/123`
| `DELETE`             | `DELETE http://my.api.url/posts/123`
| `GET_MANY`           | `GET http://my.api.url/posts?filter={ids:[123,456,789]}`
| `GET_MANY_REFERENCE` | `GET http://my.api.url/posts?filter={author_id:345}`

**注意**: 这个简易的REST客户端期望API在响应 `GET_LIST` 调用中包含一个`Content-Range`头。该值必须是集合中的资源总数。这使 admin-on-rest 能够知道总共有多少页资源，并生成分页控件。

```
Content-Range: posts 0-24/319
```

在JS代码中如果您的API是在另一个域中，你需要到白名单中为这个头添加一个`Access-Control-Expose-Headers` [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) 头。

```
Access-Control-Expose-Headers: Content-Range
```

这里是如何在你的admin中用它：

```jsx
// in src/App.js
import React from 'react';

import { simpleRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';

const App = () => (
    <Admin restClient={simpleRestClient('http://path.to.my.api/')}>
        <Resource name="posts" list={PostList} />
    </Admin>
);

export default App;
```

### JSON Server REST客户端{#JSONServerREST}

此REST客户端适合API由[JSON Server](https://github.com/typicode/json-server)驱动比如 [JSONPlaceholder](http://jsonplaceholder.typicode.com/)。

| REST verb            | API calls
|----------------------|----------------------------------------------------------------
| `GET_LIST`           | `GET http://my.api.url/posts?_sort=title&_order=ASC&_start=0&_end=24&title=bar`
| `GET_ONE`            | `GET http://my.api.url/posts/123`
| `CREATE`             | `POST http://my.api.url/posts/123`
| `UPDATE`             | `PUT http://my.api.url/posts/123`
| `DELETE`             | `DELETE http://my.api.url/posts/123`
| `GET_MANY`           | `GET http://my.api.url/posts/123, GET http://my.api.url/posts/456, GET http://my.api.url/posts/789`
| `GET_MANY_REFERENCE` | `GET http://my.api.url/posts?author_id=345`

**注意**: 这个jsonServer REST客户端期望在响应GET_LIST调用中包含一个`X-Total-Count`头。该值必须是集合中的资源总数。这使admin-on-rest能够知道总共有多少页资源，并生成分页控件。

```
X-Total-Count: 319
```

在JS代码中如果您的API是在另一个域中，你需要到白名单中为这个头添加一个`Access-Control-Expose-Headers`[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) 头。

```
Access-Control-Expose-Headers: X-Total-Count
```

这里是如何在你的admin中用它：

```jsx
// in src/App.js
import React from 'react';

import { jsonServerRestClient, Admin, Resource } from 'admin-on-rest';

import { PostList } from './posts';

const App = () => (
    <Admin restClient={jsonServerRestClient('http://jsonplaceholder.typicode.com')}>
        <Resource name="posts" list={PostList} />
    </Admin>
);

export default App;
```

### Adding Custom Headers

Both the `simpleRestClient` and the `jsonServerRestClient` functions accept an http client function as second argument. By default, they use admin-on-rest's `fetchUtils.fetchJson()` as http client. It's similar to HTML5 `fetch()`, except it handles JSON decoding and HTTP error codes automatically.

That means that if you need to add custom headers to your requests, you just need to *wrap* the `fetchJson()` call inside your own function:

```jsx
import { simpleRestClient, fetchUtils, Admin, Resource } from 'admin-on-rest';
const httpClient = (url, options = {}) => {
    if (!options.headers) {
        options.headers = new Headers({ Accept: 'application/json' });
    }
    // add your own headers here
    options.headers.set('X-Custom-Header', 'foobar');
    return fetchUtils.fetchJson(url, options);
}
const restClient = simpleRestClient('http://localhost:3000', httpClient);

render(
    <Admin restClient={restClient} title="Example Admin">
       ...
    </Admin>,
    document.getElementById('root')
);
```

Now all the requests to the REST API will contain the `X-Custom-Header: foobar` header.

**Tip**: The most common usage of custom headers is for authentication. `fetchJson` has built-on support for the `Authorization` token header:

```jsx
const httpClient = (url, options) => {
    options.user = {
        authenticated: true,
        token: 'SRTRDFVESGNJYTUKTYTHRG'
    }
    return fetchUtils.fetchJson(url, options);
}
```

Now all the requests to the REST API will contain the `Authorization: SRTRDFVESGNJYTUKTYTHRG` header.

## Decorating your REST Client (Example of File Upload)

Instead of writing your own REST client or using a third-party one, you can enhance its capabilities on a given resource. For instance, if you want to use upload components (such as `<ImageInput />` one), you can decorate it the following way:

```jsx
/**
 * Convert a `File` object returned by the upload input into
 * a base 64 string. That's easier to use on FakeRest, used on
 * the ng-admin example. But that's probably not the most optimized
 * way to do in a production database.
 */
const convertFileToBase64 = file => new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);

    reader.onload = () => resolve(reader.result);
    reader.onerror = reject;
});

/**
 * For posts update only, convert uploaded image in base 64 and attach it to
 * the `picture` sent property, with `src` and `title` attributes.
 */
const addUploadCapabilities = requestHandler => (type, resource, params) => {
    if (type === 'UPDATE' && resource === 'posts') {
        if (params.data.pictures && params.data.pictures.length) {
            // only freshly dropped pictures are instance of File
            const formerPictures = params.data.pictures.filter(p => !(p instanceof File));
            const newPictures = params.data.pictures.filter(p => p instanceof File);

            return Promise.all(newPictures.map(convertFileToBase64))
                .then(base64Pictures => base64Pictures.map(picture64 => ({
                    src: picture64,
                    title: `${params.data.title}`,
                })))
                .then(transformedNewPictures => requestHandler(type, resource, {
                    ...params,
                    data: {
                        ...params.data,
                        pictures: [...transformedNewPictures, ...formerPictures],
                    },
                }));
        }
    }

    return requestHandler(type, resource, params);
};

export default addUploadCapabilities;
```

This way, you can use simply your upload-capable client to your app calling this decorator:

```jsx
import jsonRestClient from 'aor-json-rest-client';
import addUploadFeature from './addUploadFeature';

const restClient = jsonRestClient(data, true);
const uploadCapableClient = addUploadFeature(restClient);

render(
    <Admin restClient={uploadCapableClient} title="Example Admin">
        // [...]
    </Admin>,
    document.getElementById('root'),
);

```

## Writing Your Own REST Client

Quite often, none of the the core REST clients match your API exactly. In such cases, you'll have to write your own REST client. But don't be afraid, it's easy!

### Request Format

REST requests require a *type* (e.g. `GET_ONE`), a *resource* (e.g. 'posts') and a set of *parameters*.

*Tip*: In comparison, HTTP requests require a *verb* (e.g. 'GET'), an *url* (e.g. 'http://myapi.com/posts'), a list of *headers* (like `Content-Type`) and a *body*.

Possible types are:

Type                 | Params format
-------------------- | ----------------
`GET_LIST`           | `{ pagination: { page: {int} , perPage: {int} }, sort: { field: {string}, order: {string} }, filter: {Object} }`
`GET_ONE`            | `{ id: {mixed} }`
`CREATE`             | `{ data: {Object} }`
`UPDATE`             | `{ id: {mixed}, data: {Object} }`
`DELETE`             | `{ id: {mixed} }`
`GET_MANY`           | `{ ids: {mixed[]} }`
`GET_MANY_REFERENCE` | `{ target: {string}, id: {mixed}, pagination: { page: {int} , perPage: {int} }, sort: { field: {string}, order: {string} }, filter: {Object} }`

Examples:

```jsx
restClient(GET_LIST, 'posts', {
    pagination: { page: 1, perPage: 5 },
    sort: { field: 'title', order: 'ASC' },
    filter: { author_id: 12 },
});
restClient(GET_ONE, 'posts', { id: 123 });
restClient(CREATE, 'posts', { title: "hello, world" });
restClient(UPDATE, 'posts', { id: 123, { title: "hello, world!" } });
restClient(DELETE, 'posts', { id: 123 });
restClient(GET_MANY, 'posts', { ids: [123, 124, 125] });
restClient(GET_MANY_REFERENCE, 'comments', {
    target: 'post_id',
    id: 123,
    sort: { field: 'created_at', order: 'DESC' }
});
```

### Response Format

REST responses are objects. The format depends on the type.

Type                 | Response format
-------------------- | ----------------
`GET_LIST`           | `{ data: {Record[]}, total: {int} }`
`GET_ONE`            | `{ data: {Record} }`
`CREATE`             | `{ data: {Record} }`
`UPDATE`             | `{ data: {Record} }`
`DELETE`             | `{ data: {Record} }`
`GET_MANY`           | `{ data: {Record[]} }`
`GET_MANY_REFERENCE` | `{ data: {Record[]}, total: {int} }`

A `{Record}` is an object literal with at least an `id` property, e.g. `{ id: 123, title: "hello, world" }`.

Examples:

```jsx
restClient(GET_LIST, 'posts', {
    pagination: { page: 1, perPage: 5 },
    sort: { field: 'title', order: 'ASC' },
    filter: { author_id: 12 },
})
.then(response => console.log(response));
// {
//     data: [
//         { id: 126, title: "allo?", author_id: 12 },
//         { id: 127, title: "bien le bonjour", author_id: 12 },
//         { id: 124, title: "good day sunshine", author_id: 12 },
//         { id: 123, title: "hello, world", author_id: 12 },
//         { id: 125, title: "howdy partner", author_id: 12 },
//     ],
//     total: 27
// }

restClient(GET_ONE, 'posts', { id: 123 })
.then(response => console.log(response));
// {
//     data: { id: 123, title: "hello, world" }
// }

restClient(CREATE, 'posts', { title: "hello, world" })
.then(response => console.log(response));
// {
//     data: { id: 450, title: "hello, world" }
// }

restClient(UPDATE, 'posts', { id: 123, { title: "hello, world!" } })
.then(response => console.log(response));
// {
//     data: { id: 123, title: "hello, world!" }
// }

restClient(DELETE, 'posts', { id: 123 })
.then(response => console.log(response));
// {
//     data: { id: 123, title: "hello, world" }
// }

restClient(GET_MANY, 'posts', { ids: [123, 124, 125] })
.then(response => console.log(response));
// {
//     data: [
//         { id: 123, title: "hello, world" },
//         { id: 124, title: "good day sunshise" },
//         { id: 125, title: "howdy partner" },
//     ]
// }

restClient(GET_MANY_REFERENCE, 'comments', {
    target: 'post_id',
    id: 123,
    sort: { field: 'created_at', order: 'DESC' }
});
.then(response => console.log(response));
// {
//     data: [
//         { id: 667, title: "I agree", post_id: 123 },
//         { id: 895, title: "I don't agree", post_id: 123 },
//     ],
//     total: 2,
// }
```

### Error Format

When the REST API returns an error, the rest client should `throw` an `Error` object. This object should contain a `status` property with the HTTP response code (404, 500, etc.). Admin-on-rest inspects this error code, and uses it for [authentication](./Authentication.md) (in case of 401 or 403 errors).

### Example implementation

Check the code from the [simple REST client](https://github.com/marmelab/admin-on-rest/blob/master/src/rest/simple.js): it's a good starting point for a custom rest client implementation.