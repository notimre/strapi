&nbsp;    

<img width="100" align="left" alt="Strapi monogram logo" src="https://github.com/NewcastleRSE/tools_and_practices/assets/52718007/3b7c715c-e61c-4f1b-beee-f34f3b3b91d4">

# A Comprehensive Guide to Strapi
### Created by [Imre Draskovits](https://github.com/notimre) - Summer 2023

&nbsp;   
   
TODO: Add table of contents    
TODO: Add best design practices when creating collection types


## Getting Started

Fork the [Standard-Strapi-Project Repository](https://github.com/NewcastleRSE/Standard-Strapi-Project)
   
The repository includes:
- everything mentioned in this guide 
- an up to date version of Strapi
- a ready-to-go admin portal set up for you
- additional explanatory and example files (e.g. `DEVELOPER.md`, `.env.example`) 

## Updating your existing Strapi Project

1. Stop the server 
   (Skip if you have GitHub Actions set up)
3. Rewrite all the `@strapi/*` dependencies in `package.json`:   
   
   <img width="1052" alt="image" src="https://github.com/NewcastleRSE/strapi/assets/52718007/5fe2deee-b4e7-4937-9f3f-d017ed4ecf46">
   You can find the versions by hovering over the version number, IntelliJ automatically finds and suggests the latest version.
   
4. Install the updates 
   ```
    yarn install
   ```
6. Build the project again
   ```
    yarn build
   ```
   **(Imporant step! Not equivalent to `yarn develop`)**
8. Run the server again  
   ```
    yarn develop
   ```
   or [create a new release](https://github.com/NewcastleRSE/strapi/tree/main#create-a-new-version-release)
         
         
## Running Strapi locally

1. **Create a local MySQL Server**  
   
    a. Download [MySQL Workbench](https://www.mysql.com/products/workbench/)   
    b. Set up a new connection (leave everything as prefilled) [DATABASE_PORT]  
    c. Create a 'New Schema'    
    d. Provide Schema Name and Password [DATABASE_HOST, DATABASE_NAME]   
    e. Under 'Users and Privileges' create a new user   
    f. Provide username and password [DATABASE_USERNAME, DATABASE_PASSWORD]   
    g. Go to 'Schema Privileges' and select all object rights and all DDL rights   
    h. Apply  

2. **Create a new a `.env` file to the root folder directory**  

    The `.env` file should contain the following:
    ```
    HOST=0.0.0.0
    PORT=8080
    URL=http://localhost/
    APP_KEYS=
    API_TOKEN_SALT=
    JWT_SECRET=
    ADMIN_JWT_SECRET=
    DATABASE_HOST=
    DATABASE_PORT=3306
    DATABASE_SSL=false
    DATABASE_NAME=
    DATABASE_USERNAME=
    DATABASE_PASSWORD=
    SENTRY_DSN=           
    PUBLIC_URL=http://localhost:8080/
    PUBLIC_ADMIN_URL=http://localhost:8080/admin
    SENDGRID_API_KEY=
    ```
    
    Do **NOT** commit the `.env` file to the repository.   
    If you need help with the file ask Imre or another team member.  

3. Install the Strapi dependencies
   ```
    yarn install
   ```

4. **Run the server in development mode**   
    ```
    yarn develop
    ```
         
## Connect GitHub Actions to Azure

You are advised to set up the Strapi repository with GitHub Workflow Actions to help publish and deploy your application to Azure easily.

You need to modify 5 lines across two files in `.github/workflows/`, indicated with `tobemodified` values:


1. **In `build.yaml` file:**
   ```
   env:
    REGISTRY: tobemodified
    IMAGE_NAME: tobemodified
   
   ...
   ```
   
   
   You can find the `REGISTRY` and `APP_NAME` value at:  
     - Azure Portal ➡️    
     - Project Resource Group ➡️    
     - Project Container Registry ➡️   
     - Access Keys     
     
    The `APP_NAME` is unique Azure wide, there can be no duplicates, it is likely your project name on Azure.   
    You can find the `IMAGE_NAME` value at: `REGISTRY`/`APP_NAME`   

   **[Example](https://github.com/NewcastleRSE/rockart-oman-api/blob/dev/.github/workflows/build.yaml)**:
   ```
   env:
    REGISTRY: rockartcareoman.azurecr.io
    IMAGE_NAME: rockartcareoman.azurecr.io/api
   
   ...
   ```
   

2. **In `build-deploy.yaml` file:**
   ```
   env:
     REGISTRY: tobemodified
     IMAGE_NAME: tobemodified
     APP_NAME: tobemodified

   ...
   ```
   
     **[Example](https://github.com/NewcastleRSE/rockart-oman-api/blob/dev/.github/workflows/build-deploy.yaml)**:
   ```
   env:
     REGISTRY: rockartcareoman.azurecr.io
     IMAGE_NAME: rockartcareoman.azurecr.io/rockartcareoman
     APP_NAME: rockartcareoman

   ...
   ```
 3. Set up the Secrets in the repository
    Go to `https://github.com/NewcastleRSE/project-name/settings/secrets/actions`
    
       a. **Add a new repository secret:** `PUBLISH_PROFILE`   

       You can find it at:
       - Azure Portal ➡️
       - Project Resource Group ➡️
       - Project Container Registry ➡️
       - `Download Publish Profile` on the top side bar

      ![Screenshot 2023-06-02 at 3 29 08 pm](https://github.com/NewcastleRSE/strapi/assets/52718007/0851251f-66f7-409f-9f8a-96297f109778)
      (Ignore the fact Azure is incapable of loading simple SVG Icons on occasions, normally it's a ⬇️ icon)

       Then copy paste the XML file contents to GitHub.

       b. **Add repository secret:** `REGISTRY_USERNAME`   

       You can find it at:
       - Azure Portal ➡️
       - Project Resource Group ➡️
       - Project Container Registry ➡️
       - Access Keys ➡️
       - `Username`

       c. **Add repository secret:** `REGISTRY_PASSWORD`   

       You can find it at:
       - Azure Portal ➡️
       - Project Resource Group ➡️
       - Project Container Registry ➡️
       - Access Keys ➡️
       - `Password`

       **Make sure you name everything exactly the same as above**

## Example of GitHub Action for other cloud providers
To adapt the above approach for Oracle, you may need to ssh into the VM and run git and docker commands, for example from the OrQA project:
```
name: staging

on:
  push:
    branches: 
      - dev

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: SSH Remote Commands
      uses: appleboy/ssh-action@v0.1.10
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: ${{ secrets.STAGING_USERNAME }}
        key: ${{ secrets.STAGING_KEY }}
        script: |
          docker compose stop strapi
          docker compose rm orqa-container 
          cd orqa-strapi
          git pull
          cd ..
          docker compose build --no-cache strapi
          docker compose up -d strapi
```

## Create a new version release

The deployment is done by [Terraform](https://www.terraform.io/).  
You can see (and adjust) the process of deployment in `.github/workflows/` directory.   

1. Commit changes to your branch
2. Merge your branch to main
3. This triggers an automated GitHub Workflow deployment
4. Create a release   
   a. Go to [the root of the repository](https://github.com/NewcastleRSE/rockartcare-api/)   
   b. Click ['Releases'](https://github.com/NewcastleRSE/rockartcare-api/releases)   
   c. Click 'Draft a new release'  
   d. Under 'Choose a new tag' create a new version number (e.g. 2.0.1)  
   e. 'Release Title' should match the created version number     
   f. Make sure 'target: main' branch is selected to deploy from  
   g. Leave 'Set as pre-release' un-ticked.  
   h. Tick 'Set as the latest release'  
   i. Click 'Publish Release'   
   j. Wait for Azure to deploy it (can take up to 15 minutes)  

## Image mounting with Azure

The container registry restarts on occasion (with no cause necessary).
It's important to set up Images and Media Mounting for Azure to prevent  Azure storing the media files in a separate blob.
Not setting up this will result in the container registry restarting, resulting you 'losing' read access to any media files prior uploaded on Strapi.

1. Navigate to your Project's Storage Account
2. Choose `File Shares` from the left side bar
3. Click `➕ File Share` to add a new entry
4. Name it related to your project
5. Choose `Hot` tier
6. Create

This in theory should mount your App Container to your Storage Account, therefore not 'losing' any media when the server restarts.

## Other cloud providers
In the case of Oracle, you need to deploy Strapi to a VM. Oracle calls these 'Compute Instances'

Make sure you have VCN set up in advance, which will contain the Strapi Compute Instance, the database and the frontend e.g. a Container Instance.

1. Create a compute instance on the Oracle Cloud, ensuring it is inside the correct compartment, and in the public subnet of the VCN. Generate or add an ssh key as part of the creation.

2. Once created, select 'Quick Actions' from the left hand menu under the Resources title, and set up the instance to access the internet.

3. Add the ssh key to your local .ssh folder

4. Log into the instance from your local command line with `ssh -i
.ssh/ssh-key.key opc@<instance_IP>`
(NB, if having issues, check the ssh key has the right permission settings with `chmod 400 ssh-key.key`

5. Once inside the instance, install git, yarn, node, docker and nvm (using yum,
assuming using an oracle linux base which is closest to CentOS).

6. Pull the strapi repo to the instance. You may need to update the environment variables in the Dockerfile using, e.g. nano.

7. Build the docker container inside the repository directory using `sudo docker build -t orqa-strapi .`

8. Once built, run the docker container using `sudo docker run -p 1337:1337 --detach --name orqa-container orqa-strapi`

9. Check everything is working as intended by viewing the logs. First check the
instance ID with `sudo docker ps` then use the ID that shows up for the running
container to access the logs: `sudo docker logs -f <container_ID>`

10. You may need to disable the firewall if the web interface is not showing.
Check the firewall status with `sudo systemctl status firewalld`, if it is on
then you can turn it off with sudo `systemctl stop firewalld`.

##### Updating the docker container

1. First stop the running instance, find the id with `sudo docker ps`, then
`sudo docker stop <instance_ID>` or use the container name `sudo docker stop orqa-container`

2. Remove the container using `sudo docker rm orqa-container` (this is so we can reuse the container name which is necessary for the GitHub Action)

2. Pull the latest changes from github `git pull` 

3. Rebuild the docker container and re-run it, following instructions from number 7 onwards above.

## Email setup with Sendgrid

 **[Do NOT use the default email provider with Strapi](https://antonioufano.com/articles/easily-send-emails-in-strapi-with-any-provider/)**   
     
 **Set up an [SMTP Server](https://sendgrid.com/blog/what-is-an-smtp-server/) as a plugin instead:**
  1. Sign up to [Sengrid](https://app.sendgrid.com)
  2. Create a [Single Sender Verification](https://app.sendgrid.com/settings/sender_auth/senders/new)   
     a. You need an existing email address for this. 
        You can request an alias from NUIT via [submitting a ticket](https://nuservice.ncl.ac.uk/HEAT/Modules/SelfService/#serviceCatalog)
  3. Create an [API Key](https://app.sendgrid.com/settings/api_keys) on `sendgrid`
     **If you set up email on an existing Strapi project**  
        a. Add extra property to your local `.env` file:   
        ```
        SENDGRID_API_KEY=createdInStep3Link
        ```
        b. Add the new variables to terraform build files in `terraform/variables.tf`:   
        ```
         variable "sendgrid_api_key" {
           type        = string
           sensitive   = true
         }
        ```
        c. As well as in `terraform/main.tf`:
        ```
        app_settings = {
           ...
           SENDGRID_API_KEY = var.sendgrid_api_key
        }
        ```
  4. Install `sendgrid` plugin to your Strapi project   

     ```
     yarn add @strapi/provider-email-sendgrid --save
     ```
  5. Add the following code to your `config/plugin.js` file:
     ```
        email: {
          config: {
            provider: "sendgrid",
            providerOptions: {
              apiKey: env("SENDGRID_API_KEY"),
            },
          settings: {
            defaultFrom: "sender@email.com",
            defaultReplyTo: "userRepliesToThis@email.com",
           },
          },
        },
     ```
  6. Re-build the project
      ```
      yarn build
      ```
  7. Apply the terraform changes, so Azure has the API Token   
     (run the command in `./terraform/` directory)
     ```
     terraform init
     ```
     ```
     terraform apply
     ```   
     Find the properties in the `.env` file for terrafrom to deploy.   
     You also need the `.tfstate` (terraform state) file for the project. (This is not stored on version control)

## Making strapi available behind https when running on a VM

When running on cloud services such as Oracle, Strapi may be deployed on a VM. To make Strapi run on https, you can use Nginx and Certbot docker containers. 

This assumes you have strapi running in a docker container on a VM with docker installed, and witha  public facing IP address. It also assumes you have access to the teams Cloudflare account and have pruchased a domain, e.g. through Google Domains, or are using the team's domain.

1. Add an A record in Cloudflare pointing your chosen url at the public IP of the VM. Turn off proxy status by clicking the orange cloud.
1. Create a `docker-compose.yml` file describing the nginx, certbot and strapi containers. Cerbot will be used to create the required certificates. For example, this is one for the OrQA project:
```
version: "3"
services:
  strapi:
    image: orqa-strapi:latest
    build:
      context: ./orqa-strapi
    container_name: orqa-container
    restart: unless-stopped
    env_file: .env
    environment:
      - HOST:${HOST}
      - PORT:${PORT}
    networks:
      - strapi-network
    volumes:
      - ./app:/srv/app
    ports:
      - 1337:1337
  nginx:
    image: nginx:1.15-alpine
    ports:
      - 80:80
      - 443:443
    networks:
      - strapi-network
    volumes:
      - ./data/nginx:/etc/nginx/conf.d
      - ./etc-letsencrypt:/etc/letsencrypt
      - ./certbot/www/:/var/www/certbot
  certbot:
    image: certbot/certbot
    networks:
      - strapi-network
    volumes:
      - ./etc-letsencrypt:/etc/letsencrypt
      - ./var-lib-letsencrypt:/var/lib/letsencrypt
      - ./certbot/www/:/var/www/certbot
      - ./var-log-letsencrypt:/var/log/letsencrypt
networks:
  strapi-network:
```
1. You will need to add a section to the `middlewre.js` file in your strapi repo, as described [here](https://stackoverflow.com/questions/74829211/resolve-strapi-io-content-security-policy-error). Add this to `config/middleware.js`:
```
{
    name: 'strapi::security',
    config: {
      contentSecurityPolicy: {
        useDefaults: true,
        directives: {
          'connect-src': ["'self'", 'http:', 'https:'],
          upgradeInsecureRequests: null,
        },
      },
    },
  },
```
1. If you need, create a `.env` file which contains the env variables listed in `docker-compose.yml`. Change the strapi urls to match what you set up in Cloudflare.
1. Create a nginx config file `data/nginx/app.conf`, replacing the urls with the one you set up in Cloudflare. The certificates section refers to the locations where certbot will save your certificates. If you change this location in the `docker-compose.yml` file, you will need to update them here. For example:
```
server {
    listen 80;
    server_name orqa-strapi-dev.orqa.uk;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name orqa-strapi-dev.orqa.uk;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/orqa-strapi-dev.orqa.uk/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orqa-strapi-dev.orqa.uk/privkey.pem;

    location / {
        proxy_pass  http://orqa-strapi-dev.orqa.uk:1337;
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_pass_request_headers on;
    }
}

```
1. Ask certbot to create the required certificates `docker compose run certbot certonly -d orqa-strapi-dev.orqa.uk --manual --preferred-challenges dns` (inserting your url). You will be asked to provide your email address. At the point where you are asked to create a TXT file, leave the terminal running and go to Cloudflare and use the name and value provided by certbot to create a new TXT record in cloudflare. You can test this has worked by visiting the Google DNS dig service that certbot recommends (e.g. https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.orqa-strapi-dev.orqa.uk) and checking that the value you provided in the TXT record is shown. Once this has worked, return to the terminal and press Enter. You should then see a message confirming that cerbot has created your certificates. 
1. You can now bring the rest of the stack up using `docker compose up -d`. Other useful commands are `docker compose restart`, `docker compose logs -f strapi` (replace strapi with nginx to view nginx logs). If you encounter any issues, check what's going on in the logs.

This certificate will not be automatically renewed and expires after 3 months. We need to add here instructions for setting up automatic renewal or how to renew. 

Note, for convenience, you can add your user to the docker users group to avoid using sudo: `sudo usermod -aG docker $USER` `newgrp docker`

### Useful links about strapi and https
https://hackmd.io/@Vatten/HyA1k1_ut   
https://www.willianantunes.com/blog/2022/08/create-a-certificate-using-certbot-through-docker
