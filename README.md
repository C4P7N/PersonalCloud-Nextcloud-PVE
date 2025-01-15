<p align="center"><img src="https://github.com/user-attachments/assets/509d8212-8b84-4f9c-ade5-a16bf21061f7" width="250" height="200"><br>
<a href="https://www.proxmox.com/en/products/proxmox-virtual-environment/overview">Proxmox VE</a> | <a href="https://www.debian.org/">Debian</a> | <a href="https://nextcloud.com/">Nextcloud</a> | <a href="https://www.cloudflare.com/">Cloudflare</a>
</p>

# Personal Cloud via Nextcloud in an LXC w/ Proxmox VE

#### $\textcolor{red}{\textsf{This guide assumes you already have Proxmox VE installed.}}$

### Skills & Tools Used in this Project
Linux Shell<br>
Proxmox VE<br>
Debian<br>
Apache<br>
MariaDB<br>
Nextcloud<br>
Cloudflare Tunnel

### Why would I use Nextcloud and Proxmox?
  Nextcloud is an open source client and server software that allows you to host your own collaboration efforts without relying on third party options. From file hosting, an office suite, chat and more, Nextcloud is like having your own self-hosted GSuite or Microsoft Teams where you're in full control. Hosting Nextcloud on our own Proxmox server furthers that control and eliminates any reliance on third parties. Nextcloud on PVE gives us full control on how our files are kept without the worry of a third party getting involved potentially shutting us down without reason or misusing our content for whatever their many reasons may be.<br>

# Getting Started
To set up Nextcloud on a Debian LXC in Proxmox VE, first we'll create our container, update Debian, and install Apache, MariaDB, and PHP, Download and unpack Nextcloud into the web server directory, Setup MariaDB by creating and configuring a database for Nextcloud, then configure Apache with necessary modules. Finish by accessing Nextcloud through a browser to set it up, and set up a Cloudflare tunnel so we can access it from the internet.

<details>
<summary>Creating our Debian LXC</summary>
- The first thing we have to do is make sure we have a Debian template downloaded. Head on over to your PVE WebUI, open your local storage, click CT Templates, and verify you have Debian-12-standard listed. If not click the Templates button and download it.
<details>
<summary>Template Download</summary>
  <img src="https://github.com/user-attachments/assets/e82ad414-2c65-45af-9036-04ff156e6985">
</details>
- Next we'll click the "Create CT" button in the top right.<br>
- This will bring up a window "Create: LXC Container.
  <details>
<summary>Create: LXC Container Window</summary>
  <img src="https://github.com/user-attachments/assets/7f78898c-eee9-45a5-830d-bda2173bdb64">
</details>
- In this first tab we'll enter an ID, a hostname like 'nextcloud' so we can easily identify it, and create a password. <br>
- Our next tab "Template" We'll select the storage we downloaded our template to(Default Local) and select our debian template. <br>
- Next we'll set up Disks, I'm just installing mine to local-lvm, and for now I'll give it 8GiB for the size. <br>
- Next the amount of CPU cores, for now we'll set 2. <br>
- Now we have Memory, we'll set 2048 for 2 GiB. <br>
- Network depends on your setup, I'll be leaving everything default except IPv6 will be DHCP, and IPv4 I want to set a static IP so I can easily access the Nextcloud WebUI later. <br>
- DNS I'll leave blank so it'll use host settings. <br>
- And then we'll confirm. <br>
- Once done we'll head over to our newly created LXC which should apear on the left. <br>
- Click on start to start it up. <br>
- Login to the Console with root as the username and the password will be the one you created earlier. <br>
- now update with the following command:
<code>apt update && apt full-upgrade -y</code>

</details>

<details>
<summary>Getting Everything Downloaded</summary>
We can start with this command to download all the packeges we'll need:
<code>
  apt install -y apache2 mariadb-server libapache2-mod-php php-gd php-json php-mysql php-curl php-intl php-mbstring php-imagick php-xml php-zip php-bcmath php-apcu php-redis
</code>
Now we can download and unzip the latest version of Nextcloud. Check for the latest <a href="https://download.nextcloud.com/server/releases/">here</a>. Currently the latest is 30. <br>
Change directory to /var/www/ <code>cd /var/www/</code> <br>
Use wget to download: <code>wget https://download.nextcloud.com/server/releases/latest-30.zip</code> <br>
Unzip with: <code>unzip latest-30.zip</code>
</details>
<details>
<summary>Setting Up MariaDB</summary>
To start we'll secure MariaDB by running a script, setting a password, and answering some prompts.<br>
- First run the script in the console of the LXC you created: <code>mysql_secure_installation</code><br>
- We'll be prompted for a password, hit enter.<br>
- Switch to unix_socket we'll answer n for no and hit enter.<br>
- Change the root password we'll answer y for yes and then create a new password.<br>
- Remove anonymouse users, we'll answer y.<br>
- Disallow root login remotely, we'll answer y.<br>
- Remove test database and access to it, answer y.<br>
- Reload privilege tables now, answer y.<br>
<br>
Now we'll create a database.<br>
- First we'll enter MariaDB with: <code>mysql</code><br>
<details>
<summary>It should look something like this</summary>
  <img src="https://github.com/user-attachments/assets/c2b55b17-3911-4d13-9a32-29789da9b0b3">
