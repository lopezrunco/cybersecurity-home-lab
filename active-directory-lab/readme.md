# Active Directory Lab

## Project description

In short, we can think of **Active Directory** as *Database* that contains users, computers, groups and more.
In order to use Active Directory, a server must install a service called **Active Directory Domain Services** (ADDS), and the server must then be promoted to a **Domain controller** (DC), by doing this, it will allows to perform **Authentication** using a protocol called **Kerberos** and **Authorization**.

I will be using:

- Target machine (Windows 10)
- Attacker machine (Kali Linux)
- Active Directory (Windows Server 2022)
- Splunk (Ununtu Server)
- Virtual Box

## Download and install VirtualBox

Go to the [Virtual box download page](https://www.virtualbox.org/wiki/Downloads) and click on **Windows hosts**.

![virtualbox-download-page.png](../basic-home-lab/assets/virtualbox-download-page.png)

While it’s downloading, go ahead and check the `SHA256 checksums` to verify that the downloaded file has not been altered. To do this, go to your `Downloads` folder (or wherever you saved the installer), open a PowerShell console, and type:

```powershell
    Get-FileHash .\VirtualBox-7.1.10-169112-Win.exe  
```

This will return a SHA256 hash. Copy the hash, go back to the `SHA256 checksums` page on the VirtualBox website, and check if the hash matches one of the listed values (use Ctrl + F and paste the hash to search).

If the hash matches, you can be confident that the file was not altered during download. Now you can proceed with the installation of VirtualBox—just follow the instructions in the setup wizard and you should be good to go.

## Install Windows 10

Go to [this link](https://www.microsoft.com/en-ca/software-download/windows10), scroll down to the **Create Windows 10 installation media** section and click **Download now**. This will download the Media Creation Tool, which will help us create the Windows 10 image file.

Once you run the tool, on the **What do you want to do?** screen, select **Create installation media**, then choose **ISO file**.

![Create installation media](../basic-home-lab/assets/create-installation-media.png)

After downloading the Windows 10 ISO file, open VirtualBox and create your first virtual machine.

Click on **New**, and the **Create Virtual Machine** wizard. wizard will appear. Name the machine `windows10`, select the ISO image, and check the box **Skip unattended installation** — this allows us to install the operating system manually. 

Click **Next** to view the virtual machine specifications.

![Install windows10](../basic-home-lab/assets/install-windows10.png)

Note that these settings will depend on your computer's specifications. In this example, I’ll assign **2048 MB** of base memory and **1 CPU** under the processor settings.

![Windows 10 Virtual machine hardware](../basic-home-lab/assets/hardware.png)

For the virtual hard disk, I’ll leave it at **50 GB** and click **Next**.

![Windows 10 Virtual machine Virtual hard disk](../basic-home-lab/assets/virtual-hard-disk.png)

Next, the wizard will show you a summary of your virtual machine settings. If everything looks good, click **Finish**.

To power on your Windows 10 machine, simply click **Start**. You should now see the Windows 10 setup screen.

![](../basic-home-lab/assets/windows-10-setup.png)

Click on **Install**, and when you reach the **Activate Windows** screen, click **I don't have a product key**. Then, from the list of options, select **Windows 10 Pro** and click **Next**. Accept the license terms.

On the next screen, choose **Custom: Install Windows only (advanced)**, select the drive, and click **Next**. Windows 10 should now begin installing in the background.

## Install Kali Linux

First, navigate to the [Kali website](https://www.kali.org), click on **Download**, and then select the **Pre-built VMs** menu item. I’ll be downloading the 64-bit version of Kali, but you should choose the option that matches your system architecture.

![Download Kali Linux](../basic-home-lab/assets/download-kali.png)

A `.7z` file will be downloaded, so you’ll need **7-Zip** to extract its contents. Once decompressed, look for the file with the `.vbox` extension and double-click it. Kali Linux should be automatically imported into VirtualBox.

Now you can start the Kali virtual machine.  
**Note:** The default credentials for the Kali Linux machine are `kali/kali`.

![Start Kali Linux](../basic-home-lab/assets/start-kali-linux.png)

## Install Windows Server 2022

I downloaded Windows Server from [Microsoft Evalatuion Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) website, selecting the ISO file.

Once in Virtual Box, I named the machine `ADDC01`, added the ISO image, left all the settings as default but check in **Skip Unattended Installation** because I do not want Virtual Box to automatically install it. In the **Hardware** tab I setted the **Base Memory** to 4GB, and that's all the configuration so far.

Once in the virtual machine I choose the **Windows Server 2022 Standard Evaluation (Desktop Experience)** version of the OS. 

In the **Type of installation** screen I selected **Custom** and click **Next** until the installation started.

After the set up was completed, I was presented with the **Customize settings** screen, where I created a password and then a **Login** screen to Enter the credentials I just created. Once in the Desktop, the **Server Manager** will open automatically. 

## Install Splunk Server

I downloaded the [Ubuntu Server](https://ubuntu.com/server) **22.04.5 LTS version**. Once in VirtualBox, I created a new machine named **Splunk** with the default configuration but checking **Skip Unattended Installation**, I set the **Base memory** in 4GB, the **Processors** in 2 and the **Virtual Hard disk** in 50GB.

Run the virtual machine and in the presented screen select **Try or Install Ubuntu Server**, continuing with the default options unitl the **Profile setup** screen, there I entered the credentials, hit **Done**. I left the next configurations by default and started the installation.
After it finished, I rebooted the virtual machine. If you are presented with the **Failed unmounting /cdrom** error, just press Enter.

![Error](./assets/splunk-boot-error.png)

After Ubuntu Server finished the reboot, I was presented with a login screen where I entered the credentials I created earlier. After a successfull login, I run the command `sudo apt-get update && sudo apt-get upgrade -y` to update and upgrade all the repositories and after that the server is ready to go.

At this point, I have the four virtual machines ready:

![Machines](./assets/machines.jpg)

## Create a network

Back in Virtual box I checked that the network setting were set to NAT, that way, the virtual machines can be on the same network and still have internet access. Clicking in **Tools** and then **Network**, then selected **NAT networks** and click on **Create**.
I named this network **AD-Network**, set the IPv4 Prefix to `192.168.10.0/24` and checked the **Enable DHCP** option. Then I changed the network settings on every machine, selecting the **NAT Network** attached and selecting the network I created.

## Configure Splunk Server IP

I asigned a static IP to the Splunk Server by running the command `sudo nano /etc/netplan/00-installer-config.yaml`. In this file I indicated that I do not want any DHCP, assigned the address `192.168.10.10/24`, the Google's DNS IP `8.8.8.8` and a default route via `192.168.10.1`, at the end, the fily should look like this:

![00-installer-config.yaml](./assets/splunk-server-yaml-config.png)

Saved the file and run `sudo netplan apply` and to verify the changes `ip a`, the ip address should be `192.168.10.10/24`:

![Splunk server static IP](./assets/splunk-server-static-ip.png)

Check the connection by pinging Google, for example.

## Install Splunk in the Ubuntu server

In the **host machine** I downloaded the Linux version (.deb) of **Splunk Enterprise**. 

Back to the Splunk virtual machine it's time to install the guest add-ons for VirtualBox by running `sudo apt-get install virtualbox-guest-additions-iso`.
After a successfull installation, I added a new Shared folder for the Splunk virtual machine, by selecting `Devices => Shared Folders => Shared Folders Settings` and then `Add new Shared folder`. The **Folder Path** here must be the folder where we put the **Splunk installer**, in my case is named `isos`, and check the options `Read only`, `Auto mount` and `Make Permanent`.

After rebooting the virtual machine, I added a user to the **vboxsf**. First, running `sudo apt-get install virtualbox-guest-utils` and rebooting, and after that we should be able to add the user by running the command `sudo adduser splunk vboxsf`. 

Then, I created a new directory called **share** and then mounted the shared folder onto the new **share** directory, in my case, by running `sudo mount -t vboxsf -o uid=1000,gid=1000 isos share/`:

- `-t vboxsf` specifies the filesystem type, here means **VirtualBox Shared Folder**. 
- `-o uid=1000` sets the **owner** of the mounted files to the user with UID `1000`, in my case the main user.
- `gid=1000` sets the **group** ownership to the group with GID `1000`, usually the same as the main user's primary group. This makes sure that this user, not just root, can read/write the files in the shared folder.
- `share/` is the **mount point**, the directory inside the Ubuntu virtual machine where the contents of `isos` will apear.

Navigating to the `share` folder and running `ls -la` the list of files should look something like this:

![Ubutntu server share folder](./assets/ubutntu-server-share-folder.png)

I installed Splunk running the command `sudo dpkg -i splunk-10.0.0-e8eb0c4654f8-linux-amd64.deb`. After the installation, Splunk is installed uder `/opt/splunk` so I changed to that directory and run `ls -la` to list the content:

![/opt/splunk content](./assets/opt-splunk-content.png)

Notice that all belongs to both the user and group `splunk` (which is good, as limits the permissions to that user and group), so I switched to that user by running `sudo -u splunk bash`.

Now, changed to the directory `bin`, for using one of the the binaries used by Splunk. Type `./splunk start` to run the installer and followed the steps. 

After a successfull installation, I wanted to make sure that **Splunk** starts up every time the virtual machine reboots. I `exit` and changed the directory to `bin` and run the command `sudo ./splunk enable boot-start -user splunk`.

## Install Splunk Universal Forwarder & Cismon on the Target machine

Once in the Windows 10 VM, I will set up a cpuple of things. First, change the host name by hitting Windows => This PC => Properties => Rename this PC. I will name it `target-pc`. Reb0ot and done.

Next, I will change the IP address, in order to follow the stablished diagram. Open **Network and Internet settings** => **Change adapter options** =>  Right click in the adapter and select Properties => Look for Internet protocol v4, and select Properties => And in Use the following IP I will use `192.168.10.100`, a Subnet mask of `255.255.255.0`, a Default gateway of `192.168.10.1` and a Preffered DNS server `8.8.8.8`.

After that, we should be able to open the Splunk server that runs in the port `8000` by typing `192.168.10.100:8000` (The Splunk VM should be running at the same time). The browser should show the Splunk enterprise login page:

![Splunk enterprise login page](./assets/splunk-enterprise-login-page.png)

After that, go to the Splunk website (www.splunk.com), log in with your account, click on **Trials & Downloads** and download **Universal Forwarder**. Once the .msi file is downloaded, open it and run the installer. 

In **Use this UniversalForwarder with:** select the option **An on-premises Splunk Enterprise instance**. I will type `admin` as username and leave the installer to generate a random password.
Skip the **Deployment server** options and in the **Receiving Indexer** is were we will set our Splunk server, so we type `192.168.10.10:9997` and click Install.

Now is time to install **Sysmon** from [this](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) page. We will use Sysmon config from Olaf Hartong by this [repo](https://github.com/olafhartong/sysmon-modular), scrolling down and dowloading the file `sysmonconfig.xml`.

Run a Powershell console with Admin priviledges on the extracted Sysmon folder. Then I will run the Sysmon64.exe file indicating the configuration file we just downloaded, for that I run the command `.\Sysmon64.exe -i ..\sysmonconfig.xml`. Once is installed you should see something like this in the console:

![Sysmon installed](./assets/sysmon-installed.png)

Now we need to instruct **Splunk Forwarder** on what we want to send over the **Splunk server**. To do that, we must configure a file called `inputs.conf`. Go to C: => Program files => Splunk universal forwarder =>  etc => system => default and copy the `inputs.conf` file. Go back to the **local** directory and paste the file (The reason is not to edit the default file in the default directory).

There, I will edit the `inputs.conf` file with this content:

```
[WinEventLog://Application]

index = endpoint

disabled = false

[WinEventLog://Security]

index = endpoint

disabled = false

[WinEventLog://System]

index = endpoint

disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]

index = endpoint

disabled = false

renderXml = true

source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

This file is instructing Splunk Forwarder to push the vents related to **Application**, **Security**, **System** and **Sysmon** over to the Splunk server. The `index = endpoint` means that all the events sent to the server will be placed under the index **endpoint**. If the Splunk server does not have an index named endpoint won't receive any of this events.

An important thing, every time we edit the `inputs.conf` file, as now, we must restart the Splunk Universal Forwarder service. To do so, press the Windows key and search for Services and run it as Administrator. In the Services list search for SplunkForwarder, right click it and select Properties. 
In the Properties windows go to the **Log On As** tab and you might see the selected account as **NT SERVICE\SplunkForwarder**. Leaving it like that would mean that the logs won't be collected, because of the permissions, so instead, slect the option **Local System account**:

![Log on as Local System account](./assets/log-on-as-local-system-account.png)

Then, restar the SplunkForwarder service.

Now, head to the browser to the Splunk web portal and login using the credentials created during the Splunk install and you should be presented with the Splunk Administrator screen. 
Recalling the `inputs.conf` file, all the events are being sent over an index call `endpoint`, here we will create that index. Go to the top bar and click on Settings => Indexes. We will be presented with all the indexes that Splunk has, click on **New Index** and create a new Index with the name `endpoint`. 

Now, we need to make sure that the Splunk server receives the data. For that, click on Settings => Forwarding and receiving => Configure receiving => New receiving port => Type 9997 (the same port we introduced during the set up).

If everything was done right, we should start seeing incoming data from the target machine. In the top bar click on Apps => Search & Reporting => In the Search abr type **index="endpoint"** and search. Many events will be returned, but focus on the left column in **host**, clickin on it we will see that correponds to **target-pc**, the name we gave to the machine previously.

![Splunk host: target-pc](./assets/splunk-host-target-pc.png)

Besides, checking in **source** we should see the values **Application**, **Security**, **System** and **Sysmon**, setted up previously as well.

![Splunk sources](./assets/splunk-sources.png)

