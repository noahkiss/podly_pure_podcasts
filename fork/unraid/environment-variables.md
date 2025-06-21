# Podly Environment Variables

## Backend Container Variables

### Required Variables
- `PUID`: User ID for file permissions (default: 99 for Unraid nobody user)
- `PGID`: Group ID for file permissions (default: 100 for Unraid nobody user)

### Optional Variables
- `CUDA_VISIBLE_DEVICES`: Which GPU to use (default: 0, use -1 for CPU only)
- `CORS_ORIGINS`: CORS origins for web requests (default: "*")

### Configuration
The backend requires a `config/config.yml` file. Copy from `config.yml.example` and configure:
- OpenAI API key
- Server URL
- Other application settings

## Frontend Container Variables

### Required Variables
- `VITE_API_URL`: Backend API URL (default: http://localhost:5002, custom: http://localhost:7638)

### Port Configuration
The frontend can be configured to connect to any backend port by setting `VITE_API_URL`:
- **Default**: `http://localhost:5002`
- **Custom ports**: `http://localhost:YOUR_BACKEND_PORT`
- **Example**: `http://localhost:7638` for backend on port 7638

## Volume Mounts

### Backend Required Volumes
- `/app/config`: Configuration directory
- `/app/in`: Input files directory
- `/app/srv`: Server files directory
- `/app/src`: Source code directory
- `/app/scripts`: Scripts directory

### Frontend
- No persistent volumes required

## Ports

### Backend
- **Internal**: `5002` (container port)
- **External**: `7638` (host port, customizable)

### Frontend
- **Internal**: `80` (container port)
- **External**: `7637` (host port, customizable)

## GPU Requirements

The backend container requires:
- NVIDIA GPU with CUDA support
- nvidia-docker runtime
- CUDA 12.4.1 compatible drivers

## Unraid-Specific Notes

Unraid defaults to the `nobody` user (UID 99, GID 100). The container will run as this user to ensure proper file permissions for Unraid's file system.

## Custom Port Configuration

### Using Custom Ports
1. **Backend**: Map container port 5002 to any host port (e.g., 7638)
2. **Frontend**: Map container port 80 to any host port (e.g., 7637)
3. **Frontend Configuration**: Set `VITE_API_URL` to point to your backend port

### Example with Custom Ports
```bash
# Backend on port 7638
docker run -d --name=podly_backend -p 7638:5002 ...

# Frontend on port 7637, connecting to backend on 7638
docker run -d --name=podly_frontend -p 7637:80 -e VITE_API_URL=http://localhost:7638 ...
```

## Example Docker Run Commands

See `docker-run-command.txt` for complete examples.
