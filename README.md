# jitsi-meet



### Jitsi Installation:
Os : Ubuntu 22.04

### Step 1 — Setting the System Hostname
```
sudo hostnamectl set-hostname jitsi.your_domain
```
ex : ```sudo hostnamectl set-hostname jitsi.idrbt.com```

Check that the hostname was set with the following command:

```hostname```

Next, you will set a local mapping of the server’s hostname to its public IP address. Do this by opening the /etc/hosts with nano or any text editor:

```sudo nano /etc/hosts```

Then, add the following line under the 127.0.0.1 localhost line:

```
<public_ip> <jitsi.your_domain>
```
ex: ```192.168.x.x jitsi.idrbt.com```

Save and close the file.

#### Step 2 — Configuring the Firewall


 The Jitsi server needs some additional ports open to communicate with the call participants. The TLS installation process also requires a port open to authenticate the certificate registration request.
 
 Run the following commnads
```
sudo ufw enable
```

 ```
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3478/udp
sudo ufw allow 5349/tcp
sudo ufw allow 10000/udp
```
```
sudo ufw status
```

#### Step 3 — Installing Jitsi Meet

You will now add the Jitsi and the Prosody APT repositories to your server. Prosody is an open-source XMPP chat server that Jitsi uses for messaging and admin authentication. You will then install the Jitsi Meet package from its repository, which will ensure that you are always running the latest stable Jitsi Meet package

First, download the Jitsi GPG key with the curl downloading utility:

```
curl https://download.jitsi.org/jitsi-key.gpg.key -o jitsi-key.gpg.key
```

Next, add the GPG key to your system’s keyring with the following gpg command:
```
sudo gpg --output /usr/share/keyrings/jitsi-key.gpg --dearmor jitsi-key.gpg.key
```

You will now add the Jitsi repository to your server by creating a new APT sources file that contains the Jitsi repository. 

Open and create the new file with the following command:
```
sudo nano /etc/apt/sources.list.d/jitsi-stable.list
```
Add this line to the ```/etc/apt/sources.list.d/jitsi-stable.list``` file:
```
deb [signed-by=/usr/share/keyrings/jitsi-key.gpg] https://download.jitsi.org stable/
```
Save and exit the editor.

Next, you will follow the same steps to add the Prosody package. Download the Prosody GPG key:
```
curl https://prosody.im/files/prosody-debian-packages.key -o prosody-debian-packages.key
```
Then, add the key to your server’s keyring:
```
sudo gpg --output /usr/share/keyrings/prosody-keyring.gpg --dearmor prosody-debian-packages.key
```
Next, open a new sources file for Prosody:
```
sudo nano /etc/apt/sources.list.d/prosody.list
```
Add the following line to the currently empty Prosody sources file:
```
deb [signed-by=/usr/share/keyrings/prosody-keyring.gpg] http://packages.prosody.im/debian jammy main
```
Save and exit the editor.

You can now delete the GPG keys that you downloaded as they are no longer needed:

```
rm jitsi-key.gpg.key prosody-debian-packages.key
```

Finally, perform a system update to collect the package list from the new repositories and then install the jitsi-meet package:

```
sudo apt update
sudo apt install jitsi-meet -y
```

During  the installation of jitsi-meet you will be prompted to enter the domain name (for example, jitsi.idrbt.com) that you want to use for your Jitsi Meet instance.


![domain](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/b29b0fc1-d8d8-460f-b405-9f1fa0065181)


You will then be shown a new dialog box that asks if you want Jitsi to create and use a self-signed TLS certificate or use an existing one:

![domain1](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/61aa6286-7dd0-4144-ae77-5f98ce53052a)


f you do not have a TLS certificate for your Jitsi domain, select the Generate a new self-signed certificate option.

Your Jitsi Meet instance is now installed using a self-signed TLS certificate. You will receive browser warnings if you don’t yet have a TLS certificate, so you will get a signed TLS certificate in the next step.

#### Step 4 — Obtaining a Signed TLS Certificate

Install certbot with the following command:
```
sudo apt install certbot -y
```
Jitsi Meet supplies a script to download a TLS certificate for your domain automatically.

