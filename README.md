#  Securing Local and Remote Access with Console Authentication, SSH, and ACL

##Recommended to view on a desktop/laptop and not Mobile due to app bugs not showing video clips on certain steps

In this project I use Cisco Packet Tracer to show how to implement Console port security (Local Physical Access) and secure remote access (SSH + ACL) on switches and routers, simulating enterprise networks. The goal is to prevent unauthorized access while ensuring secure device management. Full Configurations of the devices are included in the folder above. 

Keep in mind that some steps from this page are GIFs and not images that you can watch by clicking on the play button. GIFs, Videos, and Screenshots are included for everything.

NOTE: A more realistic environment without real physical equipment would be GNS3 or Eve-NG with VMware Workstation Pro which uses real Cisco IOS images, however due to the simplicity of this project, and those other applications requiring more resources/setup while being prone to crashes I choose Packet Tracer.

<img width="1015" height="556" alt="README1" src="https://github.com/user-attachments/assets/b68ce077-b49d-433b-bd6a-33947d529cad" />

## The Mission

- Secure device management by requiring authentication on all console ports of the network
- Enable encrypted SSH access instead of Telnet(insecure/unencrypted)
- Restrict remote access so only a single trusted host (Administrator - 172.16.1.1) can connect using an ACL.


# Step 1: Identify Serial line 
- Connect the Rollover/Console Cable (RJ-45 to USB in my case) on the port labeled "console" on the Router/Switch(RJ-45 side), and the USB side on the PC
- Open the application "Device Manager" on your local computer, Click on the PORTS (COM &LPT) section, And look for "USB Serial Port (COM?)" example: COM5, COM3.
- Open a terminal emulator (PuTTY, Tera Term,etc..), Choose Serial NOT SSH OR TELNET, Set the COM port number to the one found in Device Manager (example: COM5, COM3)
  
<img width="959" height="690" alt="Device Manager-Putty" src="https://github.com/user-attachments/assets/cf5a7e72-0764-41d5-b71c-ad979ff275a4" type="video/mp4"> </video>


# Step 1 Video Example:

<table>
  <tr>
    <td width="00%">

<video src="https://github.com/user-attachments/assets/c1d2c8dc-38c1-4955-9658-7df5770b0615" controls width="100%"></video>

  </td>
    <td width="00%">

As you can see at the end of my demonstration the Console Port Security has already been set with user authentication. In this project I will show how to enable it for physical and remote access(SSH)

Had I not configured it, anyone would be able plug into the device's CLI User EXEC mode and instantly get information like:

- IP addresses

- Interface status

- Device type

  </td>
  </tr>
</table>

# Step 2: Console Port Security (Local Physical Access) 
Goals: 
- Prevent unauthorized physical access to Switches and Routers.
- Configured password / local authentication
- Idle timeout enforced
<img width="1271" height="527" alt="Step2" src="https://github.com/user-attachments/assets/95a8c2c6-6bbe-4086-85cb-20dbb27c566d" />

## INSIDE THE CLI OF SWITCH-ACCESS-1:
<table>
  <tr>
    <td>
      Enter these commands:
      <img src="https://github.com/user-attachments/assets/49f921df-2d21-4e9a-b6ff-18a84a1ef01a"  width="1271" height="527" /> <br><br>
      Result:<br><br>
      <img src="https://github.com/user-attachments/assets/47b82991-8b38-44c3-b12a-482c1026d9de" />
      <br><br>
      As you can see above User EXEC Mode has now been configured with username/password authentication and unathorized users cant login anymore without credentials.
    </td>
    <td>
      <b>COMMANDS EXPLANATION </b><br><br>
>enable <br>   
-Enter Priviledged EXEC Mode<br><br>

#configure terminal <br>
-Enter Global Configuration Mode

(Config)#service password-encryption <br>
-Encrypts all future plaintext passwords.

(config)#username kgonzalez secret StrongPasswordHere <br>
-Creates local user with encrypted password.

