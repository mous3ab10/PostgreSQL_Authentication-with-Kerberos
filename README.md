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
![Capture d’écran 2023-04-03 102221](https://user-images.githubusercontent.com/116025610/235907432-fb0d2476-ade4-4ce1-b115-bbec98894095.png)


In my case :
- Client machine(physical machine) ip address is 192.168.56.1
- Service server machine (one the virtual machines) ip address is 192.168.56.108
- KDC machine (the other virtual machine) ip address is 192.168.56.102
Now that we added ip addresses to the virtual machines, we will start by setting hostnames for each machine :
- KDC machine

hostnamectl --static set-hostname kdc.uc.tn
![Capture d'écran 2023-04-03 094850](https://user-images.githubusercontent.com/116025610/235900241-cef63b22-2211-4fb8-ab5f-8a08e8c5a603.png)
- Service Server machine
hostnamectl --static set-hostname pg.uc.tn
 ![Capture d'écran 2023-04-03 033334](https://user-images.githubusercontent.com/116025610/235900039-5a48181c-be94-4acb-b637-8dc8810b930c.png)
- Client machine
hostnamectl --static set-hostname client.uc.tn
![Capture d’écran 2023-04-03 101748](https://user-images.githubusercontent.com/116025610/235900048-62d6e115-b2aa-401a-8494-2280b935c034.png)

*And open a new terminal for changes to take effect.*

*We can check the hostname of a machine by running the command : ```hostname```**

Next, we will be mapping these hostnames to their corresponding IP addresses on all three machines using /etc/hosts file.
sudo vi /etc/hosts

Now, we should set below information to /etc/hosts **for all three machines** :
```
<KDC_IP_ADDRESS>    kdc..tn  uc     kdc
<PG_SERVER_ADDRESS>    pg.uc.tn        pg
<CLIENT_ADDRESS>    client.uc.tn    client
```
![Capture d’écran 2023-04-03 165242](https://user-images.githubusercontent.com/116025610/235906935-742e31b2-25af-4108-af49-3ec7e387eb11.png)

Once the setup is done, we can check if everything is working fine by using the nslookup command to query the DNS to obtain the mapping we just did and the ping command to ensure that all three machines are reachable.

This an example in the client machine :

![Capture d’écran 2023-04-03 165242](https://user-images.githubusercontent.com/116025610/236160692-dd2c040c-6e34-4c13-b5f1-578f20fb76a5.png)

### Key Distribution Center Machine Configuration
Following are the packages that need to installed on the KDC machine :
```
$ sudo apt-get update
   $ sudo apt-get install krb5-kdc krb5-admin-server krb5-config
```
During the installation, we will be asked for configuration of :
- the realm : 'UC.TN' (must be all uppercase)

![Capture d’écran 2023-04-03 170107](https://user-images.githubusercontent.com/116025610/236161324-608980d0-84d2-4531-a901-a3284da497f2.png)
- the Kerberos server : 'kdc.uc.tn'
![Capture d’écran 2023-04-03 170107](https://user-images.githubusercontent.com/116025610/236161819-fa83edea-b9e8-45aa-81f4-96c4c6c59398.png)
![Capture d’écran 2023-04-03 170157](https://user-images.githubusercontent.com/116025610/236161755-675113e2-c6c0-4e56-a698-ca80d4f51855.png)
