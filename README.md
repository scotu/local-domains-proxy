# Local HTTPS Proxy Setup

This setup provides a local HTTPS reverse proxy using NGINX in Docker, allowing you to serve multiple local services over HTTPS (locally to this computer) using custom (local) domain names. This is useful for multiple reasons: 

- Have multiple local development environments using a more mnemonic domain name than remembering port numbers
- Frontend development is simplified by not having to deal with 
   - https restricted browser features such as Service Workers, getUserMedia (Webcam and Microphone)
- Running local services without 

## Prerequisites

- Docker and Docker Compose
- mkcert (for generating local SSL certificates)
- Admin access to modify your hosts file

## Initial Setup

1. Install mkcert and generate certificates:
   ```bash
   # Install mkcert (varies by OS)
   # macOS:
   brew install mkcert
   brew install nss # if using firefox
   # Ubuntu/Debian:
   sudo apt install mkcert
   
   # Install local CA
   mkcert -install
   
   # Generate certificate for each domain configured (you can also credate wildcard domains if applicable)
   mkcert "chosen.domain.local" 127.0.0.1 ::1
   
2. Create a `certs` directory and move all your certificates there:
   mv chosen.domain.local.pem certs/
   mv chosen.domain.local-key.pem certs/

3. Add your domains to your hosts file:
   - On Linux/MacOS: Edit `/etc/hosts` (you probably need admin rights/sudo)
   - On Windows: Edit `C:\Windows\System32\drivers\etc\hosts`
   
   Add a line like the one below for each domain already configured in nginx.conf:
   ```
   127.0.0.1 chosen.domain.local
   ```

4. Once all services are added you can start the proxy:
   ```bash
   docker compose up -d
   ```

## Adding a New Service

1. Create certificate if necessary (see mkcert above)

2. Edit `nginx.conf` and add a new server block:
   ```nginx
   server {
       listen 443 ssl;
       server_name your-service.local;

       ssl_certificate /etc/nginx/certs/chosen.domain.local.pem;
       ssl_certificate_key /etc/nginx/certs/chosen.domain.local-key.pem;

       location / {
           proxy_pass http://host.docker.internal:YOUR_PORT/;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

   - Reminder that at `proxy_pass` host.docker.internal refers to the local machine where docker is running, not the container, which is why nginx uses it to refer to the host machine from inside the container.

3. Add the new domain to your hosts file:
   ```
   127.0.0.1 your-service.local
   ```

4. Restart the proxy:
   ```bash
   docker compose restart
   ```

## Troubleshooting

- **Certificate Issues**: Make sure your certificates are properly generated and placed in the `certs` directory
- **Connection Refused**: Verify that your target service is running on the specified port
- **Name Resolution**: Check that your domain is properly added to the hosts file
