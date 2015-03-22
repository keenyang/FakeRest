# FakeRest

Intercept XMLHttpRequest to fake a REST server based on JSON data. Use it on top of [Sinon.js](http://sinonjs.org/) to test JavaScript REST clients on the browser side (e.g. single page apps) without a server.

## Usage

```html
<script src="/path/to/FakeRest.min.js"></script>
<script src="/path/to/sinon.js"></script>
<script type="text/javascript">
// initialize fake REST server and data
var restServer = new FakeRest.Server();
restServer.init({
    'authors': [
        { id: 0, first_name: 'Leo', last_name: 'Tolstoi' },
        { id: 1, first_name: 'Jane', last_name: 'Austen' }
    ],
    'books': [
        { id: 0, author_id: 0, title: 'Anna Karenina' },
        { id: 1, author_id: 0, title: 'War and Peace' },
        { id: 2, author_id: 1, title: 'Pride and Prejudice' },
        { id: 3, author_id: 1, title: 'Sense and Sensibility' }
    ]
});
// use sinon.js to monkey-patch XmlHttpRequest
var server = sinon.fakeServer.create();
server.respondWith(restServer.handle.bind(restServer));

// Now query the fake REST server
var req = new XMLHttpRequest();
req.open("GET", "/authors", false);
req.send(null);
console.log(req.responseText);
// [
//    {"id":0,"first_name":"Leo","last_name":"Tolstoi"},
//    {"id":1,"first_name":"Jane","last_name":"Austen"}
// ]

var req = new XMLHttpRequest();
req.open("GET", "/books/3", false);
req.send(null);
console.log(req.responseText);
// {"id":3,"author_id":1,"title":"Sense and Sensibility"}

var req = new XMLHttpRequest();
req.open("POST", "/books", false);
req.send(JSON.stringify({ author_id: 1, title: 'Emma' }));
console.log(req.responseText);
// {"author_id":1,"title":"Emma","id":4}

// restore native XHR constructor
server.restore();
</script>
```

## Installation

FakeRest is available through npm and Bower:

```sh
# If you use Bower
bower install fakerest --save-dev
# If you use npm
npm install fakerest --save-dev
```

## REST Flavor

FakeRest uses a standard REST flavor, described below.

* `GET /foo` returns a JSON array. It accepts three query parameters: `filter`, `sort`, and `range`. It responds with a status 200 if there is no pagination, or 206 if the list of items is paginated. The reponse contains a mention of the total count in the `Content-Range` header.

```
GET /books?filter={author_id:1}&sort=['title','desc']&range=[0-9]

HTTP 1.1 200 OK
Content-Range: items 0-1/2
Content-Type: application/json
[
  { id: 3, author_id: 1, title: 'Sense and Sensibility' },
  { id: 2, author_id: 1, title: 'Pride and Prejudice' }
]
```

* `POST /foo` returns a status 201 with a `Location` header for the newly created resource, and the new resource in the body.

```
POST /books
{ author_id: 1, title: 'Emma' }

HTTP 1.1 201 Created
Location: /books/4
Content-Type: application/json
{ "author_id": 1, "title": "Emma", "id": 4 }
```

* `GET /foo/:id` returns a JSON object, and a status 200, unless the resource doesn't exist

```
GET /books/2

HTTP 1.1 200 OK
Content-Type: application/json
{ id: 2, author_id: 1, title: 'Pride and Prejudice' }
```

* `PUT /foo/:id` returns the modified JSON object, and a status 200, unless the resource doesn't exist
* `DELETE /foo/:id` returns the deleted JSON object, and a status 200, unless the resource doesn't exist

## Configuration

```js
// initialize a rest server with a custom base URL
var restServer = new FakeRest.Server('http://my.custom.domain'); 
// you can create more than one fake server to listen to several domains
var restServer2 = new FakeRest.Server('http://my.other.domain');
var server = sinon.fakeServer.create();
server.respondWith(restServer.handle.bind(restServer));
server.respondWith(restServer2.handle.bind(restServer2));
// Set all JSON data at once - only if identifier name is 'id'
restServer.init(json);
// Set data collection by collection - allows to customize the identifier name
var authorsCollection = new RestServer.Collection([], '_id');
authorsCollection.addOne({ first_name: 'Leo', last_name: 'Tolstoi' }); // { _id: 0, first_name: 'Leo', last_name: 'Tolstoi' }
authorsCollection.addOne({ first_name: 'Jane', last_name: 'Austen' }); // { _id: 1, first_name: 'Jane', last_name: 'Austen' }
// collections have autoincremented identifier but accept identifiers already set
authorsCollection.addOne({ _id: 3, first_name: 'Marcel', last_name: 'Proust' }); // { _id: 3, first_name: 'Marcel', last_name: 'Proust' }
restServer.addCollection('books', authorsCollection);
```

## Development

```sh
# Install dependencies
make install
# Watch source files and recompile dist/FakeRest.js when anything is modified
make watch
# Run tests
make test
# Build minified version
make build
```

## License

FakeRest is licensed under the [MIT Licence](LICENSE), courtesy of [marmelab](http://marmelab.com).
