Установка NodeJS на ubuntu 12.10
================================

Скачаем нод последней версии отсюда http://nodejs.org/download/
```bash
apt-get install python g++ make
mkdir ~/nodejs && cd $_
wget -N http://nodejs.org/dist/node-latest.tar.gz
tar xzvf node-latest.tar.gz && cd `ls -rd node-v*`
./configure
make install
```

Откроем порт на котором будет node
```bash
iptables -A INPUT -p tcp --dport 8124 -j ACCEPT
```

простое приложение, которое можно запустить командой ```node server.node.js```
```js
/**
 * Hello world server
 *   every request to the ip gets the same response
 */
 
// Require the http library
var http = require('http');
// Create the server
var server = http.createServer(function (request, response) {
    // Every request outputs "hello world"
    response.writeHead(200, {'Content-Type': 'text/plain'});
    response.end('Hello World\n');
    // And also logs
    console.log('Page requested at ' + new Date() + ', for ' + request.url);
});
// It listens on port 1337 and IP 127.0.0.1
server.listen(1337, "127.0.0.1");
// For the joy
console.log('Server running at <a href="http://127.0.0.1:1337/'">http://127.0.0.1:1337/'</a>);
```
