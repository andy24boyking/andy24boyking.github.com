---
layout: post
title: linux内核数据结构:链表
category: programming
tags: [linux, linked list]
---
近期由于实验室项目需要，学习了一下 linux 内核中的链表数据结构的使用和实现。链表是一种常用的组织数据的数据结构，学过数据结构的人都会非常熟悉。它是线性表的一种常见的实现方式，优点在于具有较好的插入和删除效率，而且建立链表时无需知道数据总量，也不需要大量连续的分配空间。但是链表的顺序性不好，而且为了组织链表除了存储数据外还需要一些额外的内存空间来存储指针。

常见的链表数据结构至少包含数据域和指针域，按指针域的组织形式可分为单链表、双链表、循环链表等。由于这不是本文的重点，不在此赘述。

## Linux内核链表结构

通常一个单链表结点的数据结构定义往往如下：

{% assign lang = 'c' %}
{% capture codeblock %}struct list_node {
    ElemType data;
    struct list_node *next;
};
{% endcapture %}
{% include AH/print_code %}

这样是在链表结点中嵌入数据元素，因此对于每一种 ElemType 的数据都要定义新的链表结构。而 Linux 为了方便和通用，采用了将链表结点嵌入到元素结构体的做法，这样一定程度上实现了数据和链表结构的松耦合。

<!-- excerpt -->

Linux 内核链表是一个双向循环链表结构，结构体定义如下：

{% assign lang = 'c' %}
{% capture codeblock %}typedef struct list_head {
    struct list_head *prev, *next;
} list_t;
{% endcapture %}
{% include AH/print_code %}

### 链表初始化

循环链表的初始化即将前、后指针都指向自己，分为静态和动态两种：

{% assign lang = 'c' %}
{% capture codeblock %}#define LIST_HEAD_INIT(name) {&(name), &(name)}
/*静态初始化链表，声明时*/
#define LIST_HEAD(name) \
    struct list_head name = LIST_HEAD_INIT(name)

/*动态初始化链表*/
static inline void INIT_LIST_HEAD(struct list_head *list)
{
    list->next = list;
    list->prev = list;
}
{% endcapture %}
{% include AH/print_code %}

基于上述初始化链表的方法，可以根据头结点的 next 指针是否指向自己来判断一个链表是否为空：

{% assign lang = 'c' %}
{% capture codeblock %}static inline int list_empty(const struct list_head *head)
{
    return head->next == head;
}
{% endcapture %}
{% include AH/print_code %}

### 插入操作

Linux 提供了在表头和表尾插入结点两种方式，由于是双向循环链表，均可通过调用内置函数 `__list_add` 来方便地实现：

{% assign lang = 'c' %}
{% capture codeblock %}/*
* 插入结点
*/
static inline void __list_add(struct list_head *new,
    struct list_head *prev,
    struct list_head *next)
{
    next->prev = new;
    new->next = next;
    new->prev = prev;
    prev->next = new;
}
static inline void list_add(struct list_head *new, struct list_head *head)
{
    __list_add(new, head, head->next);
}
static inline void list_add_tail(struct list_head *new, struct list_head *head)
{
    __list_add(new, head->prev, head);
}
{% endcapture %}
{% include AH/print_code %}

### 删除操作

删除结点，准确的说是将结点从链表中脱离的实现如下：

{% assign lang = 'c' %}
{% capture codeblock %}/*
* 删除结点
*/
static inline void __list_del(struct list_head *prev, struct list_head *next)
{
    prev->next = next;
    next->prev = prev;
}
static inline void list_del(struct list_head *entry)
{
    __list_del(entry->prev, entry->next);
    entry->prev = NULL; /*entry->next = LIST_POISON1;*/
    entry->next = NULL; /*entry->prev = LIST_POISON2;*/
}
{% endcapture %}
{% include AH/print_code %}

原本的实现中，是把从链表中移除出来的结点的前后指针指向两个特殊的值， LIST_POISON1 和 LIST_POISON2 。

{% assign lang = 'c' %}
{% capture codeblock %}#define LIST_POISON1 ((void *) 0x00100100)
#define LIST_POISON2 ((void *) 0x00200200)
{% endcapture %}
{% include AH/print_code %}

Linux 内核中解释：These are non-NULL pointers that will result in page faults under normal circumstances, used to verify that nobody uses non-initialized list entries. 当用户打算访问这两个地址时将会产生页故障。我在自己的代码中实现时改为了简单的赋空值，再调用 free 函数来释放该结点。

Linux 提供的删除操作还有 `list_del_init` ，即将指定的结点从链表中移除同时再以该结点为头结点初始化一个新的循环链表。


