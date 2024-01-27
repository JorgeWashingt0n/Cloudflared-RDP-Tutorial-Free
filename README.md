# Setting up **RDP** _and_ **SSH**

## ( _via_ **Cloudflare Tunnels** on the **Free Plan** )
  
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Procedure](#procedure)
- [Part 1: Downloading 'cloudflared.exe'](#part-1-downloading-cloudflaredexe)
- [Part 2: Creating a Cloudflare Tunnel](#part-2-creating-a-cloudflare-tunnel)
  - [1. Start the Login Process](#1-start-the-login-process)
  - [2. Handle Redirection Bug](#2-handle-redirection-bug)
  - [3. Authorize Your Domain](#3-authorize-your-domain)
  - [4. Certification File Download](#4-certification-file-download)
  - [5. Navigate to the .cloudflared Folder](#5-navigate-to-the-cloudflared-folder)
  - [6. Create Your Tunnel](#6-create-your-tunnel)
- [Part 3: Configuring the Tunnel and Creating Subdomains  
\( This is where things usually get hairy... \)](#part-3-configuring-the-tunnel-and-creating-subdomains)
  - [1. Download and Open the Template](#1-download-and-open-the-template)
  - [2. Update the Tunnel ID](#2-update-the-tunnel-id)
  - [3. Set Global Options](#3-set-global-options)
  - [4. Decide on Subdomains](#4-decide-on-subdomains)
  - [5. Configure Service Blocks](#5-configure-service-blocks)
  - [6. Create DNS Records](#6-create-dns-records)
- [RDP \& SSH, what you've all been waiting for...](#part-4-rdp--ssh-what-youve-all-been-waiting-for)
  - [RDP](#rdp)
  - [SSH](#ssh)
  - [TODO](#todo)

## Introduction

**Welcome** to this step-by-step guide on setting up **RDP (Remote Desktop Protocol)** and SSH **(Secure Shell)** access through Cloudflare's free tier. I dedicated an enourmous amount of time trying to figure it out, even sometimes thinking it was impossible, but I never gave up... _and I finally did it._ Contrary to some misconceptions, it is entirely possible to configure RDP under the free tier. This guide aims to demystify the process, providing clear and straightforward instructions to help you succeed where others (and myself) may have encountered difficulties. To note, I am using a Windows computer to do all of this, but I imagine it would be pretty much the same from any platform, as the only parts on our machine that we configure  are .yaml files.

To simplify the process further, I've included templates for the configuration file, in addition to some simple scripts. These templates are designed for ease of use; you'll only need to paste your specific information into them, such as your domain and hostname information, (for example . Additionally, I've prepared a few dedicated scripts to enhance your experience. These scripts include one for starting the tunnel effortlessly—eliminating the need to remember complex command-line arguments—and another that allows this starter script to run as a hidden background service via Task Scheduler. This ensures that the tunnel operates seamlessly in the background, without interfering with your regular computer use, such as when closing multiple open Chrome tabs.

Many online resources, including forums and official documentation, can be challenging to navigate and understand. This guide cuts through the complexity, offering an easy-to-follow approach to setting up RDP and SSH by manually configuring your Cloudflare tunnels.
Towards the end, we will also explore advanced topics such as automating the setup process and running it in the background without a visible window, enhancing both convenience and functionality

## Prerequisites

In order to successfully set up RDP/SSH access through Cloudflare tunnels, there are a few prerequisites you'll need to meet:

   1. Domain Requirement: Having a domain name is essential. This can be a free domain, such as one provided by a Dynamic DNS (DDNS) service like DuckDNS, or a domain you own. The key requirement is the ability to set DNS records for the domain, which you will later import into Cloudflare.

   2. Use Cloudflare for DNS: Ensure that your domain, whether free or owned, uses Cloudflare as its DNS provider. This enables the creation of tunnels through Cloudflare's network.

   3. Get started with Cloudflare Tunnels: It's beneficial to reach the point in Cloudflare's tutorials where you're able to create tunnels via their dashboard. Once you're comfortable with this, switch to this guide for a more detailed and manual setup process.

   4. Create a Dedicated Folder: On your local machine, make a folder in your home directory \n (C:\\Users\\\<your username>) named '.cloudflared' (including the period at the beginning). This folder will be used for storing configuration files and other necessary data for the Cloudflare tunnels."

## Procedure

### Part 1: Downloading 'cloudflared.exe'

[Go back to top](#setting-up-rdp-and-ssh)

Follow these steps to download and prepare cloudflared on your machine:

   1. Download Cloudflared: Visit the Cloudflare/cloudflared GitHub repository and download the latest release. Make sure to download the .exe file, not the MSI installer.

   2. Create a Main Folder: On your C drive, create a new folder named 'Cloudflared'. The path should be C:\\Cloudflared.

   3. Create a Subfolder for Executables: Inside the 'Cloudflared' folder, create another folder named 'bin'. The full path will be C:\\Cloudflared\\bin.

   4. Place the Executable: Move the downloaded .exe file into the 'bin' folder.

   5. Rename the Executable: Rename the file to 'cloudflared.exe', removing any additional characters or version numbers.

   6. Add to PATH: Now, it's time to add this folder to your system's PATH for easy access.

   7. Access Environment Variables: Type 'env' in the start menu and select 'Edit the system environment variables'. In the System Properties window, click 'Environment Variables...' at the bottom right.

   8. Edit PATH Variable: In the 'Environment Variables' window, under 'System variables', find the 'PATH' variable. Select it and click 'Edit'. Add a new entry with the path C:\\Cloudflared\\bin. Confirm by clicking 'OK' on all open windows.

## Part 2: Creating a Cloudflare Tunnel  

[Go back to top](#setting-up-rdp-and-ssh)

Next, we will get the tunnel started, which will allow us to later use our own rules, instead of the limiting dashboard that cloudflare offers.

### 1. Start the Login Process

- Open your terminal (Command Prompt, PowerShell, etc.) and enter cloudflared tunnel login. This command will prompt your web browser to open for you to log in to your Cloudflare account

#### 2. Handle Redirection Bug

 If you're redirected to your Cloudflare homepage instead of the authorization page, return to your terminal. Ctrl+Click the link provided by the cloudflared tunnel login command again. Now that you're logged in, it should redirect you correctly to the 'Authorize Cloudflare Tunnel' page.

#### 3. Authorize Your Domain

 Select the domain you want to use for accessing your tunnels and click 'Authorize'. Note that you can add multiple subdomains to this later.

#### 4. Certification File Download

 This step automatically downloads a cert.pem file into the .cloudflared folder in your user home directory `(C:\\Users\\<your username>\\.cloudflared)`'.

#### 5. Navigate to the .cloudflared Folder

- Use the terminal or the graphical user interface (GUI) to navigate to the .cloudflared folder. If using the GUI, you can right-click in the folder and select "Open Terminal here" or "Open PowerShell here".

#### 6. Create Your Tunnel

- In the terminal, type cloudflared tunnel create \<choose a name>. This command uses the cert.pem file in your current directory to authenticate and registers a tunnel with the name you provide. It doesn't start the tunnel but prepares it for use. A unique GUID (a string of random numbers, letters, and dashes) will be assigned to your tunnel. Additionally, a configuration file named \<GUID>.json will be downloaded into the .cloudflared folder. This file is crucial for authenticating and running the tunnel.

```markdown
Tips:
- If you encounter redirection issues during the login process, the workaround mentioned in step 2 should help.
-  The name you choose for your tunnel can be anything descriptive and meaningful to you

```

## Part 3: Configuring the Tunnel and Creating Subdomains

### ( This is where things usually get hairy... )

[Go back to top](#setting-up-rdp-and-ssh)

#### 1. Download and Open the Template  

- Access the GitHub repository for the configuration template. Download the template, I recommend in the .cloudflared folder in your home directory, and open it in a text editor like Notepad++.

##### 2. Update the Tunnel ID

- Find the `tunnel: parameter` in the template.
- Replace its value with the GUID of your tunnel.  
  - An easy way to get the GUID is by copying the name of the .json file (excluding the '.json' part).  
  - For the credential-file: parameter, add the full path of the .json file.  

#### 3. Set Global Options

- One the line below `credential-file:<####-AAAA-#####.json>`, add any global options you desire, such as noTLSverify. See template file for exact layout.

#### 4. Decide on Subdomains  

- Choose subdomains for your services, like plex.yourdomain.com or RDP.yourdomain.com, pihole.yourdomain.com, etc. You don't ahve to name it after your services, it could be scroogemcduck.example.com, as long as you send it to the desired place in the configuration file.

#### 5. Configure Service Blocks

Use the provided blocks in the template as a guide.

- Change `hostname:` to your chosen subdomains.
- Under `service:`, specify the local hostname or IP address and the port of the service.

  - Example: For a web service, use `http://192.168.1.10:80`.
  - For SSH, format it as `ssh://\<IP>:\<Port>`.
  - For RDP, use `rdp://\<IP>:\<Port>`.

Indentation is Key: Ensure proper indentation in the configuration file. The dash in `- hostname:` should be two spaces from the margin, and the 's' in  `service:` four spaces in.

#### 6. Create DNS Records

- Log in to the Cloudflare dashboard and navigate to the DNS Rules page.
- Set up CNAME records for each subdomain. For the target, use `<GUID OF TUNNEL>.cfargotunnel.com`.
This redirects traffic to your tunnel instead of your personal IP. Again, you can get this from the name of the .json file if you forget it.

```markdown
# **Special Steps for RDP and SSH:**

For both RDP and SSH, you need cloudflared.exe on the client machine.
Simply download the .exe file and place it in a folder on the client machine, such as C:\Cloudflared\bin.
Optionally, add this folder to the PATH for convenience (but seriously do it).

# **Notes:**

For wildcard subdomains (like *.yourdomain.com), enclose the URL in quotes to avoid errors.
Example: hostname: '*.example.com'
For HTTP/S services, once DNS records are set, you can access them globally as if on your local network.
```

## Part 4: RDP & SSH, what you've all been waiting for

`RDP AND SSH BOTH HAVE SPECIAL ADDITIONAL STEPS TO TAKE ON THE CLIENT MACHINE BEFORE YOU ATTEMPT TO CONNECT.`

For both cases, you need the cloudflared.exe program on the client machine. You DO NOT need to install anything, eg. the MSI version, or install as a service, or anything like that. Just download the .exe and put it into a folder. For consistency and ease of use, I would suggest to make a C:\Cloudflared\bin folder on your client machine(s) and add it to the PATH as well, but it's up to you.

### RDP

This one was tricky and took me a while to figure out how to use. I read it in the documentation multiple times but how they have it made absolutely no sense to me. Before you try to connect to anything with RDP, you need to run the following command in a terminal window (and you need to run it each time before you RDP to these machines.) Under hostname put the subdomain you set up for RDP, and under URL leave it exactly as it is, unless your RDP is running on something other than the default port, in which case you would put that.

```PowerShell
cloudflared access rdp --hostname rdp.yourdomain.com --url rdp://localhost:3389  
```

When you execute this command, it should say `INF Start Websocket listener host=localhost:<your port>`. Now, another unintuitive thing, you open the RDP client and then CONNECT TO LOCALHOST:3389.

That's right, you don't put the subdomain you set up, you don't put an IP address, you RDP TO YOURSELF!!

But due to the command we just ran, it redirects you automagically to the device you set up in the configuration file. If you set up Cloudflare Access for authentication, at this point it would open a browser window and prompt you to authenticate before making the connection. And that's all there is to it! I was banging my head against the wall for weeks before I figured it out.

### SSH

In order to get SSH to work, you must create a "config" file in your ".ssh" folder in your home directory (C\Users\<username>\.ssh). No extension, just create a text file named "config" and then erase the extension. You want to follow this format, substituting the host subdomain for whatever you named yours (NOTE: you do not need to name your SSH subdomain 'ssh', just for illustrative purposes), having the correctly formatted "service:" that your subdomain points to in the config.yml file on your server is the important part.

```SSH
Host ssh.yourdomain.com
  ProxyCommand "C:\Program Files (x86)\cloudflared\cloudflared.exe" access ssh --hostname %h
```

You can add add additional entries such as this below if you have multiple subdomains pointing to SSH, for example if you have multiple devices and have a separate ssh subdomain set up for each.

Now, to connect, you just connect as normal, for example "ssh <tsmith@ssh.yourdomain.com>" in any terminal.  

### TODO

  [ ] Upload Templates
  [ ] Edit Final Section to Match Formatting
