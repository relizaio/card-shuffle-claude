# Mafia Card Shuffle Express Back-end

This is a back-end Node.js Express project for Mafia Card Shuffle (simple card and player order shuffle for a classic Mafia game). You can find deployed project at [https://mafia.brolia.com](https://mafia.brolia.com).

For details on deployment, please refer to [mafia deployment project](https://github.com/taleodor/mafia-deployment).

## Run locally

Prerequisites:

- Node.js 18+
- A local Redis on port 6379 (e.g. `docker run --rm -p 6379:6379 redis:7-alpine`)

Install and start:

```bash
npm install
node index.js
```

The server listens on port `3000` (hardcoded in `index.js`) and serves socket.io on path `/api`. If Redis runs elsewhere, point the app at it with the `REDIS_HOST` environment variable (defaults to `127.0.0.1`; port is always `6379`):

```bash
REDIS_HOST=my-redis-host node index.js
```


# Other
Any contributions are welcome!