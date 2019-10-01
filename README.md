# Privacy Impact Assesment Tool

The Private Impact Assesment (PIA) software is a free tool published by the [Commission Nationale de l'Informatique et des Libertés (CNIL)](https://www.cnil.fr) which aims to help data controllers build and demonstrate compliance to the GDPR. The tool is developped in angular/rails/nging/postgres by https://atnos.com and is distributed here: https://github.com/LINCnil/pia

For more background information, visit the [CNIL PIA software page](https://www.cnil.fr/fr/outil-pia-telechargez-et-installez-le-logiciel-de-la-cnil)

## Getting started

This repository adds a step by step how-to deploy the tool with SSL in production.
It is a fork of Killian Kemps (https://github.com/KillianKemps) who has created an awesome Docker configuration to setup a Docker environment for production purpose. Thanks to Killian it is much easier now to run PIA. A simple docker-compose up does everything and gives a running website in 3 containers: front-end, back-end and database. 

You will still have to create a domain, valid certificate and add the back-end URL on the front-end interface manually, which will be explained in this readme. This fork of is an example how we deployed our instance for testing purposes.

IMPORTANT!
If both front and backend are running properly, you still have to add your backend url to the frontend manually by clicking on the Start button, and then on the menu item Tools and then on the word Settings, else the PIA's will only be stored locally, not in the database.

## Prerequisites

1. Get and install **Docker** https://www.docker.com/get-docker on your machine
2. Clone this repository

## Local development

1. `cd pia-docker`
2. Run the containers by typing `docker-compose up` into the shell
3. Access the website with `localhost:80`

## How to deploy on digitalocean.com

You can also use this video to guide you how to use docker-machine in production: https://www.youtube.com/watch?v=jlVrYgVEl6M

### On your domain name provider
 1. Create a domain name on any domain provider, needed to be able to use ssl
 2. Add the 3 nameservers of digitalocean to this domain provider (this takes 1 day)
    A tutorial can be found [here](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars).

### On Digitalocean
 3. Create an account on https://digitalocean.com, add your billing information and create a new project. (We will create a droplet in step 7)
 4. In security: Add your ssh public key to be able to access the Droplet.

### In your local terminal
 5. Add your access-token to your environment using: `export DO_TOKEN=your_access_token`
 6. Add the SHA1 fingerprint created by Digitalocean of your ssh access `export DO_SSH_KEY=your_fingerprint_id`
 7. Create a new Droplet by using docker-machine (the server needs to have at least 2gb, to create the minified js files):
`docker-machine create --driver digitalocean --digitalocean-access-token $DO_TOKEN --digitalocean-region ams3 --digitalocean-size 2gb --digitalocean-ssh-key-fingerprint $DO_SSH_KEY your_name_of_droplet`
 8. Log into the created droplet: `ssh root@your_name_of_droplet`
 9. Create an SSL certificate by using letsencrypt:
    A tutorial can be found [here](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-16-04).
10. Solve the 'cert file not found' error when starting nginx by breaking the symlinks and using the origonal folder:
	```
	cd /etc/letsencrypt
	mkdir copy
	cp -rL live/* copy/
	```
11. Set the password variables on your local machine to be used with docker-machine later when starting docker-compose:
   ```
   export POSTGRES_PASSWORD=your_database_password
   export DATABASE_PASSWORD=your_database_password
   export SECRET_KEY_BASE=your_secret_key_password
   ```
12. Modify the frontend cnil_pia.conf by replacing pia-amsterdam.nl with your own domain and certificate name.
13. Modify the backend Dockerfile by replacing pia-amsterdam-cert with your own certificate name.
14. Start docker-machine in the Droplet: `eval $(docker-machine env pia-amsterdam)`
15. Start the database as a service: `docker-compose -d database`
16. Build the backend and frontend: `docker-compose build`
17. Start the backend and frontend: `docker-compose up`
18. Go to the ip address of the Droplet using port 80 to test if the site runs.
19. Visit your website by going to your domain name to test the SSL and nameserver connections.

## How to restrict access to the website for internal use

### On Digitalocean
20. In Domain: Connect the domain name to the Droplet.
21. In Domain: Check in the domain setup if the DNS A Type contains the ip of the created Droplet.
22. In Firewall: Setup the appropriate firewall ip restrictions to ports:
    -   80 = HTTP
    -  443 = HTTPS
    - 2376 = Docker-machine access
    - 3000 = API
23. In Firewall: Attach the Droplet to the firewall rules.
24. Open the website on a whitelisted and non-whitelisted ip address to test the firewall.
