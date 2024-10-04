# sveltekit-adapter-node-with-websocket

[Adapter](https://kit.svelte.dev/docs/adapters) for SvelteKit apps that generates a standalone Node server with added support for WebSocket.

## Installation

```bash
npm install -D sveltekit-adapter-node-with-websocket
```

## Usage examples

Create websocket.js in `src` folder

```js
// src/websocket.js
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ noServer: true });

wss.on('connection', (ws) => {
	console.log('Client connected');

	// Listen to connection
});

/**
 *
 * @param {import('http').IncomingMessage} req
 * @param {import('stream').Duplex} socket
 * @param {Buffer} head
 */
export function upgrade(req, socket, head) {
	if (req.url === '/') {
		wss.handleUpgrade(req, socket, head, (ws) => {
			wss.emit('connection', ws, req);
		});
	}
}
```

Update svelte.config.js

```js
// svelte.config.js
import adapter from 'sveltekit-adapter-node-with-websocket';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter({ serverUpgrade: 'websocket' })
	}
};

export default config;
```

During development, you can create a separate server

```js
// dev.js
import http from 'node:http';
import { upgrade } from './src/websocket.js';

const PORT = 8000;

const server = http.createServer();

server.on('upgrade', upgrade);
server.listen(PORT, () => {
	console.log(`Listening on http://localhost:${PORT}`);
});
```

Then run it using `node` or `nodemon`:

```bash
nodemon dev.js
```

## Changelog

[The Changelog for this package is available on GitHub](https://github.com/sveltejs/kit/blob/main/packages/adapter-node/CHANGELOG.md).

## License

[MIT](LICENSE)
