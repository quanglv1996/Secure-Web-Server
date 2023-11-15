
# Secure Web Server With Docker

This Docker Compose project sets up a web server using Nginx and includes a Certbot service for managing SSL certificates. The web server serves content from the `public_html` directory and supports HTTPS.

![Docker Logo](https://www.docker.com/sites/default/files/d8/styles/role_icon/public/2019-07/Docker-Logo-White-RGB_Vertical.png)

## Prerequisites

Before you get started, ensure that you have Docker and Docker Compose installed on your system. You can install them following the official documentation:

- [Docker Installation Guide](https://docs.docker.com/get-docker/)
- [Docker Compose Installation Guide](https://docs.docker.com/compose/install/)

## Installation

1. Clone this repository to your local machine:

    ```bash
    git clone https://github.com/quanglv1996/Secure-Web-Server.git
    ```

2. Navigate to the project directory and using root user:

    ```bash
    cd Secure-Web-Server
    sudo -i
    ```

3. Create `dhparam` folder and generate ssh key with openssl
    ```bash
    mkdir dhparam
    cd dhparam/
    openssl dhparam -out dhparam-2048.pem 2048
    ```

4. Create `conf.d` folder and create file config `default.conf`
    ```bash
    cd ..
    mkdir conf.d
    nano conf.d/default.conf
    ```
    Copy config below and paste to `default.conf`
    ```python
    server {
      listen 80;
      server_name yourdomain.com;
      root /public_html/;

      location ~ /.well-known/acme-challenge{
          allow all;
          root /usr/share/nginx/html/letsencrypt;
      }
    }
    ```

5. Start the Docker Compose services:

   ```bash
   docker-compose up -d
   ```

   This will launch the Nginx web server and the Certbot service.

6. Re-fix `default.conf` file

   ```python
    server {
      listen 80;
      server_name yourdomain.com;
      root /public_html/;

      location ~ /.well-known/acme-challenge{
          allow all;
          root /usr/share/nginx/html/letsencrypt;
        }
          location / {
          return 301 https://www.yourdomain.com$request_uri;
        }
    }
    server {
      listen 443 ssl http2;
      server_name www.yourdomain.com;
      root /public_html/;

      ssl on;
      server_tokens off;
      ssl_certificate /etc/nginx/ssl/live/www.yourdomain.com/fullchain.pem;
      ssl_certificate_key /etc/nginx/ssl/live/www.yourdomain.com/privkey.pem;
      ssl_dhparam /etc/nginx/dhparam/dhparam-2048.pem;
      
      ssl_buffer_size 8k;
      ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
      ssl_prefer_server_ciphers on;
      ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

      location / {
          index index.html;
      }
    }
    ```
   Certbot will request and install SSL certificates for your specified domain.
7. Restart the Docker Compose services:

   ```bash
   docker-compose down
   docker-compose up -d
   ```
   This will launch the Nginx web server and the Certbot service.
  
8. Set to start the service when the server is turned on.

    
    First, create a systemd unit file. Using a text editor, you can create a new file, for example my-docker-service.service:
    ```bash
    sudo nano /etc/systemd/system/my-docker-service.service
    ```
    In this file, you can define the systemd unit file for your Docker Compose service. Here is an example:
    ```Python
    [Unit]
    Description=My Docker Compose Service
    After=network.target

    [Service]
    ExecStart=docker-compose -f /path/to/your/docker-compose.yml up -d
    WorkingDirectory=/path/to/your/docker-compose-directory
    Restart=always
    User=yourusername

    [Install]
    WantedBy=multi-user.target
    ```
    Update systemd: After you have created the systemd unit file, you need to update information about systemd services with the following command:
    ```bash
    sudo systemctl daemon-reload
    ```
    Turn on the service: Now, you can turn on the docker compose service and set it to automatically run when the computer starts with the following commands:
    ```bash
    sudo systemctl start my-docker-service
    sudo systemctl enable my-docker-service
    ```
    Check service status: You can check the status of the service with the command:
    ```bash
    sudo systemctl status my-docker-service
    ```

9. Set to renew SSL certificate daily

    Edit Crontab: Use crontab -e command to edit your crontab and add docker-compose run --rm certbot renew command to crontab. Make sure you specify its runtime. For example:
    ```bash
    nano crontab -e
    ```
    Add to `crontab`
    ```bash
    0 0 * * * docker-compose -f /path/to/your/docker-compose.yml run --rm certbot renew
    ```
    Reboot server
    ```bash
    sudo reboot
    ```

## Directory Structure

- `public_html`: Place your web content here.
- `conf.d`: Store Nginx configuration files.
- `dhparam`: Directory for Diffie-Hellman parameters.
- `certbot/conf`: Certbot configuration and SSL certificates.
- `certbot/logs`: Certbot log files.
- `certbot/data`: Certbot webroot directory.

![Certbot Logo](https://certbot.eff.org/images/Certbot-Logo-3.svg)

## License

This project is open-source and available under the [MIT License](LICENSE).
```