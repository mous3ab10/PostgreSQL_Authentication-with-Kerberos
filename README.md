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

![Capture d’écran 2023-04-03 170157](https://user-images.githubusercontent.com/116025610/236161755-675113e2-c6c0-4e56-a698-ca80d4f51855.png)
- the administrative server : 'kdc.insat.tn
![Capture d’écran 2023-04-03 170251](https://user-images.githubusercontent.com/116025610/236165002-19e889c3-b5fb-4124-b4fd-ffcb939ba4c3.png)
Realm is a logical network, similar to a domain, that all the users and servers sharing the same Kerberos database belong to.

The master key for this KDC database needs to be set once the installation is complete :
```
sudo krb5_newrealm
```
![Capture d’écran 2023-04-03 170508](https://user-images.githubusercontent.com/116025610/236166159-934ca304-2e91-4095-a364-2c6e332a0055.png)

The users and services in a realm are defined as a principal in Kerberos. These principals are managed by an admin user that we need to create manually :
```
    $ sudo kadmin.local
    kadmin.local:  add_principal root/admin
    ```
    ![Capture d’écran 2023-04-03 170821](https://user-images.githubusercontent.com/116025610/236166623-5462126d-4862-43ce-b46b-1ce30f3475ed.png)
    kadmin.local is a KDC database administration program. We used this tool to create a new principal in the INSAT.TN realm (add_principal).

We can check if the user root/admin was successfully created by running the command : kadmin.local: list_principals. We should see the 'root/admin@INSAT.TN' principal listed along with other default principals.
![Capture d’écran 2023-04-03 170911](https://user-images.githubusercontent.com/116025610/236166944-af23de5b-1119-4d56-abb8-536d2b5a0c75.png)

Next, we need to grant all access rights to the Kerberos database to admin principal root/admin using the configuration file /etc/krb5kdc/kadm5.acl .
sudo vi /etc/krb5kdc/kadm5.acl

In this file, we need to add the following line :
```
*/admin@UCT.TN    *
```
![Capture d'écran 2023-04-03 172914](https://user-images.githubusercontent.com/116025610/236167617-3c9254c1-70ff-46da-8739-88d85da654f2.png)

For changes to take effect, we need to restart the following service : sudo service krb5-admin-server restart

Once the admin user who manages principals is created, we need to create the principals. We will to create principals for both the client machine and the service server machine.

Create a principal for the client
```
    $ sudo kadmin.local
    kadmin.local:  add_principal moussab
    ```
    ![Capture d'écran 2023-04-03 170815](https://user-images.githubusercontent.com/116025610/236168364-e2c68120-a11a-432b-bd1a-7d662207a97c.png)
    Create a principal for the service server
```
         kadmin.local:  add_principal postgres/pg.uc.tn
 ```
We can check the list of principals by running the command : ```kadmin.local: list_principals```

![Capture d'écran 2023-04-03 194319](https://user-images.githubusercontent.com/116025610/236168884-a936c294-ec74-4226-909e-84eb32d6bb9d.png)

### Service server Machine Configuration
For an easier configuration of Postgres I changed the Operating System login name from 'yosra' to 'postgres'.

Configuration of Kerberos

Installation of Packages

Following are the packages that need to be installed on the Service server machine :
```
$ sudo apt-get update
$ sudo apt-get install krb5-user libpam-krb5 libpam-ccreds
```
During the installation, we will be asked for the configuration of :

- the realm : 'UC.TN' (must be all uppercase)
- the Kerberos server : 'kdc.uc.tn'
- the administrative server : 'kdc.uc.tn'
PS : We need to enter the same information used for KDC Server.

Preparation of the keytab file
We need to extract the service principal from KDC principal database to a keytab file.

In the KDC machine run the following command to generate the keytab file in the current folder :
  ````
   $ ktutil 
   ktutil:  add_entry -password -p postgres/pg.insat.tn@INSAT.TN -k 1 -e aes256-cts-hmac-sha1-96
   Password for postgres/pg.insat.tn@INSAT.TN: 
   ktutil:  wkt postgres.keytab
  ``` 
  ![Capture d'écran 2023-04-04 013437](https://user-images.githubusercontent.com/116025610/236170814-e5837ea1-0281-48eb-b2ce-33c886eedd0e.png)
  Send the keytab file from the KDC machine to the Service server machine :
In the Postgres server machine make the following directories :

mkdir -p /home/postgres/pgsql/data

In the KDC machine send the keytab file to the Postgres server :

scp postgres.keytab postgres@<PG_SERVER_IP_ADDRESS>:/home/postgres/pgsql/data

! We need to have openssh-server package installed on the service server : sudo apt-get install openssh-server.
![Capture d'écran 2023-04-04 175121](https://user-images.githubusercontent.com/116025610/236171113-f5d6dd8f-12aa-4e57-839d-29051e00f60f.png)

Verify that the service principal was succesfully extracted from the KDC database :

- List the current keylist

      ktutil:  list

- Read a krb5 keytab into the current keylist

      ktutil:  read_kt pgsql/data/postgres.keytab

- List the current keylist again

       ktutil:  list
  ![Capture d'écran 2023-04-04 175621](https://user-images.githubusercontent.com/116025610/236171492-e9f1fa74-3b70-4bbc-9527-7cdb798497df.png)
       #### Configuration of the service (PostgreSQL)
##### Installation of PostgreSQL
Update the package lists

sudo apt-get update

Install necessary packages for Postgres

sudo apt-get install postgresql postgresql-contrib

Ensure that the service is started

sudo systemctl start postgresql

Create a Postgres Role for the Client
We will need to :

create a new role for the client

create user yosra with encrypted password 'some_password';

create a new database

create database moussab;

grant all privileges on this database to the new role

grant all privileges on database moussab to moussab;
![Capture d'écran 2023-04-04 181034](https://user-images.githubusercontent.com/116025610/236172362-30286857-0b53-427c-a818-310814474f5c.png)
To ensure the role was successfully created run the following command :

```postgres=# SELECT usename FROM pg_user WHERE usename LIKE 'moussab';```

![Capture d'écran 2023-04-04 181303](https://user-images.githubusercontent.com/116025610/236172721-2704c49b-cc8a-4bb3-bd3a-2deeaa4e1ca4.png)
The client yosra has now a role in Postgres and can access its database 'yosra'.

Update Postgres Configuration files (postgresql.conf and pg_hba.conf )
Updating postgresql.conf
To edit the file run the following command :

````sudo vi /etc/postgresql/12/main/postgresql.conf```

By default, Postgres Server only allows connections from localhost. Since the client will connect to the Postgres server remotely, we will need to modify postgresql.conf so that Postgres Server allows connection from the network :

```listen_addresses = '*'```
We will also need to specify the keytab file location :

```krb_server_keyfile = '/home/postgres/pgsql/data/postgres.keytab'````

![Capture d'écran 2023-04-05 063723](https://user-images.githubusercontent.com/116025610/236173186-6b7846db-8099-432b-aef0-e33c1467d16b.png)

