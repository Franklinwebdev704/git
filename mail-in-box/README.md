# Setting up a private email server with mail-in-a-box

## 1. INTRODUCTION

[Mail-in-a-box](https://mailinabox.email/) is an open-source software package that facilitates the process of setting up and managing a private email server. Mail-in-a-box is designed to run on a [VPS(virtual private server)](https://en.wikipedia.org/wiki/Virtual_private_server), it provides a complete email solution, including [SMTP (simple mail transfer protocol)](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol), [IMAP (internet message access protocol)](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol), webmail, [spam filtering](https://en.wikipedia.org/wiki/Email_filtering), and other security features.

In this documentation, we are going to guide you through the process of creating a private email server with mail-in-a-box.

**Requirements:**
- Fixed IP address
- Virtual private server
- A domain name 

> [!NOTE]
> At the moment of writing this documentation mail-in-a-box only supports versions 14.04,18.04 and 22.04 of Ubuntu

## 2. INSTALLATION AND CONFIGURATION OF MAIL-IN-A-BOX

2.1. creating a new user and granting root privileges
  Before we start with the installation and configuration of mail-in-a-box, we need to create a new user that will keep all our configurations for our mail server
  
**Execute:**           
                                                                                            
    sudo adduser franklin


![image](https://github.com/Franklinwebdev704/git/assets/40476836/23612640-4738-4ee4-ab59-860f1e3baa20)

![image](https://github.com/Franklinwebdev704/git/assets/40476836/709f6845-3e7f-45d7-8ecb-c914901f7498)


### 2.1.2. Granting Root Privileges

After the creation of our new user, we have to grant root privileges to that user to enable him
to execute commands that require root privileges
**Execute:** 

    sudo usermod -aG sudo franklin


![image](https://github.com/Franklinwebdev704/git/assets/40476836/792f6e8a-8ead-4049-add9-4f3741cfc8f7)

### 2.2. installing mail-in-a-box
Firstly, we need to switch from our root to our newly created user and update our packages

**Execute:** 

    su - franklin

![image](https://github.com/Franklinwebdev704/git/assets/40476836/0248f557-1d08-46bc-aa7d-26066d084c39)

**Execute:** 

     sudo apt update
![image](https://github.com/Franklinwebdev704/git/assets/40476836/1005e668-e53a-460a-bcd1-6a5f7d8e93da)


after updating our packages, we can finally install mail-in-a-box

**Execute:** 

    curl -s https://mailinabox.email/setup.sh | sudo -E bash

![image](https://github.com/Franklinwebdev704/git/assets/40476836/a275b214-0a99-4d84-8321-0a1ff5687eda)

After executing this command, you will be prompted with this installation wizard, it will run
you through the process of installing mail-in-a-box

![image](https://github.com/Franklinwebdev704/git/assets/40476836/cbef88cd-5009-4cd0-8c22-0ece8d4c8ef4)

After validating the previous wizard, you will be prompted with this second wizard which will
ask you to enter the email address you want to use with the server

> [!NOTE]
> The email address you enter should contain your domain name

![image](https://github.com/Franklinwebdev704/git/assets/40476836/b780e5ef-2284-4c39-bbf3-1465490e89cd)

Next, you will be prompted to enter a hostname that will be used to access your box when the installation is over

![image](https://github.com/Franklinwebdev704/git/assets/40476836/2c786380-3660-4e86-b4a1-316facc08899)

The two next wizards will be used by mail-in-a-box to configure your geographic area and your time zone

![image](https://github.com/Franklinwebdev704/git/assets/40476836/e69e440a-7741-4397-8121-32a69c6fe03b)

![image](https://github.com/Franklinwebdev704/git/assets/40476836/c9167b2b-837a-4012-886e-ec5d32445145)

After filling out all the required information, mail-in-a-box will start automatically configuring itself. it will download the required applications it needs like
- nsd( DNS server)
- Postfix (SMTP server)
- Nginx (Web Server)
- Dovecot (IMAP server)
It will equally install and configure an SSL/TLS certificate for your domain
![image](https://github.com/Franklinwebdev704/git/assets/40476836/9231f357-610c-4fb3-b8fb-37d53c0fa95c)

After the completion of the configuration, a link will be generated dor my case the link is **https://138.197.22.171/admin**. The link will permit you to access and manage your
mail server.

### 2.3. ENABLING SSL/TLS ON OUR MAIL SERVER

Though our server is currently up and running, we still have a small problem we need to fix. If we try to access our now through any browser, we will be greeted with this error page

![image](https://github.com/Franklinwebdev704/git/assets/40476836/c2f3495a-5724-4a9f-9c75-239143adc827)

This error page signifies that our SSL/TLS is not yet registered. To fix this problem, we will need to access our mail server and provision it to our mail server. To access the login form that will enable us to interact with our server, we will need to click on **Advanced** and then click on Continue to **138.197.22.171 (unsafe)**

![image](https://github.com/Franklinwebdev704/git/assets/40476836/1c372e10-4f03-4094-8776-e14d26fd1c2c)

After clicking on the link prompting you to proceed to the next page, a login form will be displayed on your browser and on that login form you will enter the email and password you entered when installing mail-in-a-box  

![image](https://github.com/Franklinwebdev704/git/assets/40476836/46974d77-3180-4224-90b5-54f65d101ed5)


> [!NOTE]
> Always double-check your information before signing in

Once you are signed in you will be greeted with this screen

![image](https://github.com/Franklinwebdev704/git/assets/40476836/38540452-9cec-40d8-837b-d7cd79bddc60)

![image](https://github.com/Franklinwebdev704/git/assets/40476836/19c54e59-3663-4993-9081-3b414c704477)

And click on **TLS/SSL certificate**

![image](https://github.com/Franklinwebdev704/git/assets/40476836/31badf3b-7c43-49ed-8f21-05a622e644b5)

on this page you will see different certificates that need to be installed for the error to be solved, to install those certificates we need to manually click on **install certificate**

## 3.Testing our mail server

This chapter will be dedicated to testing the mail server we created with mail-in-a-box

**Requirements:**
- A web browser (for example Google Chrome)
  
### 3.1. Accessing our server

To access our mail server, we need to enter the domain name or IP address of our server into our browser. For example: the IP address of our server is 138.197.22.171 to access our mail
server we will enter 138.197.22.171/mail Once that is done, you will be presented with this interface where you will be asked to enter your username( which is the email address you entered when creating your box) and your password (the password you entered when creating your box)

![image](https://github.com/Franklinwebdev704/git/assets/40476836/0526b89a-3169-4d89-af95-9a13c87b3cef)

Once signed in, you will be presented with this page, if the information you entered is correct

![image](https://github.com/Franklinwebdev704/git/assets/40476836/5ba17c73-1b47-4cce-9d4b-59abc69b0c8a)

3.2 sending a mail
On your home page, you will click on compose, it looks like this
![image](https://github.com/Franklinwebdev704/git/assets/40476836/cb221a87-db1e-4768-8cc1-b9397b0d3989)

It will open a page where we will be able to write our emails

![image](https://github.com/Franklinwebdev704/git/assets/40476836/138d6c3e-2b78-4d4f-95b5-cb77966f29f0)

This is how the page looks like, on this page, there are 3 important sections.
- a section where you enter the receiver of this email and the subject.
- another section to include attachments like pictures or files
- a final section where you will include your message
Now let's write our first email. Below is a simple test mail that we are going to send to ourselves


![image](https://github.com/Franklinwebdev704/git/assets/40476836/e8c4a77e-cf08-466b-ab5a-86f31d00b7e4)

once that is done, all what is left to click on **send**



    







