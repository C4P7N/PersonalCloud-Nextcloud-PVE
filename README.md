# Personal Cloud via Nextcloud in an LXC w/ Proxmox VE
This guide assumes you already have Proxmox VE installed.
### Why would I use Nextcloud and Proxmox?
  Nextcloud is an open source client and server software that allows you to host your own collaboration efforts without relying on third party options. From file hosting, an office suite, chat and more, Nextcloud is like having your own self-hosted GSuite or Microsoft Teams where you're in full control. Hosting Nextcloud on our own Proxmox server furthers that control and eliminates any reliance on third parties. Nextcloud on PVE gives us full control on how our files are kept without the worry of a third party getting involved potentially shutting us down without reason or misusing our content for whatever their many reasons may be.

# Getting Started
To set up Nextcloud on a Debian LXC in Proxmox VE, first we'll create our container, update Debian, and install Apache, MariaDB, and PHP, Download and unpack Nextcloud into the web server directory, Setup MariaDB by creating and configuring a database for Nextcloud, then configure Apache with necessary modules. Finish by accessing Nextcloud through a browser to set it up, and set SSL for security.

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
