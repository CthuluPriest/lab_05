#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/proc_fs.h>
#include <asm/uaccess.h>

MODULE_AUTHOR("Serge Staravoitau");

/* Module proc file name. */
#define CALC_PROC "calc"

/* Module proc files names. */
#define PROCFS_RESULT "calc_result"
#define PROCFS_FIRST "calc_first"
#define PROCFS_SECOND "calc_second"
#define PROCFS_OPERAND "calc_operand"

/* Procfs maximum buffer size. */
#define PROCFS_MAX_SIZE 16

/* Procs files names. */
static char names[][16] = {PROCFS_FIRST, PROCFS_SECOND, PROCFS_OPERAND, PROCFS_RESULT};

/* Indices fot the readers and writers. */
static int indices[4];

/* Files small chars buffers. */
static char** procfs_buffer;

/* Procfs files entries structures. */
static struct proc_dir_entry** child_procs;

/* Hooks when reading from the module proc file. */
static int proc_read(char* buffer, char** buffer_location, off_t offset, int buffer_length, int* eof, void* data)
{
	int ret, number;
	long a, b, result = 0;
	char op, *end;

	number = *((int*) data);
	printk(KERN_INFO "Reading /proc/%s", names[number - 1]);

	if (offset > 0) {
		ret = 0;
	} else {
		if (number == 4) {
			a = simple_strtol(procfs_buffer[0], &end, 10);
			if (a == 0 && (*end == procfs_buffer[0][0])) {
				return sprintf(buffer, "\n%s\n\n", "First operand is not integer.");
			}

			b = simple_strtol(procfs_buffer[1], &end, 10);
			if (b == 0 && (*end == procfs_buffer[1][0])) {
				return sprintf(buffer, "\n%s\n\n", "Second operand is not integer.");
			}
			op = procfs_buffer[2][0];
			switch (op) {	
				case '+': result = a + b; break;
				case '-': result = a - b; break;
				case 'x': result = a * b; break;
				case '/':
				if (b == 0) {
					return sprintf(buffer, "\n%s\n\n",
					"Division by zero!");
				}
				result = a / b;
				break;

				default: 
				return sprintf(buffer, "\nUnknown operand: %c\n\n", op);
			}
			ret = sprintf(buffer, "\n%ld %c %ld = %ld\n\n", a, op, b, result);
		} else {
			ret = sprintf(buffer, "\n%s\n\n", procfs_buffer[number - 1]);
		}
	}

	return ret;
}

/* Hooks when writing the data into the module proc file. */
static int proc_write(struct file* file, const char* buffer,unsigned long count, void* data)
{
	int procfs_buffer_size, index;

	if (count >= PROCFS_MAX_SIZE) {
		procfs_buffer_size = PROCFS_MAX_SIZE - 1;
	} else {
		procfs_buffer_size = count;
	}

	index = *((int*) data);
	if (index != 4) {
		if (copy_from_user(procfs_buffer[index - 1], buffer, procfs_buffer_size)) {
			printk(KERN_ERR "Failed to write to /proc/%s\n", names[index - 1]);
			return -EFAULT;
		}
		procfs_buffer[index - 1][procfs_buffer_size - 1] = '\0';
	}

	printk(KERN_INFO "Writing to /proc/%s\n", names[index - 1]);
	return procfs_buffer_size;
}



int __init init_module(void){
	int i;

	printk(KERN_INFO "Calc module is loaded.\n");

	procfs_buffer = (char**) kmalloc(sizeof(char*) * 4, GFP_KERNEL);
	for (i = 0; i < 4; i++) {
		procfs_buffer[i] = (char*) kmalloc(sizeof(char) * PROCFS_MAX_SIZE, GFP_KERNEL);
		procfs_buffer[i][0] = '\0';
	}

	child_procs = (struct proc_dir_entry**) kmalloc(sizeof(struct proc_dir_entry *) * 4, GFP_KERNEL);

	for (i = 0; i < 4; i++) {
		child_procs[i] = create_proc_entry(names[i], 0666, NULL);
		if (!child_procs[i]) {
			remove_proc_entry(names[i], NULL);
			printk(KERN_ERR "Failed to create /proc/%s", names[i]);
			return -ENOMEM;
		}
		indices[i] = i + 1;
		child_procs[i]->read_proc = proc_read;
		child_procs[i]->write_proc = proc_write;
		child_procs[i]->data	= (void*) &indices[i];
	}

	printk(KERN_INFO "Calc procs were created.\n");
	return 0;
}

void __exit cleanup_module(void){
	int i;

	for (i = 0; i < 4; i++) {
		remove_proc_entry(names[i], NULL);
		kfree(procfs_buffer[i]);
	}

	kfree(child_procs);
	kfree(procfs_buffer);

	printk(KERN_INFO "Calc procs were removed.\n");
	printk(KERN_INFO "Calc module is unloaded.\n");
}