{% assign lang = 'c' %}
{% capture codeblock %}static inline void list_del_init(struct list_head *entry)
{
    __list_del(entry->prev, entry->next);
    INIT_LIST_HEAD(entry);
}
{% endcapture %}
{% include AH/print_code %}

### 移动结点操作

Linux 提供了两个移动结点的操作 `list_move` 和 `list_move_tail`，功能是把指定结点移动到另一个链表的表头或者表尾，其实就是删除和添加的组合。

{% assign lang = 'c' %}
{% capture codeblock %}static inline void list_move(struct list_head *list,
    struct list_head *head)
{
    __list_del(list->prev, list->next);
    list_add(list, head);
}

static inline void list_move_tail(struct list_head *list,
    struct list_head *head)
{
    __list_del(list->prev, list->next);
    list_add_tail(list, head);
}
{% endcapture %}
{% include AH/print_code %}

### 链表合并操作

Linux 提供了合并两个链表的操作 `list_sllice` 和 `list_splice_init` 。

{% assign lang = 'c' %}
{% capture codeblock %}static inline void __list_splice(struct list_head *list, 
    struct list_head *head)
{
    struct list_head *first = list->next;
    struct list_head *last = list->prev;
    struct list_head *at = head->next;
    
    first->prev = head;
    head->next = first;
    
    last->next = at;
    at->prev = last;
}

static inline void list_splice(struct list_head *list,
    struct list_head *head)
{
    if (!list_empty(list))
        __list_splice(list, head);
}

static inline void list_splice_init(struct list_head *list,
    struct list_head *head)
{
    if (!list_empty(list)){
        __list_splice(list, head);
        INIT_LIST_HEAD(list);
    }
}
{% endcapture %}
{% include AH/print_code %}

该操作实现了将一个非空的链表 list 插入到另一个链表的 head 处。可以形象的理解成将两个环(循环链表)拆开，然后分别首首相接、尾尾相接，合成一个新的环(循环链表)。因此对 list 有一个判空的步骤。而合并后原链表中的 list 结点已经被弃用， `list_splice_init` 则在合并之后将 list 初始化为一个空链表头。

### 返回上层结构操作 <i class="icon-star"></i>

这是 Linux 链表结构最关键也是实现的最巧妙的地方。前面已经提过，Linux 内核的链表结构是将链表结点嵌入到元素结构体中。一个已经建立好的链表可以用下图来简单表示：

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-05-21-list1.png)
Linux 内核链表结构示意图

通过 list_head 结构构成的循环链表可以遍历到每一个结点，如何取到该结点处宿主结构里的数据成员，则需要返回到上层的宿主结构体中，也就是本节的内容。

{% assign lang = 'c' %}
{% capture codeblock %}typedef struct Node {
    struct data_area data;
    struct list_head head;
} Node_st;
{% endcapture %}
{% include AH/print_code %}

假设链表示意图中的结点结构体定义如上，我们需要遍历到某个特定结点后，从 struct list_head 结构体返回到 Node_st 结构体。Linux 提供了如下几个宏来实现这一操作：

{% assign lang = 'c' %}
{% capture codeblock %}#define list_entry(ptr, type, member) container_of(ptr, type, member)

#define container_of(ptr, type, member) ({ \
        const typeof( ((type *)0)->member ) *__mptr = (ptr); \
        (type *)( (char *)__mptr - offsetof(type,member) );})

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
{% endcapture %}
{% include AH/print_code %}

使用起来很方便，假设 `struct list_head *list` 是指向当前结点中 head 结构的指针，则  如下代码即可得到指向宿主结点的指针 p :

{% assign lang = 'c' %}
{% capture codeblock %}Node_st *p;
p = list_entry(list, Node_st, head);
{% endcapture %}
{% include AH/print_code %}

原理稍微有点复杂，使用的是一个利用编译器技术的小技巧，先求得已知结构成员(list_head)在宿主结构体(Node_st)中的偏移量，再根据已知结构成员的地址和偏移量来求得宿主结构体的地址。
主要就是靠 `container_of()` 和 `offsetof()` 这两个宏定义来实现(这是两个 Linux 内核的通用的宏定义，不仅仅用在了链表操作中)。

先来看 offsetof 宏：

- `(TYPE *)0` 将地址 0 强制转换成 TYPE 结构类型的指针；
- `(TYPE *)0)->MEMBER` 访问该结构体中的 MEMBER 成员；
- `&((TYPE *)0)->MEMBER)` 取得 MEMBER 成员的地址；
- `((size_t) &((TYPE *)0)->MEMBER)` 将取得的地址值转换为 size_t 类型(size_t 是标准 C 库中定义的，应为 unsigned int，在64位系统中为 long unsigned int)。