Run this certificate installation script provided by Jitsi Meet at ```/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh``` with the following command:

```
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```
The script prints the following information when you run it and asks you to supply an email address:

Give email address and press ENTER.

The script will complete the installation and configuration of an SSL certificate for your Jitsi server without any more user input.

The default configuration for Jitsi Meet is that anyone visiting your Jitsi Meet server homepage can create a new conference room. 

This default mode will use your server’s system resources to run the conference room and is not desirable for unauthorized users.

In the next step, you will update those settings.

#### Step 5 — Locking Conference Room Creation

You will now configure your Jitsi Meet server to allow only registered users to create conference rooms.


First, open ```/etc/prosody/conf.avail/jitsi.your_domain.cfg.lua``` with a text editor:

The variable ```jitsi.your_domain``` will be used in place of your domain name

```
sudo nano /etc/prosody/conf.avail/your_domain.cfg.lua
```

edit the following line ```authentication = "anonymous"```
```
authentication = "internal_plain"
```

This configuration tells Jitsi Meet to force username and password authentication before allowing conference room creation by a new visitor.

In the same file, add the following section to the end of the file:
```
...
VirtualHost "guest.jitsi.your_domain"
    authentication = "anonymous"
    c2s_require_encryption = false
    modules_enabled = {
            "bosh";
            "ping";
            "pubsub";
            "speakerstats";
            "turncredentials";
            "conference_duration";
    }
```

This configuration allows any user to join conference rooms that an authenticated user created. However, the guest must have the unique URL (and an optional password) to enter it.

Here, you added guest. to the front of your domain name. For example, the correct name to put here for jitsi.your_domain is guest.jitsi.your_domain. The guest. hostname is used internally by Jitsi Meet, so you will never enter it into a browser or need a separate DNS record for the guest. hostname

When finished, save and close the /etc/prosody/conf.avail/jitsi.your_domain.cfg.lua file.

Next, open another configuration file at ```/etc/jitsi/meet/jitsi.your_domain-config.js``` with a text editor:

```
sudo nano /etc/jitsi/meet/jitsi.your_domain-config.js
```
Use the search tool CTRL+W again to search for anonymousdomain:, which will take you to the following line:

```  // anonymousdomain: 'guest.example.com',```

Edit this line to look like the following by removing the double slashes // at the start of the line:

```
anonymousdomain: 'guest.jitsi.your_domain',
```
You will use the guest.jitsi.your_domain hostname that you used previously. This configuration tells Jitsi Meet what internal hostname to use for the un-authenticated guests. Save and close the file

Next, create and open ```/etc/jitsi/jicofo/sip-communicator.properties```:
```
sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```

Add the following line to complete the configuration changes:
```
org.jitsi.jicofo.auth.URL=XMPP:jitsi.your_domain
```

This configuration points one of the Jitsi Meet processes to the local server that performs the user authentication that is now required.

When finished, save and close the file.

Now that Jitsi Meet is configured to require authenticated users for room creation, you need to register these users and their passwords. You will use the prosodyctl utility to do this.


Run the following command to add a user to your server:
```
sudo prosodyctl register <user> <your_domain password>
```
here give username and password.


The user that you add here is not a system user. They will only be able to create a conference room and cannot log in to your server via SSH.


When you run this command, you may see the following warning:

```general             warn        Lua 5.1 has several issues and support is being phased out, consider upgrading```


Finally, restart all the Jitsi Meet processes to load the new configuration:

```
sudo systemctl restart prosody.service jicofo.service jitsi-videobridge2.service
```

Your Jitsi Meet server is now fully installed, configured, and running. You will create a new conference room in the final step.


Step 6 - Opening a Conference Room and Inviting Participants

You can now browse to and start using your new Jitsi Meet server. Open your browser and enter your Jitsi domain name (with https) into the address bar.

ex: ```https://jitsi.idrbt.com```

To start a meeting, press the Start Meeting button on the landing page:

![domain2](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/8e9811d7-3ba1-4e23-b6bd-2f8f01308735)


