# Ubee EVW3226 modem/router multiple vulnerabilities #

## Platforms / Firmware confirmed affected: ##
- Ubee EVW3226, 1.0.20
- Product page: http://www.ubeeinteractive.com/products/cable/evw3226

## Vulnerabilities ##
### Insecure session management ###
The web interface does not use cookies at all. If admin login is successful, the IP address of the admin user is stored and everybody can access the management interface with the same IP.

### Local file inclusion ###
Setup.cgi can read any file with .htm extension using directory traversal in the gonext parameter. Although the file must have htm extension, the local file inclusion can be used to map directories, because the response is different depending on whether directory exists or not.
POC: 
`http://<device_ip>/cgi-bin/setup.cgi?gonext=../www/main2`

### Backup file is not encrypted ###
Although the web interface requires a password for encrypting the backup file, the encryption is not performed. In order to backup file password, the plain password is stored in the backup file, which is a standard tgz (gzipped tar) file with a simple header.

### Backup file disclosure ###
When a user requests a backup file, the file is copied into www root in order to make download possible. However, the backup file is not removed from the www root after download. Since there is not any session check required to download the backup file, an attacker is able to download it without authentication from LAN until the next reboot.
Since the backup file is not encrypted and contains the plain admin password, the router can be compromised from LAN.
POC:
`http://<device_ip>/Configuration_file.cfg`

### Authentication bypass (backdoor) ###
The web interface bypasses authentication if the HTML request contains the factoryBypass parameter. In this case a valid session is created and the attacker can gain full control over the device.
POC:
`http://<device_ip>/cgi-bin/setup.cgi?factoryBypass=1`

### Arbitrary code execution ###
The configuration file restore function receives a compressed tar file, which is extracted to the /tmp folder. Tar files may contain symbolic links, which can link out from the extraction folder. By creating a configuration file with a symbolic link and a folder which uses this link, the attacker can write out from the backup folder and can overwrite any file in the writable file-system.
Since www is copied to the writable file system at boot time (under /tmp), the attacker can insert a new cgi script that executes arbitrary code with root privileges.

### Default SSID and passphrase can be calculated ###
The default SSID and passphrase are derived only from the MAC address. Since the MAC address of the device is broadcasted via WiFi, the default password can be calculated easily.
Combined with code execution and factory bypass, even a botnet of Ubee routers can be deployed easily.

### Buffer overflow in configuration restore ###
During the configuration restore process, the backup file password is read from the pass.txt file. If the password is large enough (larger than 65536), a stack based buffer overflow is caused, because the file content is loaded with fscanf(“%s”) to a stack based local variable. The stack based buffer overflow can be used to execute arbitrary code with root privileges.

### Buffer overflow in configuration file request ###
The web interface identifies the configuration file download request by checking that the URL contains the Configuration_file.cfg string. If this string is found, the whole URL is copied into a stack based buffer, which can cause a buffer overflow. This stack based buffer overflow can be used to execute arbitrary code with root privileges without authentication.
POC:
`http://192.168.0.1/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaConfiguration_file.cfg`

### Buffer overflow in next file name ###
The gonext variable in the POST requests specifies the HTML file, which the cgi script should be loaded. If the gonext variable is large enough (larger than 6512 bytes), a stack based buffer overflow is caused, which can be used to execute arbitrary code with root privileges without authentication.

### Communication on the UPC Wi-Free can be sniffed within the device ###
The UPC Wi-Free communication is not separated correctly inside the device, because the whole communication can be sniffed after gaining root access to the device.

## Timeline ##
- 2015.06.24: Presenting the Ubee router problems to the CTO of UPC Magyarorszag
- 2015.07.16: UPC contacted Ubee and required some more proof about some specific problems
- 2015.07.16: Proofs, that the default passphrase calculation of the Ubee router was broken, were sent to UPC
- 2015.07.20: UPC requested the POC code
- 2015.07.21: POC code was sent to UPC
- 2015.07.30: We sent some new issues affecting the Ubee router and other findings in Technicolor TC7200 and Cisco EPC3925 devices to UPC
- Between 2015.07.31 and 08.12 there were several e-mail and phone communications between technical persons from Liberty Global to clarify the findings
- 2015.08.19: UPC sent out advisory emails to its end users to change the default WiFi passphrase
- 2015.09.16: Ubee Interactive also asked some questions about the vulnerabilities
- 2015.09.24: We sent detailed answers to Ubee Interactive
- 2016.01.27: UPC Magyarorszag send out a repeated warning to its end users about the importance of the change of the default passphrases.
- 2016.02.16: Face to face meeting with Liberty Global security personnel in Amsterdam headquarters 
- 2016.02.18: A proposal was sent to Liberty Global suggesting a wardriving experiment in Budapest, Hungary to measure the rate of end users who are still using the default passphrases.

## POC ##
POC script is available to demonstrate the following problems:
- Authentication bypass
- Unauthenticated backup file access
- Backup file password disclosure
- Code execution

[Video demonstration is also available](https://youtu.be/cBclw7uUuO4), which presents the above problems and how these can be combined to obtain full access to the modem.

## Recommendations ##
Since only the ISP can update the firmware, we can recommend for users to change the WiFi passphrase.

## Links ##
[Search-lab advisory](http://www.search-lab.hu/advisories/secadv-20150720)

[Video demonstration](https://youtu.be/cBclw7uUuO4)

