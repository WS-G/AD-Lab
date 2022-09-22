# AD-Lab

My notes on setting up an active directory lab in VirtualBox for practising pentesting on active directory. This is related to Heath Adam's course: [Practical Ethical Hacking - TCM Academy](https://academy.tcm-sec.com/p/practical-ethical-hacking-the-complete-course)

## Setup

Download the necessary isos from Microsoft eval centre:
    - windows 10 enterprise x 2
    - Windows Server 2019

### Domain Controller Setup

1. Install Windows server 2019 on VirtualBox

2. Create admin password (keep safe)

3. install Guest additions
    - install the guest additions CD image and run from `My Devices` on the box (as administrator)
    -Restart the machine

4. Rename the server to something recogniseable (e.g DC1)

5. install AD
    - `Server Manager > Manage > Add roles or features`
    - Role-Based option
    - Choose server to install on
    - select "add active directory domain services"
    - install

6. Promote Server to a Domain controller
    - select flag in upper corner to promote
    - select option `add a forest`
    - name your domain. example: `example.local`
    - select DSRM password
    - continue to hit `Next` until finished
    - Restart machine

7. Set up LDAPS 
    - Go to server manager
    - Select "Add roles and Features"
    - Add the "Active Directory Certificate Services"
    - select the "Restart if needed" option 
    
### Setting up the Workstations

1. Install the Windows 10 Enterprise ISO into Virtualbox & name them appropriately (e.g WS1 and WS2 etc.)
    - select "join a domain instead" option for setting up user.
    - select name and use a simple password for practising cracking (use one maybe on rockyou list etc.) on at least one of the user accounts.
    - wait for install to finish and install guest additions (same method as DC)
    - Rename PC to something memorable (e.g DOMAIN-WS1)
    - Add a "share" folder to root of `C:\` and make it a share:
        - right-click folder > `properties > Shares`
    - Take a snapshot
    - Clone the machine and rename pc 

### Setting up users, groups and policies

1. Organise domain groups
    - Go to server manager on DC
    - got to Tools > `AD  users and computers`
    - Move all entries in `Users` except for administrators and Guest into a newly created OU called "Groups"

2. Add users for domain
    - Select users OU and add the user you set up in workstation
    - Add other low priv users & a named domain admin account
    - Add A SQL service account and make domain Admin (to replicate a common mistake seen in AD networks)
    - example can be found in [creds.json](creds.json)

3. Set up a Network share on DC
    - Go to: `Manage > add features` and add the file sharing features for windows server under `File and ISCI services`
    - Go To: `Server Manager > File and storage services > shares` and add a new share with default settings and name it whatver you like

4. Set up Kerberos (For Kerberoasting)
    - Open powershell/cmd as admin
    - set the spn for sql account:

        ```powershell
        setspn -a XYZ-DC1/sqlservice.XYZ.local:60111 XYZ/sqlservice   
        ```

    - check it has been set for your account:
        - ```setspn -T XYZ -Q */*```
        - The account should be at the end of the output:

            ```powershell
            CN=SQL Service,CN=Users,DC=XYZ,DC=local
            XYZ-DC1/sqlservice.XYZ.local:60111
            ```

5. Disable AV
    - Go to GPO management and run as admin
    - go to your forest drop-down and then domains and find your domain name you set up
    - right-click and `add a GPO objext and link it here`
    - Find the created GPO and Right-click and select enforced to on & then select `Edit`
    - go to `computer configuration > policies > administrative templates > windows components` and click on  windows defender antivirus and then select "Turn off windows defender" and make that "enabled"
    - Do the same for `Windows Defender Exploit guard` and `Smart Screen` if applicable

### Joining machines to domain

1. on the machines, add a network share in `C:\share`

2. Add the DC IP address as DNS server in workstations for name resolution
    - Go to network adapter settings and change the DNS server to the DC's IP
    - Repeat for other workstation(s)

3. Join the Domain
    - Search for "Access work or school" in windows search and select it and click "Connect"
    - Join the domain you set up (e.g yourdomain.local)
    - Enter the domain admin's credentials
    - **Note:** If using virtualbox make sure all the relevant machines are connected to a NATnetwork together. Otherwise you will not be able to access them.
    - After restart, check the domain account can be signed in to. select the "other user" option at the sign-in ascreen and sign in using a domain account to verify it's working

4. Add a domain user to local admin on both machines
    - This is for testing an AD attack vector later
    - choose a standard domain user account to use
    - Login to workstations as domain admin
    - Run powershell (as admin) and add the user to local admin group like this: `net localgroup administrators USERNAMEHERE /add`

5. add attacker machine to network 
	- to do this in VirtualBox we must go to the kali machine settings > network add network adapter and select the network our domain is on 
	- Now we must add the DC as as a dns server 
	- run `sudo vim /etc/resolv.conf` and add the following line: `nameserver <DC mac hine's IP>`
	- save and quit
	- then run  `sudo chattr +i /etc/resolv.conf  ` to update 
	- restart network manager: `sudo /etc/init.d/networking restart`
	- now you should ping the relevant ip's to see if you get a call-back
	- **NOTE:** if one or more of the machines is not pinging you may need to edit the firewall rules
		- advanced firewall settings > inbound rules >   File and Printer Sharing (Echo Request - ICMPv4-In) and enable all rules with this name 