(config)#line console 0 <br>
-Enter console line 0. Most network devices only have one console port/interface labeled 0.

(config-line)#login local <br>
-Uses username/password for console login authentication.

(config-line)#exec-timeout 5 0 <br>
-kicks users who have been inactive for 5 minutes.

(config-line)#logging synchronous <br>
-Optional. Cleaner CLI, System logs and debug messages do not interrupt your typing.

(config-line)#do write <br>
-Save configuration.
  </td>
  </tr>
</table>

##  REPEATED STEP 2 IN SWITCH-ACCESS-2 AND DISTRIBUTION ROUTERS

# Step 2.5: Privileged EXEC mode Hardening

<img width="503" height="78" alt="enablesecret" src="https://github.com/user-attachments/assets/52316645-c230-4430-8150-c4f8d0d7a9a6" />

Used Command "enable secret" to provide encrypted password towards Privileged EXEC mode.

##  REPEATED STEP 2.5 IN SWITCH-ACCESS-2 AND DISTRIBUTION ROUTERS

### #IMPORTANT #
You should ALWAYS use this command for extra security towards privileged EXEC mode access. It is also neccesary for SSH access.

Best practices is having seperate passwords for both. User EXEC mode allows read-only access, and Privileged EXEC mode allows for device configurations.

Additionally, many enterprises also use External AAA servers such as RADIUS / TACACS+ to have better control over the level of privilege users have and what they can do inside the devices.

# Step 3: SSH Remote Access (Secure Remote Management)
Goals: 
- Enabled SSH v2
- Disabled Telnet
- Configured RSA keys
- Secured VTY lines with local authentication

https://github.com/user-attachments/assets/0ed4f9c0-73a9-4acc-8547-9778e5f33cd4


  CLI:
  <img src="https://github.com/user-attachments/assets/230d8ccf-6509-4ebd-b83f-308781384a3e"  width="592" height="451" />

 <b>COMMANDS EXPLANATION</b>
 
(config)#hostname ROUTER-DISTRIBUTION-2 <br>
Not shown in video/photo because I had already preconfigured it. <br>
-Gives device a name. Required for generating SSH keys.<br>

(config)#ip domain-name R2.domain.local <br>
-Allows for DNS Resolution, required for SSH keys and other security certificates like IPsec, and HTTPS.

(config)#crypto key generate rsa <br>
-Generates RSA keys used for SSH encryption. 2048 = key size (minimum standard today)

(config)#line vty 0 15 <br>
-Target virtual terminal lines (remote access sessions). 0 15 = Allows 15 simultaneous SSH sessions. Not the same as line console 0 (physical access).

(config-line)#transport input ssh ssh<br>
-Disables Telnet (insecure/unencrypted). Allows only encrypted SSH connections.

(config-line)#login local <br>
-SSH does not work with default login. LOGIN LOCAL (Local usernames/passwords) must be used. Without this command, login won’t work even if users exist.

(config)#ip ssh version 2 <br>
-Forces SSH version 2. More secure than version 1 (outdated).

##  REPEATED STEP 3 IN DISTRIBUTION-ROUTER-1 AND ACCESS SWITCHES EXCEPT DIFFERENT DOMAIN NAMES
DISTRIBUTION-ROUTER-1 = R1.domain.local<br>
DISTRIBUTION-ROUTER-2 = R2.domain.local<br>
SWITCH-ACCESS-1 = SW1.domain.local<br>
SWITCH-ACCESS-2 = SW2.domain.local<br>

# Step 3.5: Loopback Addresses and Switch Virtual Interfaces
Goal: <br>
Create loopbacks and SVIs to have reliable SSH access. Loopback interfaces never go down, and switches dont route by default so it needs a layer 3 interface.



