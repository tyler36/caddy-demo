# Caddy HTTPS Static Site Server

This project serves static files from the `./site` directory over HTTPS using Caddy web server in Docker.

## Prerequisites

- Docker and Docker Compose installed
- A web browser for testing

## Project Structure

```tree
.
├── docker-compose.yml    # Docker Compose configuration
├── Caddyfile            # Caddy web server configuration
├── site/                # Static files directory
│   └── index.html       # Your web content
└── README.md           # This file
```

## Setup Instructions

### 1. Clone or Create the Project

Ensure you have the following files in your project directory:

- `docker-compose.yml`
- `Caddyfile`
- `site/index.html` (or your static content)

### 2. Start the Server

Run the following command to start the Caddy server:

```bash
docker compose up
```

Or to run in the background:

```bash
docker compose up -d
```

### 3. Access Your Site

Once the container is running, you can access your site at:

- **HTTPS**: <https://localhost>
- **HTTP**: <http://localhost> (will redirect to HTTPS)

## SSL Certificate Setup

Caddy automatically generates a self-signed certificate for local development. Your browser will show a security warning the first time you visit.

### Adding Certificate to Chrome (Optional)

To avoid browser warnings, you can add Caddy's root CA certificate to Chrome:

1. Extract the certificate from the container:

   ```bash
   docker compose exec caddy cat /data/caddy/pki/authorities/local/root.crt > caddy-root-ca.crt
   ```

2. Install in Chrome:
   - Go to `chrome://settings/certificates`
   - Click the **"Authorities"** tab
   - Click **"Import"**
   - Select the `caddy-root-ca.crt` file
   - Check **"Trust this certificate for identifying websites"**
   - Click **"OK"**

3. Restart Chrome and visit <https://localhost>

## Configuration Files

### docker-compose.yml

```yaml
services:
  caddy:
    image: caddy:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./site:/usr/share/caddy:ro
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
```

### Caddyfile

```caddy
# Serve static files from /usr/share/caddy (mapped from ./site)
localhost {
  root * /usr/share/caddy
  file_server
  tls internal
}
```

## Common Commands

### Start the server

```bash
docker compose up
```

### Start in background

```bash
docker compose up -d
```

### Stop the server

```bash
docker compose down
```

### Restart the server (after config changes)

```bash
docker compose restart
```

### View logs

```bash
docker compose logs caddy
```

### Follow logs in real-time

```bash
docker compose logs -f caddy
```

## Testing and Debugging

### Basic Connectivity Test

Test if the server is responding:

```bash
# Test HTTPS (ignore certificate warnings)
curl -k https://localhost

# Test HTTP (should redirect to HTTPS)
curl -I http://localhost
```

### Check Container Status

Verify the container is running:

```bash
docker compose ps
```

Expected output:

```
NAME            IMAGE          COMMAND                  SERVICE   CREATED         STATUS          PORTS
caddy-caddy-1   caddy:latest   "caddy run --config …"   caddy     X minutes ago   Up X minutes    0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
```

### View Container Logs

Check for errors or issues:

```bash
docker compose logs caddy
```

Look for:

- ✅ `"server running"` messages
- ✅ `"certificate installed properly"`
- ❌ Any error messages or warnings

### Test File Serving

Verify specific files are being served:

```bash
# Test main page
curl -k https://localhost

# Test specific file (if it exists)
curl -k https://localhost/style.css
```

### Common Issues and Solutions

#### 1. "Connection refused" error

- **Cause**: Container not running
- **Solution**: Run `docker compose up`

#### 2. "Invalid response" error

- **Cause**: Wrong URL or port
- **Solution**: Use `https://localhost` (not `https://localhost:443`)

#### 3. Certificate warnings in browser

- **Cause**: Self-signed certificate
- **Solution**: Either ignore the warning or install the CA certificate (see SSL Certificate Setup)

#### 4. Files not updating

- **Cause**: Browser cache
- **Solution**: Hard refresh (`Ctrl+F5` or `Cmd+Shift+R`)

#### 5. Permission errors

- **Cause**: File permissions on mounted volumes
- **Solution**: Check that files in `./site` are readable:

  ```bash
  \ls --color=auto -la site/
  ```

### Configuration Validation

Test Caddyfile syntax without starting the server:

```bash
docker run --rm -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile caddy:latest caddy validate --config /etc/caddy/Caddyfile
```

### Port Conflicts

If ports 80 or 443 are already in use:

```bash
# Check what's using the ports
sudo netstat -tlnp | \grep --color=auto ':80\|:443'

# Or use different ports in docker-compose.yml
ports:
  - "8080:80"
  - "8443:443"
```

Then access via: <https://localhost:8443>

## Development Tips

1. **Auto-reload**: Caddy automatically serves new files added to the `./site` directory
2. **Log monitoring**: Use `docker compose logs -f caddy` to watch for issues
3. **Configuration changes**: Restart with `docker compose restart` after modifying `Caddyfile`
4. **Clean start**: Use `docker compose down && docker compose up` for a fresh start

## Stopping the Server

To stop the server:

```bash
docker compose down
```

This will stop and remove the container, but preserve your files and configuration.
