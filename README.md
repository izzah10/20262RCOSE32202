Project 1 brief summary : 

1. Environment used :  Windows 11, WSL2, Ubuntu 22.04.5 LTS, Linux kernel 6.6.36
2. In this project I added three new system calls to the Linux kernel that work as a
stack (LIFO): `push` (335), `pop` (336) and `clear` (337) as provided in the guidelines.
3. I also wrote a user program to test them. 
4. Implementation :
- **syscall registration** :  I added the three entries to `syscall_64.tbl` so the
  kernel knows which function to run for each syscall number. I also declared them
  in `syscalls.h`, using `asmlinkage` because they are called from assembly code.
- **Stack in the kernel (`sslab_my_stack.c`):** The stack is a singly linked list
  with a global `top` pointer, so the data stays in kernel memory between syscalls.
  - push: allocate a node with kmalloc before taking the lock, then link it in.
    The order of the two pointer updates matters, otherwise the old stack becomes
    unreachable.
  - pop :  return `-EINVAL` on an empty stack. Otherwise save the value, move `top`
    to the next node, and only then `kfree` the old node.
  - clear :  free every node in a loop and return how many were removed.
  - A mutex protects every operation so two processes can't change the stack at the
    same time. I also had to make sure every return path unlocks it.
- **Makefile:** I added `sslab_my_stack.o` to `obj-y` so the file is built into the
                kernel.
- **Test program (`call_my_stack.c`):** It clears the stack first because kernel
  memory persists between runs , pushes 5 random numbers, pops 3, and compares them
  with the last 3 pushed values to check the LIFO order. The final `clear` should
  remove the remaining 2 nodes.

5. What i learned from this project :
- Kernel code cannot use local variables to keep state between syscalls.
- Memory management (kmalloc/kfree) and locking order are easily to get wrong
  also able to crash the whole kernel.

