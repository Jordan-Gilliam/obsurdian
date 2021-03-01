# run a web server

Make a web server.js:

``` js
var http = require('http');
var server = http.createServer(function (req, res) {
    res.end("YOU'RE A WIZARD.\n");
});
server.listen(8000);
```

now run your server with node from inside a screen:

```bash
$ node server.js
```

then detach the screen with CTRL+A d.
