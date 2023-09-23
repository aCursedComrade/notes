- [NASM Docs](https://www.nasm.us/doc/)
	- [Quick tutorial](https://cs.lmu.edu/~ray/notes/nasmtutorial/)
	- [ASM tutor](https://asmtutor.com/#lesson29)
- [x86-64 Assembly Language Programming with Ubuntu](https://open.umn.edu/opentextbooks/textbooks/733) ([PDF](http://www.egr.unlv.edu/~ed/assembly64.pdf))
- [WikiBook: x86 Assembly](https://en.wikibooks.org/wiki/X86_Assembly)
- [x86 instructions reference by Felix Cloutier](https://www.felixcloutier.com/x86/)
- [Pseudo Ops](https://sourceware.org/binutils/docs/as/Pseudo-Ops.html) (parent document is about GNU assembler `as`)
- System call tables:
	- [Linux x86-64 system call table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/) (cross-references included)
	- [syscall.sh](https://syscall.sh/)
- Detailed manuals for Linux (POSIX) system calls are available in `man` pages (use `man <syscall/data type>` in the terminal)
- Find constants by using `grep` in `/usr/include/linux` directory which contains source headers or looking through the [kernel archive](https://www.kernel.org/)

***

> generic_server.s

```asm
.intel_syntax noprefix
.global _start

# a bare minimum web server made for pwn.college module
# do not expose on important hosts

# Remember the order Neo!
# rdi, rsi, rdx, r8, rbx, r8
# rax holds the syscall value and then its return value

# performing syscall modifies values at rcx and r11

.section .data
header: .ascii "HTTP/1.0 200 OK\r\n\r\n"
geneic_response: .ascii "Hello there..."
sockaddr:
    .2byte 0x2                              # AF_INET
    .2byte 0x5000                           # port 80 (written as little endian so its placed as big endian on memory)
    .4byte 0x0                              # bind to 0.0.0.0
    .8byte 0x0                              # fill the rest with 0s

.section .text
_start:
    # make a socket
    mov rdi, 2                              # address family AF_INET
    mov rsi, 1                              # type SOCK_STREAM (TCP)
    mov rdx, 0                              # protocol (IP)
    mov rax, 41                             # sys_socket
    syscall

    # bind the socket to address:port
    mov rdi, rax                            # socket fd (from last socket creation)
    lea rsi, [rip+sockaddr]                 # sockaddr struct address
    mov rdx, 16                             # struct size to read
    mov rax, 49                             # sys_bind
    syscall

    # start listening
    # rdi still holds the socket fd
    xor rsi, rsi                            # no backlog
    mov rax, 50                             # sys_listen
    syscall
    mov rbx, rdi                            # server socket to be referred later

_conn_loop:
    # accept the connection
    mov rdi, rbx
    xor rsi, rsi                            #
    xor rdx, rdx                            # zero out other 2 args
    mov rax, 43                             # sys_accept
    syscall
    mov r15, rax                            # client sock

    # using fork for multiprocessing
    mov rax, 57                             # sys_fork
    syscall
    test rax, rax
    jnz _con_close                          # loop back if we are not child

    # close the server socket
    mov rdi, rbx                            # server sock
    mov rax, 3                              # sys_close
    syscall

    # read the request
    mov rdi, r15                            # client sock
    mov rsi, rsp                            # point to stack as buffer
    mov rdx, 512                            # max read size
    mov rax, 0                              # sys_read
    syscall
    mov r12, rax                            # length of request
    dec r12

    # extract the path
    mov r8, rsp                             # buf pointer

# find the start of the path
_path_start:
    cmp byte ptr [r8], ' '
    je _start_found
    inc r8
    jmp _path_start

_start_found:
    inc r8
    mov r9, r8                              # path start

# read up to the end of path string
_path_extract:
    cmp byte ptr [r8], ' '
    je _extracted
    inc r8
    jmp _path_extract

_extracted:
    mov byte ptr [r8], 0x0                  # place a null char to terminate the path string

    # a VERY LAZY way to check the http verb
    cmp byte ptr [rsp], 'G'
    je _get_handler
    cmp byte ptr [rsp], 'P'
    je _post_handler
    jmp _generic_handler

# handle a get request (read from a file)
_get_handler:
    lea rbp, [rsp+1024] # buf space
    xor rbx, rbx # size

    # open the file at path
    mov rdi, r9                             # path string ptr
    mov rsi, 0                              # no flags
    mov rdx, 0                              # O_RDONLY
    mov rax, 2                              # sys_open
    syscall
    mov r10, rax                            # file fd

    # read file at path
    mov rdi, rax                            # file fd
    mov rsi, rbp                            # read buf
    mov rdx, 512                            # read size
    mov rax, 0                              # sys_read
    syscall
    mov rbx, rax                            # size

    # close the file
    mov rdi, r10                            # file fd
    mov rax, 3                              # sys_close
    syscall

    # send header
    mov rdi, r15                            # client sock
    lea rsi, [rip+header]                   # pointer to our header
    mov rdx, 19                             # write size
    mov rax, 1                              # sys_write
    syscall

    # send content
    mov rdi, r15                            # client sock
    mov rsi, rbp                            # read buf
    mov rdx, rbx                            # size
    mov rax, 1                              # sys_write
    syscall

    jmp _end

# handle a post request (writes to a file)
_post_handler:
    lea rbp, [rsp+1024]                     # buf space
    lea rbx, [rsp+r12]                      # end of request/buffer
    xor r8, r8                              # content size

# extract the content (starts from the end and breaks on the first '\n')
_get_content:
    cmp byte ptr [rbx], '\n'
    je _content_found
    dec rbx
    inc r8
    jmp _get_content

_content_found:
    inc rbx

    # open/create the file at path
    mov rdi, r9                             # path string ptr
    mov rsi, 1|0100                         # O_WRONLY|O_CREAT
    mov rdx, 0777                           # 0777 - rwx for all
    mov rax, 2                              # sys_open
    syscall
    mov r10, rax                            # file fd

    # write the content to file
    mov rdi, rax                            # file fd
    lea rsi, [rbx]                          # content buffer
    mov rdx, r8                             # write size
    mov rax, 1                              # sys_write
    syscall

    # close the file
    mov rdi, r10                            # file fd
    mov rax, 3                              # sys_close
    syscall

    # send header
    mov rdi, r15                            # client sock
    lea rsi, [rip+header]                   # pointer to our header
    mov rdx, 19                             # write size
    mov rax, 1                              # sys_write
    syscall

    jmp _end

# in case our server received something else other than GET, POST
_generic_handler:
    # send header
    mov rdi, r15                            # client sock
    lea rsi, [rip+header]                   # pointer to our header
    mov rdx, 19                             # write size
    mov rax, 1                              # sys_write
    syscall

    # generic response
    mov rdi, r15                            # client sock
    lea rsi, [rip+geneic_response]
    mov rdx, 14                             # size
    mov rax, 1                              # sys_write
    syscall

_end:
    # exit
    mov rdi, 0
    mov rax, 60
    syscall

_con_close:
    # close the client socket
    mov rdi, r15                            #socket fd
    mov rax, 3                              # sys_close
    syscall
    jmp _conn_loop                          # creates an event loop for the parent process
```