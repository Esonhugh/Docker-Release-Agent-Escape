> 为什么会以 kernel 的形式来 执行 release_agent 的命令呢? 

## [[usermodehelper]] program
在再次观摩了 [[reference#BlackHat 2019 Docker Escape]] 的视频后, 我发现了一个有趣的东西叫做 [[reference#Usermod helper]] 

> 用户模式小助手(x

用于帮助内核来执行用户态的程序 
> 这个过程和系统调用 syscall 相反

但本质可以看作是 call shell 去执行或者 call 应用程序去执行的 execve 函数 主要用于内核调用用户模式的 call

在 cgroup1 中 [cgroup1_release_agent](https://github.com/torvalds/linux/blob/24f6008564183aa120d07c03d9289519c2fe02af/kernel/cgroup/cgroup-v1.c#L756-L820) 是唯一一个调用 call_usermodehelper 的函数

这里可以看到 call usermod helper 的具体的执行位置, 分别限制了最基本的环境变量.

接着我们跟踪调用这个函数的调用者

简单的查询一下发现 [init_cgroup_housekeeping](https://github.com/torvalds/linux/blob/7e062cda7d90543ac8c7700fc7c5527d0c0f22ad/kernel/cgroup/cgroup.c#L1942-L1965) 函数调用了它. 从命名中不难看出 是一种类似于清理房屋整理功能安排调度的意思. 

## 如何布置 cgroup 的任务
entrypoint: https://github.com/torvalds/linux/blob/7e062cda7d90543ac8c7700fc7c5527d0c0f22ad/kernel/cgroup/cgroup.c#L1942-L1965

> 第二个是

而这里作为参数传入的 cgroup 结构体 跟踪其头文件我们可以看到通过 inlucde 进行定义的
>linux 内核将所有的文件包含放在了 项目根/include 文件夹下

```c
// file: include/linux/cgroup.h Line 28
#include <linux/cgroup-defs.h>
// file: include/linux/cgroup-defs.h Line 360
struct cgroup {
    /* self css with NULL ->ss, points back to this cgroup */
    struct cgroup_subsys_state self;
    unsigned long flags;        /* "unsigned long" so bitops work */
    /*
     * The depth this cgroup is at.  The root is at depth zero and each
     * step down the hierarchy increments the level.  This along with
     * ancestor_ids[] can determine whether a given cgroup is a
     * descendant of another without traversing the hierarchy.
     */
    int level;
    /* Maximum allowed descent tree depth */
    int max_depth;
    /*
     * Keep track of total numbers of visible and dying descent cgroups.
     * Dying cgroups are cgroups which were deleted by a user,
     * but are still existing because someone else is holding a reference.
     * max_descendants is a maximum allowed number of descent cgroups.
     *
     * nr_descendants and nr_dying_descendants are protected
     * by cgroup_mutex and css_set_lock. It's fine to read them holding
     * any of cgroup_mutex and css_set_lock; for writing both locks
     * should be held.
     */
    int nr_descendants;
    int nr_dying_descendants;
    int max_descendants;
    /*
     * Each non-empty css_set associated with this cgroup contributes
     * one to nr_populated_csets.  The counter is zero iff this cgroup
     * doesn't have any tasks.
     *
     * All children which have non-zero nr_populated_csets and/or
     * nr_populated_children of their own contribute one to either
     * nr_populated_domain_children or nr_populated_threaded_children
     * depending on their type.  Each counter is zero iff all cgroups
     * of the type in the subtree proper don't have any tasks.
     */
    int nr_populated_csets;
    int nr_populated_domain_children;
    int nr_populated_threaded_children;
    int nr_threaded_children;   /* # of live threaded child cgroups */
    struct kernfs_node *kn;     /* cgroup kernfs entry */
    struct cgroup_file procs_file;  /* handle for "cgroup.procs" */
    struct cgroup_file events_file; /* handle for "cgroup.events" */
    /*
     * The bitmask of subsystems enabled on the child cgroups.
     * ->subtree_control is the one configured through
     * "cgroup.subtree_control" while ->subtree_ss_mask is the effective
     * one which may have more subsystems enabled.  Controller knobs
     * are made available iff it's enabled in ->subtree_control.
     */
    u16 subtree_control;
    u16 subtree_ss_mask;
    u16 old_subtree_control;
    u16 old_subtree_ss_mask;
    /* Private pointers for each registered subsystem */
    struct cgroup_subsys_state __rcu *subsys[CGROUP_SUBSYS_COUNT];
    struct cgroup_root *root;
    /*
     * List of cgrp_cset_links pointing at css_sets with tasks in this
     * cgroup.  Protected by css_set_lock.
     */
    struct list_head cset_links;
    /*
     * On the default hierarchy, a css_set for a cgroup with some
     * susbsys disabled will point to css's which are associated with
     * the closest ancestor which has the subsys enabled.  The
     * following lists all css_sets which point to this cgroup's css
     * for the given subsystem.
     */
    struct list_head e_csets[CGROUP_SUBSYS_COUNT];
    /*
     * If !threaded, self.  If threaded, it points to the nearest
     * domain ancestor.  Inside a threaded subtree, cgroups are exempt
     * from process granularity and no-internal-task constraint.
     * Domain level resource consumptions which aren't tied to a
     * specific task are charged to the dom_cgrp.
     */
    struct cgroup *dom_cgrp;
    struct cgroup *old_dom_cgrp;        /* used while enabling threaded */
    /* per-cpu recursive resource statistics */
    struct cgroup_rstat_cpu __percpu *rstat_cpu;
    struct list_head rstat_css_list;
    /* cgroup basic resource statistics */
    struct cgroup_base_stat last_bstat;
    struct cgroup_base_stat bstat;
    struct prev_cputime prev_cputime;   /* for printing out cputime */
    /*
     * list of pidlists, up to two for each namespace (one for procs, one
     * for tasks); created on demand.
     */
    struct list_head pidlists;
    struct mutex pidlist_mutex;
    /* used to wait for offlining of csses */
    wait_queue_head_t offline_waitq;
    /* used to schedule release agent */
    struct work_struct release_agent_work;  // 这个struct 我们需要注意这里 有了 init work 的第一个参数
    /* used to track pressure stalls */
    struct psi_group psi;
    /* used to store eBPF programs */
    struct cgroup_bpf bpf;
    /* If there is block congestion on this cgroup. */
    atomic_t congestion_count;
    /* Used to store internal freezer state */
    struct cgroup_freezer_state freezer;
    /* ids of the ancestors at each level including self */
    u64 ancestor_ids[];
};
```

我们可以看到 work_struct 作为类型放在了 release_agent 前面

而 work_struct 通过 cgroup-defs.h 的第 21 行声明 在 `include/linux/workqueue.h` 中可以被找到

```c
struct work_struct {
	atomic_long_t data;
	struct list_head entry;
	work_func_t func;
#ifdef CONFIG_LOCKDEP
	struct lockdep_map lockdep_map;
#endif
};
```

定义了 work 的函数和数据 然后作为队列放入 workqueue 由内核去做

函数 INIT_WORK 因为其大写很容易猜测是一个 宏 

确实 我在 workqueue.h 中也找到了 他的定义

```c
#define INIT_WORK(_work, _func) \

__INIT_WORK((_work), (_func), 0)
```
是另一个宏的定义

```c
#ifdef CONFIG_LOCKDEP
#define __INIT_WORK(_work, _func, _onstack) \
do { \
static struct lock_class_key __key; \
\
__init_work((_work), _onstack); \
(_work)->data = (atomic_long_t) WORK_DATA_INIT(); \
lockdep_init_map(&(_work)->lockdep_map, "(work_completion)"#_work, &__key, 0); \
INIT_LIST_HEAD(&(_work)->entry); \
(_work)->func = (_func); \
} while (0)
#else
#define __INIT_WORK(_work, _func, _onstack) \
do { \
__init_work((_work), _onstack); \
(_work)->data = (atomic_long_t) WORK_DATA_INIT(); \
INIT_LIST_HEAD(&(_work)->entry); \
(_work)->func = (_func); \
} while (0)
#endif
```

就是个从工作队列开始不停执行 workqueue 到结束的循环

而且根据文件 `include/trace/events/cgroup.h` 

> 也就是在 `kernel/cgroup/cgroup.c` 64 行中被引入

```c
DEFINE_EVENT(cgroup, cgroup_release,
    TP_PROTO(struct cgroup *cgrp, const char *path),
    TP_ARGS(cgrp, path)
);
DEFINE_EVENT(cgroup, cgroup_rename,
    TP_PROTO(struct cgroup *cgrp, const char *path),
    TP_ARGS(cgrp, path)
);
DEFINE_EVENT(cgroup, cgroup_freeze,
    TP_PROTO(struct cgroup *cgrp, const char *path),
    TP_ARGS(cgrp, path)
);
```
这里不止 release 会有 release 事件产生

重命名和冻结等等也是可以有事件产生的

关于 notify_on_release 可以找到 cgroup-v1.c 中找到

```c
/* cgroup core interface files for the legacy hierarchies */
struct cftype cgroup1_base_files[] = {
	{
		.name = "cgroup.procs",
		.seq_start = cgroup_pidlist_start,
		.seq_next = cgroup_pidlist_next,
		.seq_stop = cgroup_pidlist_stop,
		.seq_show = cgroup_pidlist_show,
		.private = CGROUP_FILE_PROCS,
		.write = cgroup1_procs_write,
	},
	{
		.name = "cgroup.clone_children",
		.read_u64 = cgroup_clone_children_read,
		.write_u64 = cgroup_clone_children_write,
	},
	{
		.name = "cgroup.sane_behavior",
		.flags = CFTYPE_ONLY_ON_ROOT,
		.seq_show = cgroup_sane_behavior_show,
	},
	{
		.name = "tasks",
		.seq_start = cgroup_pidlist_start,
		.seq_next = cgroup_pidlist_next,
		.seq_stop = cgroup_pidlist_stop,
		.seq_show = cgroup_pidlist_show,
		.private = CGROUP_FILE_TASKS,
		.write = cgroup1_tasks_write,
	},
	{
		.name = "notify_on_release",
		.read_u64 = cgroup_read_notify_on_release,
		.write_u64 = cgroup_write_notify_on_release,
	},
	{
		.name = "release_agent",
		.flags = CFTYPE_ONLY_ON_ROOT,
		.seq_show = cgroup_release_agent_show,
		.write = cgroup_release_agent_write,
		.max_write_len = PATH_MAX - 1,
	},
	{ }	/* terminate */
};
```

这里定义了几个配置文件

了解了这些基础信息之后, **我们核心要关注的两个文件是 kernel/cgroup/cgroup.c 和 kernel/cgroup/cgroup-v1.c**

这两个定义了 cgroup 的核心处理流程 综合其他的诸如配置信息 进行阅读可以发现 [文件 `kernel/cgroup/cgroup-v1.c` 的 751 行 ](https://cs.github.com/torvalds/linux/blob/7e062cda7d90543ac8c7700fc7c5527d0c0f22ad/kernel/cgroup/cgroup-v1.c#L751)有如下定义

```c
void cgroup1_check_for_release(struct cgroup *cgrp)
{
	if (notify_on_release(cgrp) && !cgroup_is_populated(cgrp) &&
	    !css_has_online_children(&cgrp->self) && !cgroup_is_dead(cgrp))
		schedule_work(&cgrp->release_agent_work);
}
```

这个函数就是为什么当 notify 设为 1 会将 release_agent  的任务塞入最后的原因

现在所有的逻辑链全部被串通了.

在布置 cgroup subsystem 的时候 系统会对每个任务进行布置 依次塞入 workqueue 队列中, 当 notfiy 被设置时会调度入 release_agent_work 结构体进入其队列当中, 当所有任务执行完成 或者说产生了任务完成的 Event 这样这个结构体会被传入然后执行 call_usermodehelper 

同样通过查询 `->release_agent_work` 这个我们可以看到 引用这个 work 或者执行这个 work 的只有 那两个我们需要核心关注的文件.

### 一点点猜想
但是我想至于为什么只有 release_agent 可以我想是只有 release 这个特殊的事件具有 这种 call_usermodehelper 并且用配置文件 release_agent 进行配置

refer: [[cgroup]]