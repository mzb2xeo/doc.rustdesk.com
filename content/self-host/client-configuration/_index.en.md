---
title: Client Configuration
weight: 300
pre: "<b>2.3. </b>"
---

### Overview

There are a number of ways to configure RustDesk Clients to use your own self-hosted server, we will cover some below.

### 1. Custom client generator (Pro only, basic plan or custom plan)

You can have your own name, logo, icon, configuration, be signed and more.

Currently, Windows X64, Mac Arm64 / X64, [Linux](https://twitter.com/rustdesk/status/1788905463678951787), Android Arm 64 are supported.

[Video](https://twitter.com/rustdesk/status/1769171628426944539)

![](images/custom-client-qs.png)
![](images/web_console_custom_client_config.jpeg)

### 2. Manual Config

In the main RustDesk Client home click on the Menu button [ &#8942; ] next to your ID then click on Network, you can now unlock the settings using elevated privileges and set your ID, Relay, API and Keys.

![](/docs/en/self-host/client-configuration/images/network-config.png)

Enter the `hbbs` host or IP Address in the **ID Server** input box (local side + remote side). The other two addresses can be left blank, RustDesk will automatically deduce (if not specially set), and the Relay Server refers to `hbbr` (port 21117).

e.g.

```nolang
hbbs.example.com
```

or

```nolang
hbbs.example.com:21116
```

#### Set `Key`

In order to establish an encrypted connection to your self-hosted server, you need to enter its public key. The key is usualy generated on the first run of `hbbs` and can be found in the file `id_ed25519.pub` in your working directory / data folder.

As a `Pro` user you will additionally be able to retrieve the `Key` from the [web console](https://rustdesk.com/docs/en/self-host/rustdesk-server-pro/console/).

![](/docs/en/self-host/rustdesk-server-pro/console/images/console-home.png?v2)

#### Set `API Server`

This is for `Pro` user only. When you can log in on web console, but fail to log in on RustDesk client, it probably you have not set `API Server` correctly.

If your API Server does not run on default `21114` port (you may not add this port to firewall if you come from open source version), please specify `API Server` explicitly.
e.g. your API Server runs on default HTTPS port, please specify `API Server` with `https://hbbs.example.com`.

If you still can not confirm the value of `API Server`, please go to the welcome page of web console, the `API Server` is shown in above picture (The input box with `API:` label).

### 3. Setup Using Import or Export

1. Use the steps [above](https://rustdesk.com/docs/en/self-host/client-configuration/#manual-config) to configure RustDesk Client on a Device.
2. Using the above machine go to Settings then Network and unlock.
3. Click on `Export Server Config`.
4. Paste the copied string into Notepad or similar.
5. Go to new client, copy the above to clipboard.
6. Go to Settings then Network in RustDesk Client, unlock and click `Import Server Config`.
7. It will automatically paste the settings in.
8. Click `Apply`.

### 4. Automatic Config

The easiest way to setup automatically is using deployment scripts found [here](https://rustdesk.com/docs/en/self-host/client-deployment/).

### 5, Import config from `Pro` via clipboard

![](/docs/en/self-host/rustdesk-server-pro/console/images/console-home.png?v2)

https://github.com/rustdesk/rustdesk-server-pro/discussions/372#discussioncomment-10473298

### 6. Use command line `--config`
`rustdesk.exe --config <config-string>`

You can get the config string from web console (you can see it on above picture) or from RustDesk client "Settings → Network" ([here](https://github.com/rustdesk/rustdesk/discussions/7118) is a discussion about this).

### 7. WINDOWS `.exe` with public key 

**Step 1: Retrieve and Display the Public Key**

 - To get the public key of your server, run the following command:
 ```bash
 cat /var/lib/rustdesk-server/id_ed25519.pub; echo
 ```  
    
 - This will display the public key, which is used by clients to connect to your server. In this example, the output is:

   `kMtF+J8VMGEk2iLmUQ6AbVDkVRKce9r+p8n4YuQfl+c=`

***Note: Files `id_ed25519` and `id_ed25519.pub` are used to encrypt communication between clients and servers. If you want to generate a new keypair, delete these two files and restart your services. Otherwise, use the existing ones.***

**Step 2: Obtain the RustDesk Client**

 - Download the latest RustDesk client from the official GitHub repository: [https://github.com/rustdesk/rustdesk/releases/latest](https://github.com/rustdesk/rustdesk/releases/latest)

**Step 3: Configure the Client to Connect to Your Server**

 - To connect your client to your server, you'll need to rename the downloaded `.exe` file to include the server's IP address or URL and the public key.

 - For example, if your server's IP address is `192.168.1.243` and the public key from above,  the file name should be:

`rustdesk-host=192.168.1.243,key=kMtF+J8VMGEk2iLmUQ6AbVDkVRKce9r+p8n4YuQfl+c=.exe`  

**Step 4: Run the Client**

 - After renaming the `.exe` file, run the client by double-clicking on it. The client window should appear and any other clients should show up in the client automatically. 

**Troubleshooting Tips**
    * Make sure you have the correct public key displayed in the `id_ed25519.pub` file.
    * Verify that the downloaded `.exe` file has the correct name and format (e.g., `rustdesk-host=<IP or URL>,key=<public_key>.exe`).  
