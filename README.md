# A Comprehensive Guide to Strapi

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