</details>
- Next within MariaDB we'll run: <code>CREATE DATABASE nextclouddb;</code><br>
- And then after changing nextclouduser and your_password to your desired username and password we'll run: <code>CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'your_password';</code><br>
- Next, change the username to the one you used in the last command and run: <code>GRANT ALL PRIVILEGES ON nextclouddb.* TO 'nextclouduser'@'localhost';</code><br>
- Then run: <code>FLUSH PRIVILEGES;</code><br>
- Then exit: <code>exit;</code><br>
</details>
<details>
<summary>Configuring Apache</summary>
Within the LXC's console we'll now configure Apache.<br>
- First we'll enable some modules running the following commands: <code>a2enmod rewrite</code>;<code>a2enmod headers</code>;<code>a2enmod env</code>;<code>a2enmod dir</code>;<code>a2enmod mime</code><br>
- Now we'll disable the default site: <code>a2dissite 000-default.conf</code><br>
- Next we want to create a file named nextcloud.conf: <code>nano /etc/apache2/sites-available/nextcloud.conf</code><br>
- Inside this file we'll paste the following text(unfortunately the formating in this file seems to break github's markdown so you'll have to copy from this image):<br>
<br>
<img src="https://github.com/user-attachments/assets/27495570-8669-4440-8b82-d7446a7755f3"><br>
<br>
- We'll enable this new config with: <code>a2ensite nextcloud.conf</code><br>
- Now we'll set ownership of the directory with: <code>chown -R www-data:www-data /var/www/nextcloud/</code><br>
- Now restart Apache: <code>systemctl restart apache2</code>
</details>
<details>
  <summary>Finish Nextcloud Install from WebUI</summary>
We should now be able to access Nextcloud's WebUI to finish up our install.<br>
- To access the WebUI take the IP Address of the container we made and enter it into your web browser.<br>
- When we first access this WebUI we'll be asked to create an admin account, go ahead and enter a desired username and password.<br>
- Below this we're asked our database information. From here go ahead and fill in the information of the database we created earlier.<br>  
- And now after hitting next our Nextcloud service is now installed. From here you can pick which add-ons you want. Don't select any and you'll have just your file storage.<br>
</details>
<details>
  <summary>Enabling Internet Access using a Cloudflare Tunnel</summary>
$\textcolor{red}{\textsf{In order to accomplish this step, you must have your own Domain name.}}$<br>
This step will require you to have your own domain name, you can use a variety of registrars, or you could use cloudflare itself. However if you're using a registrar other than cloudflare you'll need to configure your domain to work with cloudflare. This should be easy to accomplish by using your cloudflare dashboard, on the homescreen of your dashboard you should see a button "Add a domain" this will guide you through the process.<br>

### Setting up cloudflare
- First, if you haven't already, create an account for <a href="cloudflare.com">Cloudflare.</a><br>
- Next, on our account home if you haven't already Add a domain.<br>
- Once we have a domain on our cloudflare account, on the left of the dashboard home click on Zero Trust.<br>
- Next click on Networks.<br>
- Then click on Tunnels if it isn't up already.<br>
- On this page click on the button "+ Create a tunnel"<br>
- Select your tunnel type, pick Cloudflared.<br>
- Give your tunnel a name, Nextcloud should suffice.<br>
- Now we need to install and run a connector. Copy the text under "If you don’t have cloudflared installed on your machine:"<br>
- Head on over to your Proxmox VE's WebUI and access the console of your Nextcloud's LXC.<br>
- Paste the command into the console and hit enter.<br>
- Next set the domain name you want your Nextcloud server to have, and set the type to http with the url 127.0.0.1/ and save the tunnel.<br>
- Now try and access the domain you just setup. You should be greeted by your Nextcloud server with a message saying access from untrusted domain. We'll tackle that next.<br>
<br>
### Adding a Trusted Domain to Nextcloud<br>
Now we need to add our domain to Nextcloud's trusted domains list.<br>
- In your Nextcloud's LXC console enter this command: <code>nano /var/www/nextcloud/config/config.php</code><br>
- In this config file we'll add a second trusted domain by adding: 1 => 'your.domain.name',  <br>
- The config should look similar to the below image.
<img src="https://github.com/user-attachments/assets/74f061df-6c4a-4afa-a8a1-b3aa574cb10d">
- after config is set, hit ctrl + x then y to save and exit.
- You should now be able to access Nextcloud from the new domain.
</details>

# Moving Forward
From here you can check out the other capabilities of Nextcloud through the various apps offered for Nextcloud. You can find them by clicking on your user icon in the top right of the Nextcloud dashboard and clicking Apps.<br>
Don't forget to check out their <a href="https://nextcloud.com/install/#install-clients">client software</a> as well! Available on a variety of desktop and mobile platforms.<br>
If you're ever stumped be sure to also check out their docs: <a href="https://docs.nextcloud.com/server/30/admin_manual/index.html">docs.nextcloud.com</a>