When you click the Start Meeting button, it will begin a new meeting. You will immediately be asked if you are the host (the admin of the meeting) in the following dialog box:

![domain3](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/a528d575-4dbb-41e8-8d39-50596aca2a7a)


You must click on the I am the host button to proceed. After you click on that button, you will be prompted to enter your admin username and password in the following field:


![domain4](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/27233f1b-8de6-413d-b343-17f76da58560)



The username and password are the ones that you set in Step 5 with the prosodyctl utility. Enter the username and password and click Login, which will create the new meeting and set you as the moderator.

Now that the meeting is open and running, you can invite participants. Click on the participants icon in the bottom panel:


![domain5](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/eab99708-326a-49c7-a881-d85ce3f94d25)


This action opens a right-hand panel. Click on the Invite Someone button:


![domain6](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/152eee4d-a9c2-43d3-8653-3eb5891c67ca)


This action will open the Invite more people dialog. Copy the unique URL of the meeting by clicking on the copy icon:
![domain7](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/27cb67fb-061f-4f55-b4fa-c7f94f4acce8)


You can now paste the meeting URL into an email or chat program to send to anyone you want to attend the meeting. They will not need a username and password to access the meeting after you have created it.




### Jibri setup:

#### Jibri installation steps :

Install the necessary packages from Debian 10’s default repositories.

```
sudo apt install -y software-properties-common ffmpeg curl alsa-utils icewm xdotool unzip xserver-xorg-video-dummy ruby-hocon maven gnupg2 make nginx-full apt-transport-https ca-certificates
sudo apt -y install linux-virtual linux-cloud-tools-virtual linux-generic linux-headers-generic
```
If you are using any cloud verify for the linux type. Soo if it is azure 
```sudo apt -y purge linux*azure```
```
sudo reboot
```
after reboot verify the vm type

```
uname -a #it should be generic
```
```
sudo apt update && sudo apt upgrade -y  && sudo apt purge && sudo apt autoremove && sudo apt autoclean
```

Enable the ALSA loopback module
```
echo “snd_aloop” >>/etc/modules
modprobe snd_aloop
```

If u get permission issue try using super user "sudo su" 

Download and install Google Chrome :

```
sudo su
sudo curl -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add
sudo echo deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main > /etc/apt/sources.list.d/google-chrome.list
sudo apt update
sudo apt install google-chrome-stable
```

Install latest ChromeDriver :
https://googlechromelabs.github.io/chrome-for-testing/

execute below commands as super user if you get any issue exit the super user

ex:
```
wget -N https://edgedl.me.gvt1.com/edgedl/chrome/chrome-for-testing/120.0.6099.109/linux64/chromedriver-linux64.zip
```
```
unzip ~/chromedriver-linux64.zip -d ~/
rm ~/chromedriver-linux64.zip
sudo mv -f chromedriver-linux64/chromedriver /usr/local/bin/chromedriver
sudo chmod 0755 /usr/local/bin/chromedriver
```

Disable notifications regarding scripted control in Chrome :

```
mkdir -p /etc/opt/chrome/policies/managed
echo '{ "CommandLineFlagSecurityWarningsEnablead": false }' >>/etc/opt/chrome/policies/managed/managed_policies.json
```
Install Jibri:
From here exit super user

```
sudo apt-get install jibri
```

Include Jibri’s user account in the required groups :

```
usermod -aG adm,audio,video,plugdev jibri
```

Set up Prosody configuration :

```sudo nano /etc/prosody/prosody.cfg.lua```

Add below lines at the end of the page

```
Component "internal.auth.yourdoimain.com" "muc"
modules_enabled = {
"ping";
}
storage = "internal"
muc_room_cache_size = 1000


VirtualHost "recorder.yourdomian.com"
modules_enabled = {
"ping";
}
authentication = "internal_plain"
```

here give your domain instead ```yourdoimain.com```

```
sudo prosodyctl check config
```

If you get any errors comment :

```sudo nano /etc/prosody/conf.d/yourdomain.com.cfg.lua```

```comment (cross_domain_bosh)```

Create two new Prosody user accounts for Jibri—one for control purposes and another for recording purposes.

