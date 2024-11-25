![image](https://github.com/user-attachments/assets/1fa0a5ff-67cf-4bee-86aa-400543ce4910)![image](https://github.com/user-attachments/assets/a004bfc7-28b0-4cc8-9847-d60bbb378586)# Lab #1, 22110012, Bach Duc Canh, INSE331280E_01FIE
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

## 6. Transfer the Files to Bob
Use scp to transfer the files to Bob's IP (10.9.0.6):

```sh
scp encrypted_file.bin encrypted_key.bin sample.txt.sig sender.pub bob@10.9.0.6:/home/bob/
```

## On Bob Side



# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:


## 4. Combine the header and encrypted body:

```sh
cat header.bin encrypted_body.bin > partially_encrypted.bmp
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage9.png?raw=true"><br>