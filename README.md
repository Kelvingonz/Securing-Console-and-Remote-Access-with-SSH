# Securing Console Port and Remote Access with SSH and ACL

In this project I will be using Cisco Packet Tracer to show how to implement terminal line security on network devices using text and video demonstrations. If you want a more realistic environment without real physical equipment and less configuration limitations then you can use GNS3 or Eve-NG with VMware Workstation Pro, however due to the simplicity of this project, and those other applications requiring more resources/setup while being prone to crashes I choose Cisco Packet Tracer.

<img width="1015" height="556" alt="README1" src="https://github.com/user-attachments/assets/b68ce077-b49d-433b-bd6a-33947d529cad" />

## Tasks

- Kelvin(Administrator) finds out that anyone can access the terminal lines(CLI) of the switches/routers in the network above
- Physically go to each device location with laptop to configure Console Port Security (Local Physical Access) and secure remote access (SSH)
- Only Administator host 172.16.1.1 will be permitted remote access on the network devices via Access Control List (ACL)


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
Goal: Prevent unauthorized people from plugging into the device and getting CLI access.
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

##  REPEAT STEP 2 IN SWITCH ACCESS 2 AND DISTRIBUTION ROUTERS

# Step 3:   SSH Remote Access (Secure Remote Management)
Goal: Allow secure remote login using Secure Shell protocol (instead of Telnet)

