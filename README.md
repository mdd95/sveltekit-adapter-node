# sveltekit-adapter-node-with-websocket

[Adapter](https://kit.svelte.dev/docs/adapters) for SvelteKit apps that generates a standalone Node server with added support for WebSocket.

## Installation

```bash
npm install -D sveltekit-adapter-node-with-websocket
```

## Usage examples

```ts
// websocket.ts
import { WebSocketServer } from 'ws';
import type http from 'node:http';
import type internal from 'node:stream';

const wss = new WebSocketServer({ noServer: true });

wss.on('connection', (ws) => {
	console.log('Client connected');

	// Listen to connection
});

export function upgrade(req: http.IncomingMessage, socket: internal.Duplex, head: Buffer) {
	if (req.url === '/') {
		wss.handleUpgrade(req, socket, head, function connection(ws) {
			wss.emit('connection', ws, req);
		});
	} else {
		socket.destroy();
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
		adapter: adapter({
			// path
			serverUpgrade: 'websocket.ts'
		})
	}
};

export default config;
```

During development, you can create a separate server

```js
// dev.ts
import http from 'node:http';
import { upgrade } from './websocket.ts';

const PORT = 8000;

const server = http.createServer();

server.on('upgrade', upgrade);
server.listen(PORT, () => {
	console.log(`Listening on http://localhost:${PORT}`);
});
```

Then run it using `tsx`:

```bash
npx tsx watch dev.ts
```

## Changelog

[The Changelog for this package is available on GitHub](CHANGELOG).

## License

[MIT](LICENSE)