技巧就在于，该结构体的地址是我们设定的 0 。因此最终取得的 MEMBER 地址值，即为 MEMBER 成员的地址相对于所属结构体地址的偏移量。

再来看 container_of 宏：

- typeof 是 gcc 的扩展，可以得到所给变量的类型。因此 `typeof( ((type *)0)->member )` 为获取 member 成员的变量类型。
- `const typeof( ((type *)0)->member ) *__mptr = (ptr);` 其中 ptr 为指向 member 成员变量的指针。这条语句即为用获取的 member 成员的变量类型来定义一个 const 指针 *__mptr 指向 ptr 所指内存处。
- `(char *)__mptr` 将 __mptr 转为字节类型指针， `(char *)__mptr - offsetof(type,member)` 求得 type 类型的结构体的起始处地址，此时为 char 型指针。
- `(type *)( (char *)__mptr - offsetof(type,member) );` 再通过 (type *)强制类型转换，从而得到了一个指向 type 类型结构体的指针。

list_entry 宏就是对 container_of 宏的一个简单封装，实现了我们的目的：通过已知的结构体类型指针，求得了指向它的宿主结构体的指针。下图为上述分析的图示，p指针即为最终所求。

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-05-21-container-of.png)
container_of 宏示意图

### 遍历链表操作

遍历是链表的基本操作，Linux 提供了一组设定好 for 循环条件的宏来提供遍历操作。

{% assign lang = 'c' %}
{% capture codeblock %}/*
* 遍历结点
*/
#define __list_for_each(pos, head) \
    for (pos = (head)->next; pos != (head); pos = pos->next)

#define list_for_each(pos, head) \
    for (pos = (head)->next; pos != (head); pos = pos->next)

#define list_for_each_safe(pos, n, head) \
    for (pos = (head)->next, n = pos->next; pos != head; pos = n, n = pos->next)

#define list_for_each_entry(pos, head, member) \
    for (pos = list_entry((head)->next, typeof(*pos), member); \
        &(pos->member) != (head); \
        pos = list_entry((pos->member).next, typeof(*pos), member))
{% endcapture %}
{% include AH/print_code %}

list_for_each 提供了基本的链表遍历，pos 是指向 list_head 结构的指针；list_for_each_entry 遍历链表并取得每一个链表结点的宿主结构体，这里 pos 是待赋值的指向宿主结构体的指针，member 是 list_head 结构在宿主结构中的成员名。

list_for_each_safe，顾名思义，提供了安全的链表遍历。为什么说是安全的呢？在 for 循环中 n 暂存了 pos 的下一个结点的地址，从而避免因为 pos 节点被释放而造成链表断掉。因此遍历完当前结点后将其删除也可以。这个遍历方式是很有用的，例如有多个进程等待在同一个等待队列上，当事件发生时唤醒所有进程后，可以遍历这些进程，将它们依次从等待的队列中删除。

其实链表遍历相关的宏定义还有很多，例如 list_for_each_prev 、list_for_each_entry_continue 、list_for_each_entry_reverse 、list_for_each_entry_safe 、list_for_each_entry_safe_continue 等，都有其特定的用途，可以查看相关源码了解。

另外很多介绍 Linux 内核链表的文章会提到遍历时的 prefetch 预取操作来提高遍历速度。实际上有牛人证明了执行 prefetch 所消耗的时间超出了它缓存带来的好处，实际上反而影响了效率，因此从 3.0 内核开始 prefetch 操作已经从链表，哈希表以及 sk_buff 表的遍历操作中移除了([详情点这里](http://lwn.net/Articles/444336/))。

## 应用

在实验室最近的项目中我需要编写一个配置管理的守护程序，基本工作流程就是：启动时读取各个配置文件建立起各个配置项的链表，web 前端需要提供配置的显示、修改、删除等功能，因此需要和 配置管理程序通信；同时还需要将各个配置传入相应功能模块中使之生效。因此提供给功能模块和web前端的通信接口应该只有数据块的交互，链表的增删改查都是配置管理程序内部的事情，Linux 内核链表结构的特点就很符合我的需求。

在实际使用中，可以根据自己的需要来定义和实现这种嵌入的链表结构。例如我自己定义了一个 mylist.h 头文件，里面只定义了我需要的链表操作的宏，list_head 结构也简化为了单链表(配置项数据量不大)，list_entry 的宏定义也直接合在一个表达式里给出。

(**注**：由于 3.0 以后的内核版本中 有关 Linux 链表数据结构的相关定义和声明所在的头文件和之前有了很大的差别，暂时还没有整理出来，所以文中提到的宏定义、结构体定义、内联函数均未说明在哪个头文件中。)