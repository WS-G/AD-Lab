# AD-Lab

My notes on setting up an active directory lab in Virtualbox for practising pentesting on active directory

## Setup

Download the necessary isos from Microsoft eval center:
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
    - 