```
sudo prosodyctl register jibri auth.yourdomain.com <password>
sudo prosodyctl register recorder recorder.yourdomain.com <password>
```

Modify Jicofo configuration :

```sudo nano /etc/jitsi/jicofo/sip-communicator.properties```

Add below 2 lines in above location

```
org.jitsi.jicofo.jibri.BREWERY =JibriBrewery@internal.auth.yourdomain.com
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```


Update configuration file for recording :

```sudo nano /etc/jitsi/meet/yourdomain.com-config.js```

In above location uncomment the recording and add hiddenDomain
![jibri](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/56d76880-f4b3-4be8-9e5d-ac106fa64de0)

Add hiddenDomain :

![jibri1](https://github.com/5g-ucl-idrbt/jitsi/assets/101712802/f9c2bb68-17d5-45d2-8112-d0e2f4338547)



Create a directory for storing recordings and adjust its permissions accordingly :

```
cd /opt
mkdir recordings
sudo chown jibri:jibri recordings
```

Customize Jibri settings by configuring the following options :

```sudo nano /etc/jitsi/jibri/jibri.conf```

In above locatiion add below lines

```
jibri {
  id = "Unique identifier"
  single-use-mode = false
  api {
    http {
      external-api-port = 2222
      internal-api-port = 3333
    }
    xmpp {
      environments = [
          {
                name = "prod environment"
                xmpp-server-hosts = ["yourdomain.com"]
                xmpp-domain = "yourdomain.com"

                control-muc {
                    domain = "internal.auth.yourdomain.com"
                    room-name = "JibriBrewery"
                    nickname = "jibri-zcsdfsfsfsf"
                }

                control-login {
                    domain = "auth.yourdomain.com"
                    username = "jibri"
                    password = <password>
                }

                call-login {
                    domain = "recorder.yourdomain.com"
                    username = "recorder"
                    password = <password>
                }

                strip-from-room-domain = "conference."
                usage-timeout = 0
                trust-all-xmpp-certs = true
            }]
    }
  }
  recording {
    //"/opt/recordings"
    recordings-directory = <recording_directory> 
    finalize-script = ""
  }
  streaming {
    // A list of regex patterns for allowed RTMP URLs.  The RTMP URL used
    // when starting a stream must match at least one of the patterns in
    // this list.
    rtmp-allow-list = [
      // By default, all services are allowed
      ".*"
    ]
  }
  chrome {
    // The flags which will be passed to chromium when launching
    flags = [
      "--use-fake-ui-for-media-stream",
      "--start-maximized",
      "--kiosk",
      "--enabled",
      "--disable-infobars",
      "--autoplay-policy=no-user-gesture-required"
    ]
  }
  stats {
    enable-stats-d = true
  }
  webhook {
    // A list of subscribers interested in receiving webhook events
    subscribers = []
  }

  call-status-checks {
    // If all clients have their audio and video muted and if Jibri does not
    // detect any data stream (audio or video) coming in, it will stop
    // recording after NO_MEDIA_TIMEOUT expires.
    no-media-timeout = 30 seconds

    // If all clients have their audio and video muted, Jibri consideres this
    // as an empty call and stops the recording after ALL_MUTED_TIMEOUT expires.
    all-muted-timeout = 10 minutes

    // When detecting if a call is empty, Jibri takes into consideration for how
    // long the call has been empty already. If it has been empty for more than
    // DEFAULT_CALL_EMPTY_TIMEOUT, it will consider it empty and stop the recording.
    default-call-empty-timeout = 30 seconds
  }
}
```

chnages are in above file yourdomain.com and password like "password"



Reboot all services, activate, and initiate Jibri :


```
sudo systemctl restart prosody
sudo systemctl restart jicofo
sudo systemctl restart jitsi-videobridge2
sudo systemctl enable --now jibri
```





























Reference:

https://www.digitalocean.com/community/tutorials/how-to-install-jitsi-meet-on-ubuntu-22-04


https://medium.com/@varimallashankar/install-jibri-for-debian-10-ubuntu-22-04-417a50ad0b1d

