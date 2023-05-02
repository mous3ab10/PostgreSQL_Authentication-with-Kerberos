# PostgreSQL_Authentication-with-Kerberos
### Steps To Setup Kerberos On UBUNTU :
Kerberos is a network authentication protocol used to verify the identity of two or more trusted hosts across an untrusted network. It uses secret-key cryptography and a trusted third party (Kerberos Key Distribution Center) for authenticating client-server applications. Key Distribution Cente (KDC) gives clients tickets representing their network credentials. The Kerberos ticket is presented to the servers after the connection has been established.
![R](https://user-images.githubusercontent.com/116025610/235526423-ec1f9157-03c9-4c83-8c62-fabbc5db5e7c.jpeg)
Let's start by creating a new virtual machine. Under File go to Host Network Manager ... and then click Create.

### Hostname and IP Addresses
We will need three machines. In my case I'm using three ubuntu machines : my physical machine and two virtual machines inside of VirtualBox. My physical machine will be the client and the two other machines will be the Service Server and the KDC.

Virtual machines have a NAT adapter by default but in order to assign IP addresses to these machines we will need to add a host-only adapter manually.

![add_virtual_network](https://user-images.githubusercontent.com/116025610/235707546-4081e5da-2845-457e-aeac-cf904ba6d7b0.png)
And don't forget to enable the DHCP Server.

Now it's time to connect these virtual machines to our new virtual network. Go to the Settings of each of the virtual machines and under Network enable a second adapter 
![connect_machine_to_virtual_network](https://user-images.githubusercontent.com/116025610/235708931-168f779a-c8dc-49f6-aa19-4af1c8c7cdec.png)
Specify the type of the adapter (host-only adapater) and the virtual network to connect to (the one we just created).

We can check the IP addresses of all three machines by running **hostname -I** in each one.

In my case :
- Client machine(physical machine) ip address is 192.168.56.1
- Service server machine (one the virtual machines) ip address is 192.168.56.108
- KDC machine (the other virtual machine) ip address is 192.168.56.102
Now that we added ip addresses to the virtual machines, we will start by setting hostnames for each machine :
- KDC machine

''''hostnamectl --static set-hostname kdc.insat.tn''''
 
