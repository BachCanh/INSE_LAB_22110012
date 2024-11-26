![image](https://github.com/user-attachments/assets/7a8432ab-80e4-4339-bc23-8449781436c3)# Lab #1, 22110012, Bach Duc Canh, INSE331280E_01FIE
# Task 1: Transfer files between computers
This lab explores various encryption algorithm with openssl
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**: I use Ping-pong of Security_lab 


## On Alice Side
## 1. Create a text file named `sample.txt`:

```sh
echo "This is a sample file" > sample.txt
```

## 2. Generate a checksum and Generate an RSA key pair 

Use SHA-256 to create a hash of the file.

```sh
openssl dgst -sha256 sample.txt > sample.txt.sha256
```
![image](https://github.com/user-attachments/assets/08a7ed8f-d411-4837-99c4-857202dfb427)


Generate a private key (sender.key) and a public key (sender.pub).

```sh
openssl genrsa -out sender.key 2048
openssl rsa -in sender.key -pubout -out sender.pub
```
![image](https://github.com/user-attachments/assets/fc1fea4d-f705-4f19-a0ad-13c49a193005)

## 3. Sign the hash:
Sign the checksum with the sender's private key.

```sh
openssl dgst -sha256 -sign sender.key -out sample.txt.sig sample.txt.sha256
```
![image](https://github.com/user-attachments/assets/ca3b8e97-b941-4e7d-af52-6a2ec0fc2076)


## 4. Check the connection of Bob
```sh
ssh bob@10.9.0.6
```

![image](https://github.com/user-attachments/assets/9b47d910-112b-4108-8054-43ca5d3d49c9)

## 5. After succesfully connected, Send file to Bob:

```sh
scp sample.txt sample.txt.sig sender.pub bob@10.9.0.6:/home/user/
```

![image](https://github.com/user-attachments/assets/83534012-db97-4f2c-8b87-ceb6b268394f)

## On Bob Side
## 1. Install OpenSSH server:

```sh
apt update
apt install -y openssh-server
```
![image](https://github.com/user-attachments/assets/4c90adec-e0d9-4bcf-907e-cb22f8ccef34)

## 2. Run the server and check the connection:

First, Start the SSH server

```sh
service ssh start
```
Then, Check if SSH is running

```sh
netstat -tuln | grep :22
```

![image](https://github.com/user-attachments/assets/70c9dc1b-1636-4d86-8b28-7cad8f3efe4c)

## 3. Set up the Bob user:

```sh
useradd -m -s /bin/bash bob
passwd bob
```

![image](https://github.com/user-attachments/assets/9df4fd2d-60c5-4409-9566-bbd26587d940)

## 4. After Receive file success, We must Verify the file’s checksum:
Recompute the hash and compare with the signed checksum
```sh
openssl dgst -sha256 sample.txt > sample.txt.sha256
openssl dgst -sha256 -verify sender.pub -signature sample.txt.sig sample.txt.sha256
```

After enter these, I get ```Verify OK``` that mean the signature matches
![image](https://github.com/user-attachments/assets/accad434-88e5-4673-a733-b7c4307700ef)

## 5. Then, Compare the integrity

Compare the original hash with the recomputed hash

```sh
diff sample.txt.sha256 <(openssl dgst -sha256 sample.txt)
```
And you can see, it show nothing so there are no differences are found, the file’s integrity is intact.

## 6. Last check:

```sh
cat sample.txt.sha256
openssl dgst -sha256 sample.txt
```
If the hashes are identical, integrity is verified
#### Bob: 
![image](https://github.com/user-attachments/assets/710e6e39-385f-44fc-8e76-2d10bf307656)

![image](https://github.com/user-attachments/assets/4208d39e-48d0-4fe0-8bc2-06020c8f1280)

So The hashes are identical. Integrity is verified


# Task 2. Transfering encrypted file and decrypt it with hybrid encryption

**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:
## On Alice Side

## 1. Create the Plaintext File
```sh
echo "This is a sample file for task 2" > sample.txt
```

## 2. Generate a Checksum
Create a SHA-256 hash of sample.txt to ensure its integrity.

```sh
openssl dgst -sha256 sample.txt > sample.txt.sha256
```

## 3. Generate an RSA Key Pair
Generate a private key (sender.key) and a corresponding public key (sender.pub) for signing the hash.
```sh
openssl genrsa -out sender.key 2048
openssl rsa -in sender.key -pubout -out sender.pub
```

![image](https://github.com/user-attachments/assets/c44ad4e1-6687-4741-84b5-79d0df5d2b67)


## 4. Sign the Hash
Sign the checksum file with the sender's private key to create a signature, ensuring authenticity.

```sh
openssl dgst -sha256 -sign sender.key -out sample.txt.sig sample.txt.sha256
```

## 5. Encrypt the File with a Symmetric Key (Hybrid Encryption)
### 1. Generate a random symmetric AES-256 key.
```sh
openssl rand -hex 32 > symmetric_key.bin
```
### 2. Encrypt sample.txt with AES-256 using the generated symmetric key.
```sh
openssl enc -aes-256-cbc -salt -in sample.txt -out encrypted_file.bin -pass file:symmetric_key.bin
```
### 3. Encrypt the symmetric key with Bob’s RSA public key (bob_pub.pem) so that only Bob can decrypt it.
```sh
openssl rsautl -encrypt -inkey /home/alice/bob_pub.pem -pubin -in symmetric_key.bin -out encrypted_key.bin
```

![image](https://github.com/user-attachments/assets/32f838eb-0b8d-4984-bd15-ed841aa8b122)

## 6. Transfer the Files to Bob
Use scp to transfer the files to Bob's IP (10.9.0.6):

```sh
scp encrypted_file.bin encrypted_key.bin sample.txt.sig sender.pub bob@10.9.0.6:/home/bob/
```
![image](https://github.com/user-attachments/assets/91b66d10-01ca-4158-8875-7f0b5b26e22a)

## On Bob Side

## 1. Generate Bob’s Private Key (bob_priv.pem) and Bob’s Public Key (bob_pub.pem)

```sh
openssl genpkey -algorithm RSA -out /root/bob_priv.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -in /root/bob_priv.pem -pubout -out /root/bob_pub.pem
```
![image](https://github.com/user-attachments/assets/6acc7493-6e90-404c-9654-a30b928f8cdd)


## 2. Send public key to Alice

Use scp to transfer the files to Alice's IP (10.9.0.5):
```sh
scp /root/bob_pub.pem alice@10.9.0.5:/home/alice/
```
![image](https://github.com/user-attachments/assets/707a886c-8db7-490c-ad8f-6cd902ac6570)

## 3. Decrypt the Symmetric Key with RSA after receive success

Decrypt the symmetric key using Bob’s private RSA key (bob_priv.pem).
```sh
openssl rsautl -decrypt -inkey /root/bob_priv.pem -in encrypted_key.bin -out decrypted_symmetric_key.bin
```

## 4. Decrypt the Encrypted File

Use the decrypted symmetric key to decrypt encrypted_file.bin, obtaining the original sample.txt.
```sh
openssl enc -d -aes-256-cbc -in encrypted_file.bin -out sample.txt -pass file:decrypted_symmetric_key.bin
```

![image](https://github.com/user-attachments/assets/f7a4ac3f-2c16-49aa-b49f-5dfee58d3bf7)


## 5. Verify the File’s Authenticity (Signature Verification)
Verify the file's signature to confirm authenticity.
```sh
openssl dgst -sha256 sample.txt > sample.txt.sha256
openssl dgst -sha256 -verify sender.pub -signature sample.txt.sig sample.txt.sha256
```
If the signature matches, the output will show:

```
Verified OK
```

![image](https://github.com/user-attachments/assets/86e211c1-dc8b-4357-a2c1-d344e36ea1a8)

## 5. Verify the File’s Authenticity (Signature Verification)
Compare the original hash (sample.txt.sha256) with the newly computed hash to confirm integrity.
```sh
diff sample.txt.sha256 <(openssl dgst -sha256 sample.txt)
```
If the signature matches, the output will show:

![image](https://github.com/user-attachments/assets/e879b575-4f72-4693-a12b-06edcc5718ac)

# Task 3: Firewall configuration

**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:

I will use 2 different VMS, one is SeedUbuntu-20.04 in Oracle VirtualBox and my cmd Ubuntu canh123

![image](https://github.com/user-attachments/assets/27679068-96f3-4c38-a504-ada0406f70c6)

I will install iptables on my ubuntu canh123 using 

```
sudo apt install iptables
```

![image](https://github.com/user-attachments/assets/07833ec8-bad4-492d-aa64-05b4f071f963)


Then, Use command to list all rule of iptables

```
sudo iptables -L
```
![image](https://github.com/user-attachments/assets/44d0e71e-eef5-463d-99ff-9daebf5f3deb)


Then, i will install openssh-server on canh123 ubuntu

```
sudo apt install openssh-server
```
Then, i will use 2 command to start and check status of ssh server

```
sudo service ssh start
sudo service ssh status
```

![image](https://github.com/user-attachments/assets/e9808ead-276e-4bca-a08d-0106ee9c4bfd)

Then, i will install apache2 server on canh123 ubuntu

```
sudo apt install apache2
```
![image](https://github.com/user-attachments/assets/72b3f633-7c23-46a0-b274-81058ad477a5)

Then, i will use 2 command to start and check status of apache2 server

```
sudo service apache2 start
sudo service apache2 status
```

![image](https://github.com/user-attachments/assets/f221aa74-8cb4-439c-8a7c-8082a3c3df67)

Next, i can see the ip of canh123 ubuntu using command

```
sudo ip a
```

![image](https://github.com/user-attachments/assets/e412e270-404f-4d98-adfd-aa34d9b96e7b)

And the ip is: `172.27.3.51`

In SeedUbuntu, i have ``` ip addr ```

![image](https://github.com/user-attachments/assets/4dcab27a-4d82-47b8-9906-abde541681d6)

We can see it have many ip address, so for make sure i get the true IP, I will ping from SeedUbuntu to canh123 and use tcpdump to listen the packet

First, in canh123, i enter:

```sh
sudo tcpdump -i eth0 icmp -n
```
-n flag disables hostname resolution, so the output will show raw IP addresses

![image](https://github.com/user-attachments/assets/f78cea0b-39f7-4681-8b92-0cd5bfe90797)

After that, I ping from SeedUbuntu

![image](https://github.com/user-attachments/assets/23fbb710-5cab-48b8-9e56-2798f72c131e)

And from canh123, I can catch the icmp packets:

![image](https://github.com/user-attachments/assets/ecbc9316-7710-4917-99e6-7410eacdd96f)

So the ip address of SeedUbuntu is: `172.27.0.1`

### Block http, icmp, ssh requests from the other host.

Block http using command

```
sudo iptables -A INPUT -p tcp --dport 80 -s 172.27.0.1 -j DROP
```

This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
- **`-p tcp`**: Specifies the protocol as TCP (Transmission Control Protocol).
- **`--dport 80`**: Specifies that the rule applies to traffic destined for port 80, which is commonly used for HTTP.
- **`-s 172.27.0.1`**: Specifies that the rule applies to traffic originating from the IP address `172.27.0.1`.
- **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.
  
![image](https://github.com/user-attachments/assets/4158cb0d-c41f-4cd5-9060-c2f4786e2342)

and then next, we can try to access http from SeedUbuntu Machine using command

```
curl http://172.27.3.51
```

![image](https://github.com/user-attachments/assets/31a584ff-d38c-455c-ab9b-05f05d9da1d8)


Block both incoming and outoging icmp using command

```
sudo iptables -A INPUT -p icmp --icmp-type echo-request -s 172.27.0.1 -j DROP
sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -d 172.27.0.1 -j DROP
```
These commands do the following:

1. **`sudo iptables -A INPUT -p icmp --icmp-type echo-request -s 172.27.0.1 -j DROP`**:
2. **`sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -d 172.27.0.1 -j DROP`**:
   - **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
   - **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
   - 
   - **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
   - **`-A OUTPUT`**: Appends a rule to the `OUTPUT` chain, which controls outgoing network traffic.
   
   - **`-p icmp`**: Specifies the protocol as ICMP (Internet Control Message Protocol), which is used for ping requests and replies.
   - **`--icmp-type echo-request`**: Specifies that the rule applies to ICMP echo requests (ping requests).
     
   - **`-s 172.27.0.1`**: Specifies that the rule applies to traffic originating from the IP address `172.27.0.1`.
   - **`-d 172.27.0.1`**: Specifies that the rule applies to traffic destined for the IP address `172.27.0.1`.

   - **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.

![image](https://github.com/user-attachments/assets/97a13b9a-7f20-4217-99da-1b6ed8a2c4bd)

and then next, we can try to ping from SeedUbuntu Machine using command

```
ping 172.27.3.51
```

![image](https://github.com/user-attachments/assets/e40795ef-6ba6-49ce-8990-501e5eb9f228)

As you can see, they cannot comunate each other through icmp 

![image](https://github.com/user-attachments/assets/a8803722-063d-422d-bcfb-92e7ef437ba7)


Block ssh request

We can block all SSH requests from other hosts using command

```
sudo iptables -A INPUT -p tcp --dport 22 -s 172.27.0.1 -j DROP
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-A INPUT`**: Appends a rule to the `INPUT` chain, which controls incoming network traffic.
- **`-p tcp`**: Specifies the protocol as TCP (Transmission Control Protocol).
- **`--dport 22`**: Specifies that the rule applies to traffic destined for port 22, which is commonly used for SSH (Secure Shell) connections.
- **`-s 172.27.0.1`**: Specifies that the rule applies to traffic originating from the IP address `172.27.0.1`.
- **`-j DROP`**: Specifies the action to take, which is to drop the packet, effectively blocking the connection.

![image](https://github.com/user-attachments/assets/08b5b1d7-ab89-42ba-a353-d804d3fd7e38)



On canh123 ubuntu, we can confirm that openssh-server is still running

![image](https://github.com/user-attachments/assets/ddf399e3-c031-447d-b716-b08fd4a0f34b)

On SeedUbuntu Machine, we can confirm that openssh-server is still running

![image](https://github.com/user-attachments/assets/411dd66c-bd70-44a7-ac51-8803bde114b4)

Try to send file.txt from SeedUbuntu using command
```
scp test.txt canh123@172.27.3.51:/home
```

This command does the following:

- **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
- **`test.txt`**: Specifies the file (`test.txt`) to be transferred.
- **`canh123@172.27.3.51:/home`**: Specifies the remote server's username (`canh123`), IP address (`172.27.3.51`), and the destination directory (`/home/`) where the file will be copied.

![image](https://github.com/user-attachments/assets/5d2831f2-d117-42d7-a4d7-c886af41e52d)

The picture show that we cant connect to canh123's ssh channel

### Unblock http, icmp, ssh requests from the other host.

Unblock HTTP:

Number-list iptables

![image](https://github.com/user-attachments/assets/41a6793a-d81d-40ef-be91-221ff755092c)

On canh123 ubuntu, we can unblock http request using command
```
sudo iptables -D INPUT 1
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
- **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

On SeedUbuntu using command to access http:
```
curl http://172.27.3.51
```

![image](https://github.com/user-attachments/assets/ea47e9ef-a289-4d44-963b-29f08b06c671)

Unblock ICMP:

![image](https://github.com/user-attachments/assets/bd04068f-73e4-40d4-9e39-b2939893cf40)

On canh123 ubuntu, we can unblock icmp request using command
```
sudo iptables -D INPUT 1
sudo iptables -D OUTPUT 1
```
These commands do the following:

1. **`sudo iptables -D INPUT 1`**:
1. **`sudo iptables -D OUTPUT 1`**:
   - **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
   - **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
   - **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
   - **`-D OUTPUT`**: Deletes a rule from the `OUTPUT` chain, which controls outgoing network traffic.
   - **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

the number based on the picture number-list iptables

![image](https://github.com/user-attachments/assets/a2a07172-5509-45b1-b351-3b8b2eed9764)

On SeedUbuntu, we can try to ping to canh123 ubuntu using command
```
ping 172.27.3.51
```

![image](https://github.com/user-attachments/assets/741b2d60-61a6-452c-830c-be0503cf9550)

We can see on the picture that, there is request and reply between 2 hosts

Unblock SSH

![image](https://github.com/user-attachments/assets/e2ed6d89-fee2-4942-b185-be10654d303e)

On canh123 ubuntu, we can unblock ssh using command 
```
sudo iptables -D INPUT 1
```
This command does the following:

- **`sudo`**: Executes the command with superuser privileges, which are required for modifying firewall rules.
- **`iptables`**: Invokes the `iptables` utility to manage firewall rules on a Linux-based system.
- **`-D INPUT`**: Deletes a rule from the `INPUT` chain, which controls incoming network traffic.
- **`1`**: Specifies the rule number to delete from the `INPUT` chain. In this case, `1` refers to the first rule in the list.

![image](https://github.com/user-attachments/assets/bba1373e-4099-4727-a33d-bb65f535711d)

On the woox ubuntu, i will set all priority for home folder using command
```
chmod 777 /home
```
![image](https://github.com/user-attachments/assets/bf61cb38-7d91-4bc2-892b-ac99e344ce0a)

On SeedUnbuntu, we can try to send test.txt file to canh123 ubuntu using command
```
scp test.txt canh123@172.27.3.51:/home
```
This command does the following:

- **`scp`**: Invokes the secure copy protocol to transfer files between a local machine and a remote machine over SSH.
- **`test.txt`**: Specifies the file (`test.txt`) to be transferred.
- **`canh123@172.27.3.51:/home`**: Specifies the remote server's username (`canh123`), IP address (`172.27.3.51`), and the destination directory (`/home/`) where the file will be copied.

  ![image](https://github.com/user-attachments/assets/cf2b972b-5b23-45b0-91ac-711eb05ea0d7)
