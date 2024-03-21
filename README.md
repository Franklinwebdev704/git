# Setting up a self-hosted git server with NGINX and letsencrypt

## OUTLINE 
1.	Introduction
2.	Git Configuration
3.	Configuring Nginx and Enabling SSL/TLS encryption with LetsEncrypt
4.	Testing our server

## CHAPTER1: INTRODUCTION
This documentation guides you through the process of establishing a private Git server on your infrastructure. Git is a distributed version control system (DVCS), that enables developers to efficiently track and manage changes within their codebases. By deploying a self-hosted Git server, you gain several advantages, including:
-	greater control over our data
-	Tailored workflow, and integrations
-	And potentially lower costs for organizations

### Prerequisites:

This tutorial assumes you have access to two machines:

**Server Machine:**

 This machine will act as the central repository for your Git projects.
 
**Client Machine:**

 This machine will be used for testing and interacting with the Git server. 

By following the steps outlined in this tutorial, you'll successfully establish a private self-hosted Git server.

## CHAPTER 2: GIT CONFIGURATION

1. System Updates:
   
   **Execute:**

       sudo apt update -y

   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/f99a8fc1-3f50-49d8-933a-16e389638491)

   This command ensures your system is up-to-date with the latest packages

3. Package Installation:
   
     **Execute:**

       sudo apt install git nginx fcgiwrap -y

   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/393a66da-fa20-440d-a1c7-905849b0fe2a)

   This command installs the required software packages:

- **git:** The core Git version control system.

- **nginx:** A web server

- **fcgiwrap:** A FastCGI wrapper that will bridge communication between Nginx and the git

3.  Dedicated Git User (Recommended):
   
    **Execute:**

        sudo adduser git

This command will create a dedicated user called “git” where all your repositories and public keys will be stored
When creating the new git user, you should add a password though it is optional. I will strongly recommend to add a password for security reasons 

![image](https://github.com/Franklinwebdev704/git/assets/40476836/aee4bbe2-9683-4a96-82bf-fe0448d3223a)

>[!IMPORTANT]
> After creating the git user, you must add the user to the sudoers file, this will grant the git user the ability to execute commands that require root privileges using sudo. 
> To fix this problem, you must add the user to the sudoers file, follow the steps below to fix this problem
>  **Open the sudoers file:**
>
>   **Execute:**
>-        sudo visudo
>  
>    This command provides a safe way to edit the sudoers file
>
>   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/06f9d4f3-bc25-441d-834e-7f672a411861)
>
>  **Locate the user privileges section:**
>   Scroll down to the section that defines user privileges. This section looks like this
> -     # User privilege specification
> -           root    ALL=(ALL:ALL) ALL
>   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/635b97fa-9cb1-49ca-8656-046f1a753a5e)
>
>  **Add the git user**
>
>  After the root privileges, carefully enter this line
> -     git   ALL=(ALL:ALL) ALL


