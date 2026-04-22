# Securing-Console-Port-and-Remote-Access-with-SSH

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
Not shown in this project but you should ALWAYS add extra security towards privileged (admin-level/EXEC) access
using the command "enable secret YourPassword" in configuration terminal

Best practices is having seperate passwords for both. User EXEC mode allows read-only access, and Privileged EXEC mode allows for full device configurations.

Additionally, many enterprises also use External AAA servers such as RADIUS / TACACS+ to have better control over the level of privilege users have and what they can do inside the devices.

  </td>
  </tr>
</table>




