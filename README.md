#  Securing Local and Remote Access with Console Authentication, SSH, and ACLs

In this project I will be using Cisco Packet Tracer to show how to implement console port security and safe remote access for Switches/Routers I pre-configured in order mimick a small enterprise network. Documentation for all configurations including Vlans, static routing, etc are included in the folders above. 

If you want a more realistic environment without real physical equipment and less limitations then you can use GNS3 or Eve-NG with VMware Workstation Pro, however due to the simplicity of this project, and those other applications requiring more resources/setup while being prone to crashes I choose Cisco Packet Tracer.

<img width="1015" height="556" alt="README1" src="https://github.com/user-attachments/assets/b68ce077-b49d-433b-bd6a-33947d529cad" />

## The Mission

- Secure device management by requiring authentication on all console ports of the network
- Enable encrypted SSH access instead of Telnet(insecure/unencrypted)
- Restrict remote access so only a single trusted host (Administrator - 172.16.1.1) can connect using an ACL.


# Step 1: Identify Serial line 
- Connect the Console Cable (RJ-45 to USB in my case) on the port labeled "console" on the Router/Switch(RJ-45 side), and the USB side on the PC
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

### #PLEASE READ# 

As you can see at the end of my demonstration the Console Port Security has already been set with user authentication. In this project I will show how to enable it for physical and remote access(SSH)

Had I not configured it, anyone would be able plug into the device's CLI User EXEC mode and instantly get information like:

- IP addresses

- Interface status

- Device type

### #IMPORTANT TIP#
You should ALWAYS add extra security towards privileged EXEC mode access
using the command "enable secret StrongPasswordHere" in configuration terminal. I will demonstrate it on the last step of this project.

Best practices is having seperate passwords for both. User EXEC mode allows read-only access, and Privileged EXEC mode allows for device configurations.

Additionally, many enterprises also use External AAA servers such as RADIUS / TACACS+ to have better control over the level of privilege users have and what they can do inside the devices.
  </td>
  </tr>
</table>

# Step 2: Console Port Security (Local Physical Access) 
Goals: 
- Prevent unauthorized physical access
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
      As you can see above User EXEC Mode has now been configured with username/password authentication and unathorized users cant login anymore.
    </td>
    <td>
      <b>COMMAND SUMMARY</b><br><br>
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

# Step 3: SSH Remote Access (Secure Remote Management)
Goals: 
- Enabled SSH v2
- Disabled Telnet
- Configured RSA keys
- Secured VTY lines with local authentication

https://github.com/user-attachments/assets/0ed4f9c0-73a9-4acc-8547-9778e5f33cd4


  CLI:
  <img src="https://github.com/user-attachments/assets/230d8ccf-6509-4ebd-b83f-308781384a3e"  width="592" height="451" />

 <b>COMMAND SUMMARY</b>
 
(config)#hostname ROUTER-DISTRIBUTION-2 <br>
Not shown in video/photo because I had already preconfigured it. <br>
-Gives device a name. Required for generating SSH keys.<br>

(config)#ip domain-name R2.domain.local <br>
-Allows for DNS Resolution, as well as security certificates like SSH, IPsec, HTTPS.

(config)#crypto key generate rsa <br>
-Generates RSA keys used for SSH encryption. 2048 = key size (minimum standard today)

(config)#line vty 0 15 <br>
-Target virtual terminal lines (remote access sessions). 0 15 = Allows 15 simultaneous SSH sessions. Not the same as line console 0 (physical access).

(config-line)#transport input ssh ssh<br>
-Disables Telnet (insecure/unencrypted). Allows only encrypted SSH connections.

(config-line)#login local <br>
-Use local usernames/passwords. Without this, login won’t work even if users exist.

(config)#ip ssh version 2 <br>
-Forces SSH version 2. More secure than version 1 (outdated).

##  REPEATED STEP 3 IN DISTRIBUTION-ROUTER-1 AND ACCESS SWITCHES EXCEPT DIFFERENT DOMAIN NAMES
DISTRIBUTION-ROUTER-1 = R1.domain.local<br>
DISTRIBUTION-ROUTER-2 = R2.domain.local<br>
SWITCH-ACCESS-1 = SW1.domain.local<br>
SWITCH-ACCESS-2 = SW2.domain.local<br>

# Step 3.5: Loopback Addresses and Switch Virtual Interfaces
Goal: <br>
Create loopbacks and SVIs to have reliable SSH access. Loopbacks never go down, additionally they are also used for routing(OSPF/BGP), and switches dont route by default so it needs a layer 3 interface.