![image](https://github.com/Franklinwebdev704/git/assets/40476836/da09a160-28db-4b0b-857b-1b896a09be70)

> [!IMPORTANT]
> do not forget to save (ctrl + s) before exiting the editor (ctrl + x)


4. Creating the Repository Directory

   This directory will store our git repositories
   
**Execute:** 

    su - git

This command will enable to switch to the git user

![image](https://github.com/Franklinwebdev704/git/assets/40476836/66b14d92-bc32-4a18-8f53-1d5d275f8cab)


**Execute:** 

    sudo mkdir -p /srv/git

![image](https://github.com/Franklinwebdev704/git/assets/40476836/ffb7b47c-f6fd-43d0-bcee-b739dee1dd7e)

This command creates the directory /srv/git which will store your Git repositories. The **-p** flag ensures the parent directories are created if they do not exist

5. Setting Ownership and Permissions
    
   **Execute:**

       sudo usermod -aG www-data git

   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/b202f4b8-e831-461a-97be-001357626a71)

After installing NGINX, it automatically creates a user and a group called www-data, this command will add the git user to the www-data group which will enable NGINX to access our repositories stored in **/srv/git**

Currently, the /srv/git can only be accessed by the root user, we need to modify the ownership to enable git and all the members of the www-data to access the directory 

**Execute:**: 

    sudo chown git:www-data /srv/git/

![image](https://github.com/Franklinwebdev704/git/assets/40476836/5972a59a-c214-47c4-b55a-e86b043fe6ac)

The last step we have to do is to enable git and the members of the www-data group to **read(r)**, **write(w)**, and **Execute(X)** the directory

**Execute:** 

    sudo chmod 770 /srv/git	

![image](https://github.com/Franklinwebdev704/git/assets/40476836/c027a29d-eeba-42f6-a611-cc737ac29306)

This command sets the permissions on the /srv/git directory to allow the git user and the members of the www-data group to read, write, and execute access, further enhancing security.

### Part 2: Creating a Test Repository and Enabling Push Access
1. **Creating the Repository:**

**Navigate:** 

    cd /srv/git
    
**Create directory:**

    sudo mkdir testing.git
    
**Change directory:** 

    cd testing.git
    
**Initialize bare repository:**

    sudo git init . --bare
    
![image](https://github.com/Franklinwebdev704/git/assets/40476836/b96fc7d4-3b19-462b-835a-53ad4387ec5f)

The **sudo git init . --bare** command initializes a bare repository within the current directory. A bare repository doesn't contain a working directory for editing files. It only stores the Git metadata and project history

2.**Enabling Push Access**
  **Edit configuration:** 
  
      sudo nano config 
  
  ![image](https://github.com/Franklinwebdev704/git/assets/40476836/7e9e6083-8c4e-4e0a-8e9b-c8513c456128)
  
Explanation:

We use sudo nano config to open the config file of the newly created git repository. 

**Before**

![image](https://github.com/Franklinwebdev704/git/assets/40476836/fd5c081d-cbbf-4c43-acb0-1846f9c77cde)

**After**

![image](https://github.com/Franklinwebdev704/git/assets/40476836/7fcdc9be-d9f4-4d67-8530-170cb95383bc)

**Explanation:**

We add the following section to the configuration file:

            [http]
                receivepack = true

This new section enables the receivepack service, which is required for users to push changes to the repository. By default, this service is disabled for security reasons.

> [!IMPORTANT]
> do not forget to save (ctrl + s) before exiting the editor (ctrl + x)

3. **Reload**
   
   **Reload Configuration:**

       sudo git config --bool core.bare true

   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/d8d05613-d350-4a9e-8ed7-a424d4c1547a)
   
**Explanation:**
  The **sudo git config --bool core.bare true** command reloads the configuration, ensuring the changes take effect.

4. **Changing Directory Owner:**

   **Navigate:**

       cd ..
   
   **Execute:**

       sudo chown -R www-data:www-data testing.git/

   ![image](https://github.com/Franklinwebdev704/git/assets/40476836/c1eb4430-60ad-48d1-9d10-1cc3bceccd8c)

**Explanation:**
This command sets the owner of the testing.git repository directory and its contents recursively to the www-data user. This allows the web server (Nginx) to access the repository.

> **Summary : execute these steps in the same order in which they are presented**
> 1.	Create a repository in /srv/git/
> 2.	Navigate into the repository and initialize it with git init . --bare
> 3.	Edit the configuration file and set receivepack to true to enable push
> 4.	Run sudo git config --bool core.bare true for the changes to take effect
> 5.	Change the owner of the directory to www-data

**Part 3:  CONFIGURING SSH ACCESS AND ADDING USERS**

**1. Setting Up the SSH Directory:**

**Navigate:**

    cd ~

**Create .ssh directory:** 
    
    mkdir .ssh

**Set permissions:** 
    
    chmod 700 .ssh

**Create authorized_keys file:** 

    touch .ssh/authorized_keys

**Set file permissions:** 

    chmod 600 .ssh/authorized_keys

![image](https://github.com/Franklinwebdev704/git/assets/40476836/df715ae8-16ac-4714-84ce-b29f41f51a0f)

**2. Adding a User:**

the following steps will provide a guide to adding new users to interact with our Git repositories

**2.1. Generating SSH Keys (Client-Side):**

For this you will need to switch to the client machine and Generate the key pair:

     ssh-keygen -t rsa -b 4096 -C "user@example.com"

 ![image](https://github.com/Franklinwebdev704/git/assets/40476836/187a3d00-6424-4494-ae08-d56c691258ca)

This command generates an SSH key pair, consisting of a private key (id_rsa) and a public key (id_rsa.pub)

**Explanation:**

-	The **-t rsa** flag specifies the key type as RSA
  
-	The **-b 4096** flag sets a strong key length for enhanced security.
  
-	The **-C "user@example.com"** flag adds an optional comment to identify the key, often using an email address or username

  ![image](https://github.com/Franklinwebdev704/git/assets/40476836/cb8a6865-a746-4ba4-8c7a-75712dca31e6)

**2.2. Transferring the Public Key to the git user:**

**Execute:** 

    scp -i ./juniorhoza56_rsa user1.pub juniorhoza@167.99.23.250:/tmp

![image](https://github.com/Franklinwebdev704/git/assets/40476836/50d5d3cc-0784-44fb-9ccb-5d44866b38eb)

As you can see in the image above, we transferred our public key from our client machine into the /tmp directory of our server

This command utilizes the Secure Copy Protocol (SCP) to securely transfer the public key (id_rsa.pub) to the git server, or you can just copy the content of the public key file and directly paste it into the authorized_key file

**Breakdown:**

- **user1.pub:** The path to the public key file.

- **juniorhoza:** The username you use to access the git server.

- **gitlab.ouodesignllc.com**: The domain name or IP address of the git server.

- **/tmp:** Specifies the location where the file will be copied to. in our case, it will be copied to the /tmp directory

- **2.3. Appending Public Key to authorized_keys (Server-Side):**

Switch back to the git user on the server and execute the command below

**Execute:**  
    
    cat path/to/pub/file.pub >> ~/.ssh/authorized_keys

![image](https://github.com/Franklinwebdev704/git/assets/40476836/d9cc86f8-313d-44ef-a3e9-b5a72a9ecb8a)

This command appends the contents of the public key file (path/to/pub/file.pub) to the authorized_keys file. This will enable the user to access our server

  > [!NOTE]
  > Replace path/to/pub/file.pub with the actual path where the public key was transferred.
  Once these steps are completed, the user can securely access the git server using their private key for authentication

## CHAPTER 3: CONFIGURING NGINX AND ENABLING SSL/TLS ENCRYPTION WITH LETSENCRYPT

This chapter outlines the steps to configure Nginx as a web server for our Git server and enable SSL by using [Letsencrypt](https://letsencrypt.org/getting-started/) for free SSL certificates.

### prerequisites 
1. You must have root or sudo privileges on the server
2. You should have a registered domain or subdomain (letsencrypt does not issue domains for IP addresses)

### PART 1: CONFIGURING NGINX
**Step 1: Remove Default files**
 I would advise you to remove all default Nginx configuration files within /etc/nginx/sites-available/ and /etc/nginx/sites-enabled/ directories because they can interfere with our configurations. 
 
   **Navigate:** 
   
       cd /etc/nginx/
   
![image](https://github.com/Franklinwebdev704/git/assets/40476836/9ba73ef0-3c09-4163-8a86-91aa13ca884d)
   
**Remove default file in sites-available**

**Execute:** 
    
    sudo rm sites-available/default

![image](https://github.com/Franklinwebdev704/git/assets/40476836/79261ba1-49b8-441f-98c3-e1e8918f8f9e)

**Remove default file in sites-enabled**

  **Execute:** 
  
      sudo rm sites-enabled/default

  ![image](https://github.com/Franklinwebdev704/git/assets/40476836/7285a3ab-77fb-4bfc-bd00-c3c3c2188eb0)

**Step 2: create a .conf file**
Use these commands to create a new Nginx configuration file, this will equally open up nano
**Navigate:** 
      
      cd sites-available/

**Create GitServer.conf file:** 

    sudo nano GitServer.conf

![image](https://github.com/Franklinwebdev704/git/assets/40476836/23239577-370c-414b-bb3a-a182acc21425)

Enter these configurations in the file newly created 

    Server {
     listen 80;
       server_name gitlab.ouodesignllc.com;

                root  /srv/git;

        access_log /var/log/nginx/git_access.log;
        error_log /var/log/nginx/git_error.log;
        location ~ (/.*) {

                fastcgi_pass unix:/var/run/fcgiwrap.socket;
                fastcgi_param SCRIPT_FILENAME /usr/lib/git-core/git-http-backend;
                fastcgi_param GIT_HTTP_EXPORT_ALL "";
                fastcgi_param GIT_PROJECT_ROOT  /srv/git;
                fastcgi_param PATH_INFO $1;
                include fastcgi_params;
        }
    }

![image](https://github.com/Franklinwebdev704/git/assets/40476836/8542e80e-f9a5-451c-aa86-8a7f8baa40dc)

> [!IMPORTANT]
> do not forget to save (ctrl + s) before exiting the editor (ctrl + x)

**Breakdown:**
- **Listen 80:**  This line configures the server to listen on port 443 using secure SSL/TLS encryption.
    
-	**Server name:** It sets the domain name that will trigger our server block (you can equally use your public IP address if you do not have a domain)
    
 -	**root  /srv/git:** this line defines the directory that will be used by Nginx to serve our directories
   
 -	 **access_log /var/log/nginx/git_access.log;**
 -	
 -	 **error_log /var/log/nginx/git_error.log; :**
   
These two lines are optional, but the access_log will be used by Nginx to keep records about incoming requests and error_log will be used by Nginx to record errors encountered during its operations.

 -	**location ~ (/.*) { ... }:** This block defines a location block that matches any request URL path (denoted by the ~ wildcard).
   
 -	**fastcgi_pass unix:/var/run/fcgiwrap.socket;:** This line instructs Nginx to forward incoming to FastCGI server process listening on the UNIX socket  /var/run/fcgiwrap.socket. This socket is connected to the fcgiwrap process which executes git_http_backend

-	The following lines **(fastcgi_param ...)** define parameters passed to the **git-http-backend** script:
  
    - **SCRIPT_FILENAME:** Sets the path to the script to be executed (/usr/lib/git-core/git-http-backend)**.
 	
    - **GIT_HTTP_EXPORT_ALL:** Set to an empty string here.
 	
    - **GIT_PROJECT_ROOT:** Specifies the root directory containing the Git repositories (/srv/git).
 	
    - **PATH_INFO:** Captures the remaining part of the URL path for further processing by the git-http-backend script.
 	
  -	**include fastcgi_params;:** This line includes additional pre-defined configuration parameters for FastCGI communication.


**Step 3: Create a symbolic link between /etc/nginx/sites-available/GitServer.conf and /etc/nginx/sites-enabled**

**Navigate:** 

      cd ~

**Execute:** 

    sudo ln -s /etc/nginx/sites-available/GitServer.conf  /etc/nginx/sites-enabled/

![image](https://github.com/Franklinwebdev704/git/assets/40476836/ad2da54e-33bf-4bf0-b64c-ffad74c5d34b)


> [!IMPORTANT]
> After making any change to any of your nginx configuration files, do not forget to test (sudo nginx -t) your configurations for any error and reload your server (sudo systemctl reload nginx) for the changes to take effect.

![image](https://github.com/Franklinwebdev704/git/assets/40476836/b5591177-bb25-464f-b63e-4476bce56804)

### PART 2: GETTING AN SSL/TLS CERTIFICATE FROM LETSENCRYPT

### PART 2.1: FIREWALL CONFIGURATION

Before we can start with the process of issuing an SSL certificate, we need to ensure our server allows incoming requests we can do this by modifying the settings of our firewall.
When installing Nginx it registers a few profiles with UFW(uncomplicated firewall).

**1.	Verify firewall status**

**Execute:**

      sudo ufw status
If ufw is not active, execute 

    sudo ufw enable

![image](https://github.com/Franklinwebdev704/git/assets/40476836/ad3be500-f0f1-4f41-ae16-821425416109)

**2.	Allow HTTPS traffic**

remove Nginx HTTP and Nginx HTTP (v6) as they are unnecessary for HTTPS access. Additionally, allow the Nginx Full profile and port 443:

**Execute:** 

    sudo ufw allow 'Nginx Full'
    
**Execute:** 
    
    sudo ufw allow 443
    
**Execute:** 
    
    sudo ufw allow 80

![image](https://github.com/Franklinwebdev704/git/assets/40476836/96b0a3f4-24c9-4298-8129-65212c5914ad)

### PART 2.2: OBTAINING AN SSL/TLS CERTIFICATE FROM LETSENCRYPT

**Step 1: install certbot**
Certbot is a tool that enables us to obtain SSL certificates from letsencrypt. certbot recommends using their snap package for installation.  before we can install certbot, we will have to install snapd, certbot recommends using snapd for installation (you can still use apt if you want) 

**Update packet list:** 

    sudo apt update -y
![image](https://github.com/Franklinwebdev704/git/assets/40476836/ad861c3f-c19b-42bc-b843-2f55b58e7456)

**Install Snapd:** 

    sudo apt install snap -y

![image](https://github.com/Franklinwebdev704/git/assets/40476836/941d445f-2774-4fff-94a3-de6fa898d020)

**Install certbot:**
    
    sudo snap install --classic certbot 

![image](https://github.com/Franklinwebdev704/git/assets/40476836/95b69177-1e6f-46c4-ba1c-c81a34883b83)

**Step 2: Obtain SSL certificate**

Certbot provides a variety of plugins for obtaining SSL certificates. we will make use of the Nginx plugin to obtain our SSL certificate (you can see the list of plugins here)[https://eff-certbot.readthedocs.io/en/stable/using.html]

**Execute:**

    sudo certbot --nginx -d gitlab.ouodesignllc.com

After executing the command, LetsEncrypt will guide you through a verification process to confirm your domain ownership. after verification, you will receive the following message if it was successful


![image](https://github.com/Franklinwebdev704/git/assets/40476836/d2c9ada8-fbc4-49e0-aa30-ff44ef9cf0de)

### Step 3:  Test automatic renewal (optional)

Certificates issued by certbot only last for 90 days (3 months) but fortunately, certbot comes with a cron job that will automatically renew your certificate before it expires. You can test automatic renewal by running this command:

**Execute:** 

    sudo certbot renew --dry-run

This command is used to simulate the renewal process.

![image](https://github.com/Franklinwebdev704/git/assets/40476836/0710e19f-f055-4f1d-b19d-c415c18838b9)

### PART 3: MODIFY NGINX CONFIGURATION FILE (GitServer.conf)

After we successfully obtained an SSL certificate, we need to make small modifications to the NGINX configuration we previously made to support SSL.

**Access the configuration file:** 

    sudo nano /etc/nginx/sites-available/GitServer.conf

![image](https://github.com/Franklinwebdev704/git/assets/40476836/61d60920-85b4-477d-8c61-e2eb4939ab2a)

After opening the GitServer.conf file with, update the old configurations with these new configurations

    server{
     listen 443 ssl;
       server_name your_domain.com;

        ssl_certificate /etc/letsencrypt/live/gitlab.ouodesignllc.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/gitlab.ouodesignllc.com.com/privkey.pem;

                root /srv/git;

        access_log /var/log/nginx/git_access.log;
        error_log /var/log/nginx/git_error.log;
        location ~ (/.*) {

                fastcgi_pass unix:/var/run/fcgiwrap.socket;
                fastcgi_param SCRIPT_FILENAME /usr/lib/git-core/git-http-backend;
                fastcgi_param GIT_HTTP_EXPORT_ALL "";
                fastcgi_param GIT_PROJECT_ROOT /srv/git;
                fastcgi_param PATH_INFO $1;
                include fastcgi_params;
        }
    }

![image](https://github.com/Franklinwebdev704/git/assets/40476836/b3b7a45a-abeb-4405-8c83-64e0520cdfff)

> Changes made 
>
> -	Modified
> **listen 80; to listen 443 ssl;**
> -	Added:
> **ssl_certificate /etc/letsencrypt/live/gitlab.ouodesignllc.com/fullchain.pem;**
> **ssl_certificate_key /etc/letsencrypt/live/gitlab.ouodsignllc.com.com/privkey.pem;**

> [!IMPORTANT]
> after making any change to any of your nginx configuration files, do not forget to test (sudo nginx -t) your configurations for any error and reload your server (sudo systemctl reload nginx) for the changes to take effect.

![image](https://github.com/Franklinwebdev704/git/assets/40476836/b5591177-bb25-464f-b63e-4476bce56804)


## CHAPTER 4: TESTING OUR SERVER 

This chapter will be dedicated to testing our git server with our client machine. We will make use of the testing.git repository created earlier in Chapter 2

### Prerequisites:

-	Git installed on the client machine:
  
 you can verify if git is installed on the client machine by executing **git -version** in the terminal of your machine 
 
**Execute:**

 	     git –version

  ![image](https://github.com/Franklinwebdev704/git/assets/40476836/40e262a1-9924-4a17-bee2-b6043c294948)

If Git is installed, it will display the version information. If you don’t have git installed on your machine, you will have to install it. Follow the steps below based on your operating system

-	**Linux:** 
 	1. open terminal 
  2. **Execute:**:
   
       sudo apt install git -y

-	Windows and macOS: Download and install Git from the official website: https://git-scm.com/downloads.

  **1.	Configure your username and email**
  
**Execute:**

    	      git config --global user. name "Your Name"
            git config --global user.email "your.email@example.com"
  
These commands configure your git name and your email which will be used to identify your commits. The **--global** flag ensures these settings apply globally to all Git repositories on your system. 
    
**2.	Clone the repository**

Navigate to a desired directory on your system and execute this command to clone the testing.git repository

**Execute:**  
    
    git clone https://gitlab.ouodesignllc.com/testing.git

**Expected output:** Warning you appear to have cloned an empty repository

This indicates that our cloning was successful but the testing.git repository is empty

**3.	Let's make changes to our repository**

  We will make a simple change just for testing purposes, we will create a new file called helloWorld.txt and we will enter a message in the file.
  
**Execute:** 
    
          cd testing
         echo “hello world from client” >helloWorld.txt

4.	**Pushing changes**
   
-	**Execute:**

 	    git status (optional)
 	
-	**Execute:**:
  
        git add .
 	
        #This command will add the new file to the staging area

-	**Execute:**
  
        git commit -m “testing commit”

-	**Execute:**
  
        git push

![image](https://github.com/Franklinwebdev704/git/assets/40476836/df34b668-9a64-4a0b-9497-abfc87406424)


I hope this documentation has been helpful. I strongly encourage you to explore further and customize your Git server according to your specific needs. 😊
