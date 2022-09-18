# NETWORK BOOTING RASPBERRY PI4 / PXE BOOTING 

At the beginning we have to use an SD card with Raspbian Operating System installed in our Raspberry Pi just for setup.

## SSH Connection and Configuration of Raspberry Pi

To configure our Raspberry Pi first we have to establish a connection and a network between our Server PC. For this project’s purpose a network connection through ethernet cable is sufficient. 

To share our network in Ubuntu first ethernet cable must be plugged in between Raspberry Pi and PC. In Settings/Network wired connection can be seen. In wired connection setting it is necessary to change IPv4 to ‘Shared to other computers.’

![Picture1](https://user-images.githubusercontent.com/71449553/190924931-61abe025-004e-4cac-9b71-84cd46ac2638.png)

After establishing a network with Raspberry Pi4 we have to find the Ip address of this network, this can be done by writing “ifconfig” to terminal. This command will result with showing us all network connections of our server PC. In the results we have to check for wired LAN connection and take the IP address assigned to it.

To establish a SSH connection we have to learn the IP address of our Raspberry Pi4. To find it we can scan our wired network with “sudo nmap -n ‘IP address of wired LAN connection’”.

In the results we can see our IP address of our Raspberry Pi. Then we can connect to our Raspberry Pi4 by “sudo ssh pi@ ‘Ip Adress of Raspberry Pi’” To connect security protocol will ask the password of Raspberry Pi then the connection will establish. Now our server PC is using the terminal of Raspberry Pi, by writing “sudo config” the configuration menu of Raspberry Pi opens. Then we have to select Network Boot Option in System Options / Boot Order menu.

![Picture2](https://user-images.githubusercontent.com/71449553/190924987-56e4808e-51cb-48d1-8e5c-ec5de46d5245.png)

![Picture3](https://user-images.githubusercontent.com/71449553/190924997-93ccda9a-8153-4f53-beef-25b0733d2de5.png)

![Picture4](https://user-images.githubusercontent.com/71449553/190925001-90869053-1cfe-4e58-9b2b-6b6460eede98.png)

## Creating local boot files and configuring

For our client devices to boot we have to create a bootable operating system in our network. For our project we are using our Ubuntu Pc’s local storage to create a bootable location. Since we are using Raspbian Operating System for Raspberry Pi’s it is needed to download the latest Raspbian version to our PC than transform it to a bootable location.

```
1.	sudo apt install unzip kpartx dnsmasq nfs-kernel-server
2.	unzip ‘downloaded Raspbian file’
3.	sudo kpartx -a -v ‘unzipped Raspbian file’ 
4.	mkdir root
5.	mkdir boot 
6.	sudo mount /dev/mapper/loop0p1 boot/
7.	sudo mount /dev/mapper/loop0p2 root/
```
The commands on top are essentially unzipping Raspbian operating system and creating two mounted images called root and boot. Now we want to copy the Raspbian folder contents to our newly created folders. After that we have to replace 2 files in boot folder in order to configurate the operating system.

```
1.	sudo mkdir -p/rpi/1
2.	sudo cp -a root/* /rpi/1/
3.	sudo cp -a boot/* /rpi/1/boot/
4.	cd /rpi/1/boot
5.	sudo rm start4.elf
6.	sudo rm fixup4.dat
7.	sudo wget https:github.com/Hexxeh/rpi-firmware/raw/master/start4.elf
8.	sudo wget https:github.com/Hexxeh/rpi-firmware/raw/master/fixup4.dat
```
Now we have to direct our Raspberry Pi to our boot partition, by default Raspberry looks for the folder with its serial name on it. After the Pi loaded, we need to show the path of root folder. We can achieve this by editing cmdline.txt file.

```
1.	echo "console=serial0,115200 console=tty root=/dev/nfs nfsroot=‘IP address of wired lan connection’ :/rpi/1,vers=3 rw ip=dhcp rootwait elevator=deadline" | sudo tee /rpi/1/boot/cmdline.txt
```
## Configuring the DNSMASQ

Now we have to set and configure our DNSMASQ and other services. At first, we have to edit dnsmasq config file. To do this we have to write the code below.

```
1.	echo "/rpi/1 *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
2.	echo 'dhcp-range=1,proxy' | sudo tee -a /etc/dnsmasq.conf
3.	echo 'log-dhcp' | sudo tee -a /etc/dnsmasq.conf
4.	echo 'enable-tftp' | sudo tee -a /etc/dnsmasq.conf
5.	echo 'tftp-root=/tftpboot' | sudo tee -a /etc/dnsmasq.conf
6.	echo 'pxe-service=0,"Raspberry Pi Boot"' | sudo tee -a /etc/dnsmasq.conf 
```
Now there’s an important part in some systems that when we open DNSMASQ it doesn’t work because of systemd-resolved, so we have to close it. Then we can start other necessary services to establish connection.

```
1.	sudo systemctl stop systemd-resolved
2.	sudo systemctl disable systemd-resolved
3.	sudo systemctl enable dnsmasq
4.	sudo systemctl restart dnsmasq
5.	sudo systemctl enable rpcbind
6.	sudo systemctl enable nfs-kernel-server
7.	sudo systemctl restart rpcbind
8.	sudo systemctl restart nfs-kernel-server
```

![Picture5](https://user-images.githubusercontent.com/71449553/190925234-834d8521-adb5-45f2-bf3b-d4e763dfaf01.png)

After finishing all we have to do scan our network and there we can see our Raspberry Pi4. After that connection to Raspberry Pi via SSH is possible.

### References
https://networkboot.org/fundamentals/

https://hackaday.com/2019/11/11/network-booting-the-pi-4/

https://williamlam.com/2020/07/two-methods-to-network-boot-raspberry-pi-4.html

https://codestrian.com/index.php/2020/02/14/setting-up-a-pi-cluster-with-netboot/

https://www.raspberrypi.com/documentation/
