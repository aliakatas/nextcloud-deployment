# Nextcloud deployment
Following the guide from [pimylifeup](https://pimylifeup.com/nextcloud-docker/), this repo stores the configuration for hosting Nextcloud on a homelab.

Another useful resource is found on [Medium](https://chrisgrime.medium.com/deploy-nextcloud-with-docker-compose-935a76a5eb78). 
However, the implementation in this repo is not based on this article.

## Additional considerations
Before deploying, make sure to create a `.env` file at the same directory as the `docker-compose.yml` file and set the following variables:
- `SQLPASS`: The password generated in one of the steps of the guide. Used by the database.
- `EMAIL`: An email used if anything goes wrong with the SSL certificate.
- `HOSTNAME`: The public name of the homelab/network. Remember to forward ports 80 and 443 from the router.
- `DATA`: The absolute path on the machine hosting the deployment. This is where all the data of the deployment will be saved.

Although not mentioned explicitly, before running `docker compose up -d`, you must run `docker compose build` to make sure the proxy container is built and ready to go.

## Create an autostart service
1. **Create a Systemd Service File**: Systemd is used to manage services in most modern Linux distributions. You'll create a service file to manage the Docker Compose stack.
   1. Open a terminal and create a new service file:
   ```bash
   sudo nano /etc/systemd/system/docker-compose-app.service
   ```
   2. Add the following content to the file, customizing paths as needed:
   ```ini
   [Unit]
   Description=Docker Compose Application Service
   Requires=docker.service
   After=docker.service

   [Service]
   Type=oneshot
   RemainAfterExit=true
   WorkingDirectory=/path/to/your/docker-compose
   ExecStart=/usr/bin/docker-compose up -d
   ExecStop=/usr/bin/docker-compose down

   [Install]
   WantedBy=multi-user.target
   ```

   - Replace `/path/to/your/docker-compose` with the directory containing [docker-compose.yml](./docker-compose.yml).
   - Ensure the paths to `docker-compose` are correct (e.g., `/usr/local/bin/docker compose` on some setups).
   - Alternatively, edit [nextcloud-home.service](./nextcloud-home.service) and copy to `/etc/systemd/system`.
2. **Reload Systemd and Enable the Service**: 
   1. Reload the systemd daemon to recognize the new service:
   ```bash
   sudo systemctl daemon-reload
   ```
   2. Enable the service to start at boot:
   ```bash
   sudo systemctl enable docker-compose-app.service
   ```
3. **Start the Service**: Start the service manually to verify it works:
   ```bash
   sudo systemctl start docker-compose-app.service
   ```

   Check its status:
   ```bash
   sudo systemctl status docker-compose-app.service
   ```
4. **Verify at Startup**: Reboot your machine to test if the service starts automatically:
   ```bash
   sudo reboot
   ```
   After reboot, check if your containers are running:
   ```bash
   docker ps
   ```
**Additional Notes**
---
1. **Logs**: To troubleshoot, check the service logs:
   ```bash
   journalctl -u docker-compose-app.service
   ```

2. **Environment Variables**: If `docker-compose.yml` relies on environment variables, ensure they are accessible in the service's environment. You can add:
   ```ini
   EnvironmentFile=/path/to/env/file
   ```
   to the `[Service]` section.

   