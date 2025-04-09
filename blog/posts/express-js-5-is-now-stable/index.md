I just discovered that, after years of development, [Express.js 5](https://expressjs.com) has finally been [released as stable](https://expressjs.com/2025/03/31/v5-1-latest-release.html). It’s been in development for so long that I’ve forgotten how long it’s been. The reason this is newsworthy for me though is that it used to be my go-to server for Node development. However, because development seemed to have stalled years ago, I switched over to [Fastify](https://fastify.dev) not only for my personal projects, but also for my professional projects.

The catalyst was support for [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2). About five years ago, I had to integrate support for HTTP/2 support into our backend because Google Cloud Run only allows file uploads over 32 MB if your service uses HTTP/2. Otherwise, it’s restricted. Since Express.js didn’t support HTTP/2 and the recommended way of adding it with the deprecated [spdy package](https://www.npmjs.com/package/spdy) doesn’t work with modern versions of Node, we had to move away from Express.js.

I haven’t been able to find whether Express.js 5 actually supports HTTP/2 or not, but considering the fact that there doesn’t seem to be any documentation, I’d wager it doesn’t. I did, however, ask A.I. if you could integrate it into Node’s [http2 package](https://nodejs.org/api/http2.html) and this is the code it suggested:

```javascript
import express from 'express';
import http2 from 'http2';
import fs from 'fs';

const app = express();

// Example middleware
app.get('/', (req, res) => {
  res.send('Hello, HTTP/2!');
});

// Load SSL/TLS certificates for HTTP/2
const options = {
  key: fs.readFileSync('path/to/your/private-key.pem'),
  cert: fs.readFileSync('path/to/your/certificate.pem')
};

// Create an HTTP/2 server
const server = http2.createSecureServer(options, app);

// Start the server
server.listen(3000, () => {
  console.log('HTTP/2 server running on https://localhost:3000');
});
```

I haven’t tested the code, so I don’t know if it works but if it does, it would be an even better way to use it than Fastify’s in my opinion as it would mean the Express.js project wouldn’t have to reinvent the wheel and could just piggyback off of Node’s standard library. That would make a lot of sense.

If anyone is interested in trying it out, definitely let me know what the results are. I may test it in the future and if I do, I’ll write another post here about it.