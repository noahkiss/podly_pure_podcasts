# Podly Fork - Docker Setup

This fork provides automated Docker image building and upstream syncing for the Podly podcast ad-blocker.

## Features

- 🐳 **Automated Docker Builds**: GitHub Actions build and publish Docker images to GHCR
- 🔄 **Upstream Sync**: Automated workflow to sync with upstream changes
- 🖥️ **Unraid Ready**: Docker templates and run commands for Unraid servers
- 🎯 **GPU Optimized**: Pre-built GPU-enabled images for NVIDIA GPUs
- 🔧 **Custom Port Support**: Configurable ports with frontend-backend communication

## Docker Images

Images are published to: `ghcr.io/noahkiss/podly_pure_podcasts/`

- `podly-backend:latest` - Backend API with GPU support
- `podly-frontend:latest` - Frontend web interface

## GitHub Workflows

### 1. Docker Build Workflow (`.github/workflows/docker-build.yml`)

**Triggers:**
- Push to `main` branch
- Push of version tags (`v*`)
- Manual dispatch

**Actions:**
- Builds GPU-enabled backend image
- Builds frontend image
- Publishes to GHCR with `latest` and commit SHA tags
- Caches layers for faster builds

### 2. Upstream Sync Workflow (`.github/workflows/sync-upstream.yml`)

**Triggers:**
- Scheduled: Mondays and Thursdays at 7 AM
- Manual dispatch with test mode option

**Actions:**
- Syncs changes from `jdrbc/podly_pure_podcasts`
- Handles conflicts automatically
- Reports sync status

## Usage

### Using Pre-built Images

#### Docker Compose (Custom Ports)
```bash
# Use fork-specific compose file with custom ports (7637/7638)
docker compose -f fork/compose.fork.yml up -d
```

#### Manual Docker Run
```bash
# Backend (Linux/standard - default ports)
docker run -d \
  --name=podly_backend \
  --runtime=nvidia \
  --gpus all \
  -e PUID=1000 \
  -e PGID=1000 \
  -p 5002:5002 \
  -v /path/to/config:/app/config \
  -v /path/to/in:/app/in \
  -v /path/to/srv:/app/srv \
  -v /path/to/src:/app/src \
  -v /path/to/scripts:/app/scripts \
  ghcr.io/noahkiss/podly_pure_podcasts/podly-backend:latest

# Frontend (Linux/standard - default ports)
docker run -d \
  --name=podly_frontend \
  -p 5001:80 \
  ghcr.io/noahkiss/podly_pure_podcasts/podly-frontend:latest

# Backend (Unraid - custom ports 7638)
docker run -d \
  --name=podly_backend \
  --runtime=nvidia \
  --gpus all \
  -e PUID=99 \
  -e PGID=100 \
  -p 7638:5002 \
  -v /mnt/user/appdata/podly/config:/app/config \
  -v /mnt/user/appdata/podly/in:/app/in \
  -v /mnt/user/appdata/podly/srv:/app/srv \
  -v /mnt/user/appdata/podly/src:/app/src \
  -v /mnt/user/appdata/podly/scripts:/app/scripts \
  ghcr.io/noahkiss/podly_pure_podcasts/podly-backend:latest

# Frontend (Unraid - custom ports 7637, connects to backend on 7638)
docker run -d \
  --name=podly_frontend \
  -e VITE_API_URL=http://localhost:7638 \
  -p 7637:80 \
  ghcr.io/noahkiss/podly_pure_podcasts/podly-frontend:latest
```

### Custom Port Configuration

The frontend can connect to any backend port by setting the `VITE_API_URL` environment variable:

```bash
# Example: Backend on port 9000, frontend on port 9001
docker run -d --name=podly_backend -p 9000:5002 ...
docker run -d --name=podly_frontend -p 9001:80 -e VITE_API_URL=http://localhost:9000 ...
```

### Unraid Setup

1. **Install Community Applications plugin** (if not already installed)
2. **Add Docker templates**:
   - Copy `fork/unraid/unraid-template.xml` (backend) to your Unraid server
   - Copy `fork/unraid/unraid-template-frontend.xml` (frontend) to your Unraid server
   - Add via Docker tab → Add Container → Template
3. **Configure paths and ports** as needed
4. **Start containers**

See `fork/unraid/` directory for detailed Unraid setup files.

## Configuration

### Required Setup

1. **Copy configuration template**:
   ```bash
   cp config/config.yml.example config/config.yml
   ```

2. **Add OpenAI API key** to `config/config.yml`

3. **Create required directories**:
   ```bash
   mkdir -p config in srv scripts
   ```

### Environment Variables

See `fork/unraid/environment-variables.md` for complete documentation.

## Development

### Local Development
```bash
# Use original development setup
./run_podly_docker.sh --dev
```

### Building Locally
```bash
# Build with GPU support
./run_podly_docker.sh --gpu --build

# Build with CPU only
./run_podly_docker.sh --cpu --build
```

## Troubleshooting

### GPU Issues
- Ensure NVIDIA drivers are installed
- Verify nvidia-docker runtime is available
- Check `nvidia-smi` output

### Permission Issues
- Set correct `PUID` and `PGID` for your user
  - Linux standard: 1000/1000
  - Unraid default: 99/100 (nobody user)
- Ensure volume directories have correct permissions

### Network Issues
- Verify ports are available (default: 5001/5002, custom: 7637/7638)
- Check firewall settings
- Ensure containers can communicate
- Verify `VITE_API_URL` points to correct backend port

### Frontend-Backend Communication
- Frontend uses `VITE_API_URL` environment variable to connect to backend
- Default: `http://localhost:5002`
- Custom: `http://localhost:YOUR_BACKEND_PORT`
- Check browser console for connection errors

## Upstream Sync

### Manual Sync
1. Go to Actions tab in GitHub
2. Select "Sync with Upstream" workflow
3. Click "Run workflow"
4. Optionally enable test mode first

### Automatic Sync
- Runs automatically on Mondays and Thursdays at 7 AM
- Creates commits for upstream changes
- Maintains fork compatibility

## Contributing

This fork maintains compatibility with upstream by:
- Keeping all fork-specific files in `fork/` directory
- Using separate workflow files
- Not modifying upstream source files

To contribute:
1. Make changes in fork-specific directories
2. Test with local builds
3. Push to trigger automated builds
4. Create PR if needed

## License

Same as upstream repository. See `LICENCE` file for details.
