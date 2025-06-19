# Serving Geospatial Data with GeoServer, Docker, and Nginx 🚀

Excited to share that I've successfully set up **GeoServer** using **Docker** and **Nginx** as a reverse proxy to serve geospatial data efficiently! This setup is scalable, secure, and perfect for delivering geospatial services to clients. Below, I'm sharing the configuration details to help others set this up.

## Why This Setup?

- **GeoServer**: A robust open-source server for sharing geospatial data.
- **Docker**: Simplifies deployment and ensures consistency across environments.
- **Nginx**: Acts as a reverse proxy for load balancing and added security.

## Configurations

### Docker Compose (`docker-compose.yml`)

This sets up GeoServer in a Docker container with persistent data and customizable settings.

```yaml
version: "3.8"

services:
  geoserver:
    container_name: geoserver
    image: docker.osgeo.org/geoserver:2.26.2
    # Optional: Uncomment to run as a specific user (replace <UID> and <GID> with your user/group IDs)
    # user: "<UID>:<GID>"
    volumes:
      # Mount a local directory for GeoTIFF or other raster files (replace /path/to/tiff with your directory)
      - /path/to/tiff:/opt/geoserver_data
      # Persistent volume for GeoServer configuration and data
      - geoserver_data:/opt/geoserver/data_dir
      # Optional: Uncomment to use a custom web.xml for GeoServer (replace /path/to/web.xml with your file)
      # - /path/to/web.xml:/usr/local/tomcat/webapps/geoserver/WEB-INF/web.xml
    ports:
      # Exposes GeoServer on port 8080; adjust if needed
      - "8080:8080"
    environment:
      # Required: Set admin credentials (change these for security)
      - GEOSERVER_ADMIN_USER=admin
      - GEOSERVER_ADMIN_PASSWORD=geoserver
      # Optional: Disable CSRF for testing (set to true for development, false for production)
      - GEOSERVER_CSRF_DISABLED=true
      # Optional: Configure CORS for cross-origin requests (e.g., for web clients; can be handled by Nginx)
      - CORS_ALLOWED_ORIGINS=*
      # Optional: Skip demo data to reduce startup time
      - SKIP_DEMO_DATA=true
      # Optional: Uncomment to set a proxy base URL if behind a reverse proxy like Nginx
      # - PROXY_BASE_URL=https://yourdomain.com/geoserver
      # Optional: Uncomment to customize GeoServer log location
      # - GEOSERVER_LOG_LOCATION=/opt/geoserver/data_dir/logs/geoserver.log
    networks:
      # Connect to a Docker network (see networks section below)
      - geoserver-network
    restart: unless-stopped

networks:
  # Define a network for GeoServer and Nginx (or other services)
  geoserver-network:
    name: geoserver-network
    # Optional: Set to external if using an existing network
    # external: true
    # name: your-existing-network

volumes:
  # Persistent volume for GeoServer data to retain configurations
  geoserver_data:
```

Run with: `docker-compose up -d`

### Nginx Configuration (`geoserver.conf`)

This configures Nginx as a reverse proxy to forward requests to GeoServer. For production, update to `listen 443 ssl` with an SSL certificate.

```nginx
server {
    listen 80;
    server_name geoserver.yourdomain.com;

    location /geoserver {
        proxy_pass http://geoserver:8080/geoserver;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Enable CORS for GeoServer (sufficient for most cases; GeoServer CORS setting is optional)
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range' always;

        # Handle preflight OPTIONS requests
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range' always;
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }
    }

    # Optional: Enable caching for static resources
    location ~* \.(png|jpg|jpeg|gif|css|js|woff|woff2|ttf|svg|eot)$ {
        expires 1y;
        access_log off;
        add_header Cache-Control "public";
    }
}
```

## Usage Instructions

1. **Customize Paths**:
   - Replace `/path/to/tiff` with your GeoTIFF or geospatial data directory.
   - If using a custom `web.xml` (e.g., for CORS or HTTPS), uncomment the volume mapping and specify your `web.xml` path.

2. **Set Environment Variables**:
   - Update `GEOSERVER_ADMIN_USER` and `GEOSERVER_ADMIN_PASSWORD` with secure credentials.
   - Adjust `CORS_ALLOWED_ORIGINS` to specific domains (e.g., `http://yourfrontend.com`) instead of `*` for production.
   - If behind a reverse proxy, set `PROXY_BASE_URL` to your domain (e.g., `https://geoserver.yourdomain.com/geoserver`).

3. **Network Configuration**:
   - The default network is `geoserver-network`. For an existing network (e.g., for Nginx), uncomment the `external` option and set `name` to your network.

4. **Deploy the Setup**:
   - **Install Docker and Nginx**: Ensure Docker, Docker Compose, and Nginx are installed.
   - **Run GeoServer**: Save `docker-compose.yml` and run `docker-compose up -d`.
   - **Configure Nginx**: Place `geoserver.conf` in `/etc/nginx/conf.d/`, update `server_name`, and reload Nginx with `sudo nginx -s reload`.
   - **Test**: Access GeoServer at `http://geoserver.yourdomain.com/geoserver` and log in with your credentials.

## Tips

- **Security**: Update default credentials and enable HTTPS with Let's Encrypt for production.
- **CORS**: Nginx CORS settings are typically sufficient; restrict `Access-Control-Allow-Origin` in production.
- **Performance**: Tune Nginx caching and GeoServer memory settings for high traffic.
- **Monitoring**: Use Prometheus or Grafana to monitor container health.

This setup has been a game-changer for serving geospatial data reliably. Try it out, and share your feedback or enhancements! 🌍

## For More Queries
I'm happy to help! Contact me at **django.infusion@gmail.com** or WhatsApp at **+92 303 7116786**.

#Geospatial #GeoServer #Docker #Nginx #GIS #DevOps
