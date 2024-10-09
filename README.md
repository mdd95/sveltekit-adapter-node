# @mdd95/adapter-node

[Adapter](https://kit.svelte.dev/docs/adapters) for SvelteKit apps that generates a standalone Node server with added support for WebSocket.

## Installation

```bash
npm install -D @mdd95/sveltekit-adapter-node
```

## Usage examples

### Using [Node.js WebSocket library](https://github.com/websockets/ws)

```ts
// src/websocket.ts
import { WebSocketServer } from 'ws';
import type http from 'node:http';

const wss = new WebSocketServer({ noServer: true });

wss.on('connection', (ws) => {
	ws.on('error', console.error);

	ws.on('message', (data) => {
		console.log('received: %s', data);
	});
});

export default function plugin(server: http.Server) {
	server.on('upgrade', function (req, socket, head) {
		if (req.url === '/ws') {
			wss.handleUpgrade(req, socket, head, (ws) => {
				wss.emit('connection', ws, req);
			});
		}
	});
}
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
	$effect(() => {
		const socket = new WebSocket('ws://localhost:5173/ws')

		return () => {
			socket.close();
		}
	});
</script>
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
			pluginPath: './src/websocket.ts'
		})
	}
};

export default config;
```

### Using [Socket.IO](https://github.com/socketio/socket.io)

```ts
// src/websocket.ts
import { Server } from 'socket.io';
import type http from 'node:http';

export default function plugin(server: http.Server) {
	const io = new Server(server);

	io.on('connection', (socket) => {
		// Handle connection
	});
}
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
	import { io } from 'socket.io-client';

	$effect(() => {
		const socket = io();

		return () => {
			socket.disconnect();
		}
	});
</script>
```

### Development

During development, you can create a vite plugin

```ts
// vite.config.ts
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig, type Plugin } from 'vite';
import plugin from './src/websocket';

function websocket(): Plugin {
	return {
		name: 'websocket',
		configureServer(server) {
			if (server.httpServer) plugin(server.httpServer);
		},
		configurePreviewServer(server) {
			if (server.httpServer) plugin(server.httpServer);
		}
	};
}

export default defineConfig({
	plugins: [sveltekit(), websocket()]
});
```

## Changelog

[The Changelog for this package is available on GitHub](CHANGELOG.md).

## License

[MIT](LICENSE)
