# Set up a NGINX Reverse Proxy using a Let's Encrypt certificate and docker-compose

This is intended as an easy guide to set up a Docker network using Nginx as a reverse proxy in a single configuration file.

## Introduction
I'm new in docker containers and, while setting up everything, ran in a common issue while following other guides: Nginx couldn't access certificate files.
The problem was, as I found, that `docker-compose` (in mounting volumes) has an issue with sym links.
For this reason I'm binding directly `/etc/letsencrypt` directory as you can see in my compose file.
Since I haven't found much informations around on this issue, I decided to share my solution as a guide.

## Prerequisites
1. Install [docker](https://docs.docker.com/engine/install/) & [docker-compose](https://docs.docker.com/compose/install/)

2. Request a certificate to Letsencrypt using [Certbot](https://certbot.eff.org). Follow the guide selecting `"None of the above"` and your System. For now, skip the renewal hooks.

## Installation and Configuration
1. Edit docker-compose.yml, adding the services you need.

2. Edit nginx/default.conf following the instructions.

3. Go into ./nginx and run the following command, to generate dhparams.pem:

        openssl dhparam -out dhparams.pem 4096

4. Run your docker containers:

        docker-compose -f docker-compose.yml up -d

5. You should be online. If not, try a `docker ps` to check if nginx container is running. If not, you can access the logs and try to see what's not working by:

        docker-compose -f docker-compose.yml logs nginx

6. Set the proper renewal hooks. In my case, since I chose to run Certbot --standalone, I created the following hooks files:

        sudo sh -c 'printf "#!/bin/sh\ndocker-compose -f [PATH-TO-COMPOSE-FILE]/docker-compose.yml stop nginx\n" > /etc/letsencrypt/renewal-hooks/pre/nginx.sh'
        sudo sh -c 'printf "#!/bin/sh\ndocker-compose -f [PATH-TO-COMPOSE-FILE]/docker-compose.yml up -d\n" > /etc/letsencrypt/renewal-hooks/post/nginx.sh'
        sudo chmod 755 /etc/letsencrypt/renewal-hooks/pre/nginx.sh
        sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/nginx.sh

7. Test automatic renewal:

        sudo certbot renew --dry-run

## License
All code in this repository is licensed under the terms of the `MIT License`. For further information please refer to the `LICENSE` file.