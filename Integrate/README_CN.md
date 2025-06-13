# 为非GKI或QGKI内核集成Re:Kernel
在开始前，你必须有能力从你设备的内核源码编译出一个可以开机并且能正常使用的内核，如果内核不开源，这通常难以做到。

如果你已经做好了上述准备，那么你这里有两种方法可以将Re:Kernel集成到你的内核当中

## 使用Kernel Modifier自动修改
首先 你需要下载并将Kernel Modifier放入内核源代码的根目录中 然后使用17及以上版本的Java运行Kernel Modifier, 不出意外的话 Modifier 会自动修改内核源码并将ReKernel插入到你的内核当中 接下来你只需要重新编译一次内核即可

## 手动修改
首先 你应该创建一个头或源文件 里面存放NETLINK的代码 其UNIT应为26 用户端口为100 包体大小为128
```C++
#include <linux/init.h>
#include <linux/types.h>
#include <net/sock.h>
#include <linux/netlink.h>
#include <linux/proc_fs.h>
#include <linux/freezer.h>

#define NETLINK_REKERNEL_MAX     		26
#define NETLINK_REKERNEL_MIN     		22
#define USER_PORT        			100
#define PACKET_SIZE 				128
#define MIN_USERAPP_UID 			(10000)
#define MAX_SYSTEM_UID  			(2000)
#define RESERVE_ORDER				17
#define WARN_AHEAD_SPACE			(1 << RESERVE_ORDER)

static struct sock *rekernel_netlink = NULL;
extern struct net init_net;
static int netlink_unit = NETLINK_REKERNEL_MIN;

static inline bool line_is_frozen(struct task_struct *task)
{
    return frozen(task->group_leader) || freezing(task->group_leader);
}

static int send_netlink_message(char *msg, uint16_t len) {
    struct sk_buff *skbuffer;
    struct nlmsghdr *nlhdr;

    skbuffer = nlmsg_new(len, GFP_ATOMIC);
    if (!skbuffer) {
        printk("netlink alloc failure.\n");
        return -1;
    }

    nlhdr = nlmsg_put(skbuffer, 0, 0, netlink_unit, len, 0);
    if (!nlhdr) {
        printk("nlmsg_put failaure.\n");
        nlmsg_free(skbuffer);
        return -1;
    }

    memcpy(nlmsg_data(nlhdr), msg, len);
    return netlink_unicast(rekernel_netlink, skbuffer, USER_PORT, MSG_DONTWAIT);
}

static void netlink_rcv_msg(struct sk_buff *skbuffer) { // Ignore recv msg.
}

static struct netlink_kernel_cfg rekernel_cfg = { 
    .input = netlink_rcv_msg,
};

static int rekernel_unit_show(struct seq_file *m, void *v)
{
	seq_printf(m, "%d\n", netlink_unit);
	return 0;
}

static int rekernel_unit_open(struct inode *inode, struct file *file)
{
	return single_open(file, rekernel_unit_show, NULL);
}

static const struct file_operations rekernel_unit_fops = {
	.open   = rekernel_unit_open,
	.read   = seq_read,
	.llseek   = seq_lseek,
	.release   = single_release,
	.owner   = THIS_MODULE,
};

static struct proc_dir_entry *rekernel_dir, *rekernel_unit_entry;

static int start_rekernel_server(void) {
  if (rekernel_netlink != NULL)
    return 0;
  for (netlink_unit = NETLINK_REKERNEL_MIN; netlink_unit < NETLINK_REKERNEL_MAX; netlink_unit++) {
    rekernel_netlink = (struct sock *)netlink_kernel_create(&init_net, netlink_unit, &rekernel_cfg);
    if (rekernel_netlink != NULL)
      break;
  }
  if (rekernel_netlink == NULL) {
    printk("Failed to create Re:Kernel server!\n");
    return -1;
  }
  printk("Created Re:Kernel server! NETLINK UNIT: %d\n", netlink_unit);
  rekernel_dir = proc_mkdir("rekernel", NULL);
  if (!rekernel_dir)
      printk("create /proc/rekernel failed!\n");
  else {
      char buff[32];
      sprintf(buff, "%d", netlink_unit);
      rekernel_unit_entry = proc_create(buff, 0644, rekernel_dir, &rekernel_unit_fops);
      if (!rekernel_unit_entry)
          printk("create rekernel unit failed!\n");
  }
  return 0;
}
```
然后，将调用添加到内核源代码中，如下:
```C++
// drivers/android/binder.c
static void binder_transaction(struct binder_proc *proc,
			       struct binder_thread *thread,
			       struct binder_transaction_data *tr, int reply,
			       binder_size_t extra_buffers_size)
{
........
    if (target_thread->transaction_stack != in_reply_to) {
			binder_user_error("%d:%d got reply transaction with bad target transaction stack %d, expected %d\n",
				proc->pid, thread->pid,
				target_thread->transaction_stack ?
				target_thread->transaction_stack->debug_id : 0,
				in_reply_to->debug_id);
			binder_inner_proc_unlock(target_thread->proc);
			return_error = BR_FAILED_REPLY;
			return_error_param = -EPROTO;
			return_error_line = __LINE__;
			in_reply_to = NULL;
			target_thread = NULL;
			goto err_dead_binder;
    }
		target_proc = target_thread->proc;
		target_proc->tmp_ref++;
		binder_inner_proc_unlock(target_thread->proc);
+   		if (start_rekernel_server() == 0) {
+			if (target_proc
+				&& (NULL != target_proc->tsk)
+				&& (NULL != proc->tsk)
+				&& (task_uid(target_proc->tsk).val <= MAX_SYSTEM_UID)
+				&& (proc->pid != target_proc->pid)
+				&& line_is_frozen(target_proc->tsk)) {
+     				char binder_kmsg[PACKET_SIZE];
+                       	snprintf(binder_kmsg, sizeof(binder_kmsg), "type=Binder,bindertype=reply,oneway=0,from_pid=%d,from=%d,target_pid=%d,target=%d;", proc->pid, task_uid(proc->tsk).val, target_proc->pid, task_uid(target_proc->tsk).val);
+         			send_netlink_message(binder_kmsg, strlen(binder_kmsg));
+			}
+   		}
	} else {
		if (tr->target.handle) {
			struct binder_ref *ref;

			/*
			 * There must already be a strong ref
			 * on this node. If so, do a strong
			 * increment on the node to ensure it
			 * stays alive until the transaction is
			 * done.
			 */
			binder_proc_lock(proc);
			ref = binder_get_ref_olocked(proc, tr->target.handle,
						     true);
			if (ref) {
				target_node = binder_get_node_refs_for_txn(
						ref->node, &target_proc,
						&return_error);
			} else {
				binder_user_error("%d:%d got transaction to invalid handle\n",
						  proc->pid, thread->pid);
				return_error = BR_FAILED_REPLY;
			}
			binder_proc_unlock(proc);
		} else {
			mutex_lock(&context->context_mgr_node_lock);
			target_node = context->binder_context_mgr_node;
			if (target_node)
				target_node = binder_get_node_refs_for_txn(
						target_node, &target_proc,
						&return_error);
			else
				return_error = BR_DEAD_REPLY;
			mutex_unlock(&context->context_mgr_node_lock);
			if (target_node && target_proc->pid == proc->pid) {
				binder_user_error("%d:%d got transaction to context manager from process owning it\n",
						  proc->pid, thread->pid);
				return_error = BR_FAILED_REPLY;
				return_error_param = -EINVAL;
				return_error_line = __LINE__;
				goto err_invalid_target_handle;
			}
		}
		if (!target_node) {
			/*
			 * return_error is set above
			 */
			return_error_param = -EINVAL;
			return_error_line = __LINE__;
			goto err_dead_binder;
		}
		e->to_node = target_node->debug_id;
+   		if (start_rekernel_server() == 0) {
+			if (target_proc
+				&& (NULL != target_proc->tsk)
+				&& (NULL != proc->tsk)
+				&& (task_uid(target_proc->tsk).val > MIN_USERAPP_UID)
+				&& (proc->pid != target_proc->pid)
+				&& line_is_frozen(target_proc->tsk)) {
+     				char binder_kmsg[PACKET_SIZE];
+                       	snprintf(binder_kmsg, sizeof(binder_kmsg), "type=Binder,bindertype=transaction,oneway=%d,from_pid=%d,from=%d,target_pid=%d,target=%d;", tr->flags & TF_ONE_WAY, proc->pid, task_uid(proc->tsk).val, target_proc->pid, task_uid(target_proc->tsk).val);
+         			send_netlink_message(binder_kmsg, strlen(binder_kmsg));
+			}
+   		}
		if (security_binder_transaction(proc->cred,
						target_proc->cred) < 0) {
			return_error = BR_FAILED_REPLY;
			return_error_param = -EPERM;
			return_error_line = __LINE__;
			goto err_invalid_target_handle;
		}
		binder_inner_proc_lock(proc);
........
}
```
```C++
// drivers/android/binder_alloc.c
static struct binder_buffer *binder_alloc_new_buf_locked(
				struct binder_alloc *alloc,
				size_t data_size,
				size_t offsets_size,
				size_t extra_buffers_size,
				int is_async,
				int pid)
{
+	struct task_struct *proc_task = NULL;
	struct rb_node *n = alloc->free_buffers.rb_node;
	struct binder_buffer *buffer;
	size_t buffer_size;
	struct rb_node *best_fit = NULL;
	void __user *has_page_addr;
	void __user *end_page_addr;
	size_t size, data_offsets_size;
	int ret;

	mmap_read_lock(alloc->vma_vm_mm);
	if (!binder_alloc_get_vma(alloc)) {
		mmap_read_unlock(alloc->vma_vm_mm);
		binder_alloc_debug(BINDER_DEBUG_USER_ERROR,
				   "%d: binder_alloc_buf, no vma\n",
				   alloc->pid);
		return ERR_PTR(-ESRCH);
	}
	mmap_read_unlock(alloc->vma_vm_mm);

	data_offsets_size = ALIGN(data_size, sizeof(void *)) +
		ALIGN(offsets_size, sizeof(void *));

	if (data_offsets_size < data_size || data_offsets_size < offsets_size) {
		binder_alloc_debug(BINDER_DEBUG_BUFFER_ALLOC,
				"%d: got transaction with invalid size %zd-%zd\n",
				alloc->pid, data_size, offsets_size);
		return ERR_PTR(-EINVAL);
	}
	size = data_offsets_size + ALIGN(extra_buffers_size, sizeof(void *));
	if (size < data_offsets_size || size < extra_buffers_size) {
		binder_alloc_debug(BINDER_DEBUG_BUFFER_ALLOC,
				"%d: got transaction with invalid extra_buffers_size %zd\n",
				alloc->pid, extra_buffers_size);
		return ERR_PTR(-EINVAL);
	}
+	if (is_async
+		&& (alloc->free_async_space < 3 * (size + sizeof(struct binder_buffer))
+		|| (alloc->free_async_space < WARN_AHEAD_SPACE))) {
+		rcu_read_lock();
+		proc_task = find_task_by_vpid(alloc->pid);
+		rcu_read_unlock();
+		if (proc_task != NULL && start_rekernel_server() == 0) {
+			if (line_is_frozen(proc_task)) {
+     				char binder_kmsg[PACKET_SIZE];
+                       	snprintf(binder_kmsg, sizeof(binder_kmsg), "type=Binder,bindertype=free_buffer_full,oneway=1,from_pid=%d,from=%d,target_pid=%d,target=%d;", current->pid, task_uid(current).val, proc_task->pid, task_uid(proc_task).val);
+         			send_netlink_message(binder_kmsg, strlen(binder_kmsg));
+			}
+		}
+	}
	if (is_async &&
	    alloc->free_async_space < size + sizeof(struct binder_buffer)) {
		binder_alloc_debug(BINDER_DEBUG_BUFFER_ALLOC,
			     "%d: binder_alloc_buf size %zd failed, no async space left\n",
			      alloc->pid, size);
		return ERR_PTR(-ENOSPC);
	}

	/* Pad 0-size buffers so they get assigned unique addresses */
	size = max(size, sizeof(void *));

	while (n) {
		buffer = rb_entry(n, struct binder_buffer, rb_node);
		BUG_ON(!buffer->free);
		buffer_size = binder_alloc_buffer_size(alloc, buffer);

		if (size < buffer_size) {
			best_fit = n;
			n = n->rb_left;
		} else if (size > buffer_size)
			n = n->rb_right;
		else {
			best_fit = n;
			break;
		}
	}
	if (best_fit == NULL) {
		size_t allocated_buffers = 0;
		size_t largest_alloc_size = 0;
		size_t total_alloc_size = 0;
		size_t free_buffers = 0;
		size_t largest_free_size = 0;
		size_t total_free_size = 0;

		for (n = rb_first(&alloc->allocated_buffers); n != NULL;
		     n = rb_next(n)) {
			buffer = rb_entry(n, struct binder_buffer, rb_node);
			buffer_size = binder_alloc_buffer_size(alloc, buffer);
			allocated_buffers++;
			total_alloc_size += buffer_size;
			if (buffer_size > largest_alloc_size)
				largest_alloc_size = buffer_size;
		}
........
}
```
```C++
// kernel/signal.c
int do_send_sig_info(int sig, struct siginfo *info, struct task_struct *p,
			bool group)
{
	unsigned long flags;
	int ret = -ESRCH;
+	if (start_rekernel_server() == 0) {
+ 		if (line_is_frozen(current) && (sig == SIGKILL || sig == SIGTERM || sig == SIGABRT || sig == SIGQUIT)) {
+     			char binder_kmsg[PACKET_SIZE];
+			snprintf(binder_kmsg, sizeof(binder_kmsg), "type=Signal,signal=%d,killer_pid=%d,killer=%d,dst_pid=%d,dst=%d;", sig, task_tgid_nr(p), task_uid(p).val, task_tgid_nr(current), task_uid(current).val);
+     			send_netlink_message(binder_kmsg, strlen(binder_kmsg));
+ 		}
+ 	}
	if (lock_task_sighand(p, &flags)) {
		ret = send_signal(sig, info, p, group);
		unlock_task_sighand(p, &flags);
	}

	return ret;
}
```
主要修改两个地方：

binder_transaction，通常位于 `drivers/android/binder.c`

binder_alloc_new_buf_locked，通常位于 `drivers/android/binder_alloc.c`

do_send_sig_info，通常位于 `kernel/signal.c`

改完之后重新编译内核，Re:Kernel将会融入你的内核当中。
