# sorta

Preserve the ordering of html elements in the browser as updates stream in.

# examples

## voting all in browser

[See this example live](http://substack.net/projects/sorta-vote/).

``` js
var sorta = require('sorta');
var $ = require('jquery-browserify');

var s = sorta(function (row) {
    var name = $('<span>').attr('class', 'name').text(row.key);
    var rank = $('<span>').attr('class', 'rank');
    var score = $('<span>').attr('class', 'score');
    
    var up = $('<input>')
        .attr('type', 'button')
        .val('+')
        .click(function () { row.update(row.value + 1) })
    ;
    
    var down = $('<input>')
        .attr('type', 'button')
        .val('-')
        .click(function () { row.update(Math.max(0, row.value - 1)) })
    ;
    
    row.on('update', function () {
        rank.text(row.index + 1);
        score.text(row.value);
    });
    
    return $('<div>').append(
        '#', rank, ' ', name, ', ', score, ' points', up, down
    )[0];
});
s.appendTo(document.body);

s.write({ key : 'robots', value : 0 });
s.write({ key : 'dinosaurs', value : 0 });
s.write({ key : 'insects', value : 0 });
s.write({ key : 'electromagnets', value : 0 });
```

## streaming events from the server

http server to spam updates at the browser:

``` js
var http = require('http');
var ecstatic = require('ecstatic')(__dirname + '/static');
var server = http.createServer(ecstatic);
server.listen(8002);

var JSONStream = require('JSONStream');
var shoe = require('shoe');

var sock = shoe(function (stream) {
    var str = JSONStream.stringify();
    str.pipe(stream);
    
    var rows = { x : 0, y : 0, z : 0, a : 0, b : 0, c : 0 };
    
    Object.keys(rows).forEach(function (key) {
        str.write({ key : key, value : rows[key] });
    });
    
    var iv = setInterval(function () {
        var keys = Object.keys(rows);
        var key = keys[Math.floor(Math.random() * keys.length)];
        rows[key] += 5;
        str.write({ key : key, value : rows[key] });
    }, 100);
    
    var onend = function () { clearInterval(iv) };
    stream.on('end', onend);
    stream.on('error', onend);
});
sock.install(server, '/updates');
```

browser code:

``` js
var sorta = require('sorta');
var $ = require('jquery-browserify');

var JSONStream = require('JSONStream');
var parser = JSONStream.parse([ true ]);

var shoe = require('shoe');
var stream = shoe('/updates');
stream.pipe(parser);

var s = sorta(function (row) {
    var name = $('<span>').attr('class', 'name').text(row.key);
    var rank = $('<span>').attr('class', 'rank');
    var score = $('<span>').attr('class', 'score');
    
    row.on('update', function () {
        rank.text(row.index + 1);
        score.text(row.value);
    });
    
    return $('<div>').append('#', rank, ' ', name, ', ', score, ' points')[0];
});
parser.pipe(s);
s.appendTo(document.body);
```

# methods

``` js
var sorta = require('sorta')
```

## var s = sorta(createElement)

Return a new writable stream. Incoming writes must be row objects with `'key'`
and `'value'` properties.

The `createElement(row)` function is called every time a new key shows up.
`createElement(row)` should return a new html dom element for the `row` object.

## s.write(row)

Create or update the row at `row.key` with the new value from `row.value`.

If `row.value === undefined`, the element named by `row.key` will be removed.

## s.appendTo(target)

Append the `s.element` html element to the `target` element.

## row.update(value)

Set a new value for the row object explicitly. This is the same as
`s.write({ key : row.key, value : row.value })`.

# events

## s.on('update', function (row) { ... })

Triggered when a row gets updated or created.

# attributes

## sorta.element

html dom element that contains all the rows

# install

With [npm](http://npmjs.org) do:

```
npm install sorta
```

This module is intended to be used with a node-style module system.
Use [browserify](http://github.com/substack/node-browserify) or similar.

# license

MIT
