# @mdd95/adapter-node

[Adapter](https://kit.svelte.dev/docs/adapters) for SvelteKit apps that generates a standalone Node server with added support for WebSocket as plugin.

## Installation

```bash
npm install -D @mdd95/sveltekit-adapter-node
```

## Usage examples

### Using [Node.js WebSocket library](https://github.com/websockets/ws)

```ts
// plugin.ts
import { WebSocketServer } from 'ws';
import type http from 'node:http';

const wss = new WebSocketServer({ noServer: true });

wss.on('connection', (ws) => {
	ws.on('message', (data) => {
		console.log('received: %s', data);
	});
});

export default function plugin(server: http.Server) {
	server.on('upgrade', function (req, socket, head) {
		if (req.url === '/') {
			wss.handleUpgrade(req, socket, head, (ws) => {
				wss.emit('connection', ws, req);
			});
		} else {
			socket.destroy();
		}
	});
}
```

Update svelte.config.js

```js
// svelte.config.js
import adapter from '@mdd95/sveltekit-adapter-node';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter({
			pluginPath: 'plugin.ts'
		})
	}
};

export default config;
```

During development, you can create a separate server

```js
// dev.ts
import http from 'node:http';
import plugin from './plugin.ts';

const PORT = 8000;
const server = http.createServer();

plugin(server);
server.listen(PORT, () => {
	console.log(`Listening on http://localhost:${PORT}`);
});
```

Then run it using `tsx`:

```bash
npx tsx watch dev.ts
```

And run it parallel using `concurrently`

```json
// package.json
{
	"scripts": {
		"dev:vite": "vite dev",
		"dev:ws": "tsx watch dev.ts",
		"dev": "concurrently --kill-others \"npm run dev:vite\" \"npm run dev:ws\"",
		"preview": "node build/index.js"
	}
}
```

### Using [Socket.IO](https://github.com/socketio/socket.io)

```ts
// plugin.ts
import { dev } from '$app/environment';
import { Server } from 'socket.io';
import type http from 'node:http';

export default function plugin(server: http.Server) {
	const io = new Server(server, {
		cors: { origin: dev && '*' }
	});

	io.on('connection', (socket) => {
		// Handle connection
	});
}
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
	import { dev } from '$app/environment';
	import { io } from 'socket.io-client';

	$effect(() => {
		const socket = io(dev && 'ws://localhost:8000');
	});
</script>
```

## Changelog

[The Changelog for this package is available on GitHub](CHANGELOG.md).

## License

[MIT](LICENSE)
