# Module

What are the style guidelines and best practices for our engineering team.

# Module

---

Modules allow programmers to export certain part of kernel and compile only certain part. This allows quick modification of kernel code without recompiling whole kernel

# Cautions

---

Module name and all the variable name must be different from that of existing kernel.

```bash
#upload module
insmod pxt4.ko 

#remove module
rmmod pxt4.ko
```

## Check bulitin modules

```
cat /lib/modules/$(uname -r)/modules.builtin
```

## Errors

Multiple definition of 'init_module'

[Module%20d5050bb5ed374d9b9fee008dfb9aa184/Untitled](Module%20d5050bb5ed374d9b9fee008dfb9aa184/Untitled)

EXPORT_SYMBOL <- comment out

## System call <-> module

```clike
#include <linux/module.h>	/* Required for EXPORT_SYMBOL */
#include <linux/linkage.h>
#include <linux/errno.h>

long (* mycall_ptr) (int i) = NULL;	/* Function pointer */
EXPORT_SYMBOL(mycall_ptr);	/* Export the function pointer */

asmlinkage long sys_mycall(int i)
{
	return mycall_ptr ? mycall_ptr(i) : -ENOSYS;
}
--------------------------------------------------
#include <linux/kernel.h>	/* Required by all modules */
#include <linux/module.h>	/* Required for printk */
#include <linux/init.h>		/* Required for the macros */

extern void * mycall_ptr;	/* Declare the extern function pointer  */

long mycall_real(int i)
{
	printk(KERN_INFO "I am the function to do the real job for mycall\n");
	return 0;
}

static int __init use_function_pointer_init(void)
{
	printk(KERN_INFO "Will assign a new value to mycall_ptr \n");
	mycall_ptr = mycall_real;

	return 0;
}

static void __exit use_function_pointer_exit(void)
{
	printk(KERN_INFO "Will restore the old value of mycall_ptr\n");
	mycall_ptr = NULL;	
}

module_init(use_function_pointer_init);
module_exit(use_function_pointer_exit);

```

## Module Make file

```jsx
#module make file
KDIR :=/lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

default:
        $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules

clean:
        rm *.mod.*
        rm *.ko
        rm *.o
```