<table>
  <tr>
    <td>
      ROUTER-DISTRIBUTION-1 / Loopback Address Configuration:
 <img width="1532" height="650" alt="r1loopback" src="https://github.com/user-attachments/assets/3b87cd33-694c-4576-93aa-1cbabde60900" /> <br><br>
      ROUTER-DISTRIBUTION-2 / Loopback Address Configuration:<br><br>
      <img width="1527" height="650" alt="r2loopback" src="https://github.com/user-attachments/assets/b87aad14-6bff-4191-bf9a-c3054d9fe252" />
      <br><br>
    </td>
    <td>
      NOTE: Loopback isnt reachable to other networks by default so either dynamic routing like ospf or static routing is needed. <br><br> I Used static #ip route 10.255.255.1 255.255.255.255 10.0.0.1 on R2 to allow reachability from Admin's subnet<br><br>
A loopback interface is always “up” as long as the device is on since its virtual. This allows me to have a reliable SSH ip address to target instead of rather from an physical interface IP that could get damaged in real life.<br>
<br><br>
Also perfect to have as a Routing ID for routing protocols (OSPF/BGP) to prevent instability/reconvergence issues<br>

  </td>
  </tr>
</table>
<br>

<table>
  <tr>
    <td>
      SWITCH-ACCESS-1 / SVI Configuration:
<img width="1080" height="200" alt="Switch1-SVI" src="https://github.com/user-attachments/assets/6976bf98-a286-4f45-9e23-012cf4b136cb" /> <br><br>
      SWITCH-ACCESS-2 / SVI Configuration:<br><br>
      <img width="1087" height="200" alt="Switch2-SVI" src="https://github.com/user-attachments/assets/2024abde-06cd-40d4-b710-95956198ecc1" />
      <br><br>
    </td>
    <td>
Created a management interfaces (SVI) on VLAN 1 <br> for both switches using an ip address within the same subnet as the default gateway(R1)<br>
<br>
 The ip default-gateway command lets the switch reach other networks. <br>
   <br>    
  Similar to loopbacks, the SVI ip address will be used to access the device remotely (SSH). <br>
    <br>
   ## IMPORTANT ## <br>
    Ideally its best not use vlan1(native) for security measures, and instead create a different one like vlan99 for management<br>
    <br>However I did not enable trunk links to allow inter-vlan routing in this small project (A physical interface must also be up for SVI to work) <br>


  </td>
  </tr>
</table>

# Step 4: Restrict SSH Access to One Host (ACL)

https://github.com/user-attachments/assets/5ec530bb-6449-4e3a-9e28-93a4cba65d56


## REPEATED STEP 4 IN IN SWITCH-ACCESS-2 AND DISTRIBUTION ROUTERS

<b>COMMAND EXPLANATION</b><br><br>
(config)# ip access-list standard MANAGEMENT = Creates an named ACL list called MANAGEMENT. Filters based only on source IP address.<br>
(config-std-nacl)#permit host 172.16.1.1 = Adds rule to ACL, allows only the exact ip 172.16.1.1<br>
(config-std-nacl)#line vty 0 15 = Enter virtual terminal lines configuration mode (SSH/Telnet)<br>
(config-line)#access-class MANAGEMENT in = Applies ACL to remote login access, filtering traffic coming into the device via vty<br>

# Final Step: Verification

The command to SSH from an end user device after configuring it is "ssh -l username ip address"<br>
In my case I used "ssh -l kgonzalez 10.255.255.1" to SSH into ROUTER-DISTRIBUTION-1's Loopback Address, an interface that always remains up.

# ## *SSH APPROVAL* ## #


Below you can see Administrator host 172.16.1.1 is allowed remote access.<br><br>
## Click GIF<br>
<img width="1226" height="618" alt="SSH-Verification-Admin" src="https://github.com/user-attachments/assets/47e15004-39a6-405d-acf6-5a84339ae479" />

# ## *SSH REFUSED* ## #


Here you can see that other hosts are not allowed into the console due to the access control list (ACL) entry, ensuring that unauthorized users cant get into the CLI even if they have the user credentials<br>
## Click GIF<br>
<img width="1102" height="618" alt="SSH-Refused" src="https://github.com/user-attachments/assets/ae56afdd-0fa3-4cbf-b26e-ba6586a951ef" />



