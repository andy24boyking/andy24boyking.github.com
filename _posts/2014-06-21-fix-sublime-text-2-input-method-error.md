---
layout: post
title: 解决Linux下Sublime Text 2中文输入法问题
category: programming
tags: [linux, gtk, sublime]
---
原文地址：[http://my.oschina.net/wugaoxing/blog/121281](http://my.oschina.net/wugaoxing/blog/121281)

和原文作者的情形类似，我之前在 ubuntu12.04 下安装了fcitx输入框架下的谷歌拼音输入法，勉强解决了在 Sublime Text 2 的中文输入问题。之所以说勉强，是因为输入中文时若拼音打错时按退格键，删除的不是输入法上的字母，而是 Sublime 文本编辑区上的文字，而且没办法中英文混合输入。虽说有这么多问题，不过至少还能用，就没有去想怎么解决。可是系统升级到 ubuntu 14.04之后，突然发现在 Sublime Text 2 下无法切换到fcitx谷歌拼音输入法，因此不能输入中文了。还好看到了上面贴的这篇博文，找到了解决方法。大概原理就是利用 **LD_PRELOAD** 环境变量在 Sublime Text 2 运行前优先加载我们写好的动态链接库来修复  Sublime Text 2 的中文输入法支持问题。

<!-- excerpt -->

首先需要编译一个共享库，源代码 **sublime_imfix.c** 如下:

{% assign lang = 'c' %}
{% capture codeblock %}/*
sublime-imfix.c
Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
By Cjacker Huang <jianzhong.huang at i-soft.com.cn>

gcc -shared -o libsublime-imfix.so sublime_imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC
LD_PRELOAD=./libsublime-imfix.so sublime_text
*/
#include <gtk/gtk.h>
#include <gdk/gdkx.h>
typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
	long size;
	long numRects;
	GdkRegionBox *rects;
	GdkRegionBox extents;
};

GtkIMContext *local_context;

void gdk_region_get_clipbox(const GdkRegion *region, GdkRectangle *rectangle)
{
	g_return_if_fail(region != NULL);
	g_return_if_fail(rectangle != NULL);

	rectangle->x = region->extents.x1;
	rectangle->y = region->extents.y1;
	rectangle->width = region->extents.x2 - region->extents.x1;
	rectangle->height = region->extents.y2 - region->extents.y1;
	GdkRectangle rect;
	rect.x = rectangle->x;
	rect.y = rectangle->y;
	rect.width = 0;
	rect.height = rectangle->height; 
	//The caret width is 2; 
	//Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
	if (rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
		gtk_im_context_set_cursor_location(local_context, rectangle);
	}
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter(GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
	XEvent *xev = (XEvent *)xevent;

	if (xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
		GdkWindow * win = g_object_get_data(G_OBJECT(im_context),"window");
		if (GDK_IS_WINDOW(win))
			gtk_im_context_set_client_window(im_context, win);
	}

	return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window(GtkIMContext *context, GdkWindow *window)
{
	GtkIMContextClass *klass;

	g_return_if_fail(GTK_IS_IM_CONTEXT(context));
	klass = GTK_IM_CONTEXT_GET_CLASS (context);
	if (klass->set_client_window)
		klass->set_client_window(context, window);

	if (!GDK_IS_WINDOW(window))
		return;
	g_object_set_data(G_OBJECT(context),"window",window);

	int width = gdk_window_get_width(window);
	int height = gdk_window_get_height(window);
	
	if (width != 0 && height !=0) {
		gtk_im_context_focus_in(context);
		local_context = context;
	}
	gdk_window_add_filter(window, event_filter, context); 
}
{% endcapture %}
{% include AH/print_code %}

编译这个源代码需要 gtk 开发库的支持，安装命令如下：

    sudo apt-get install libgtk2.0-dev

编译共享库的命令如下：

    gcc -shared -o libsublime-imfix.so sublime_imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC

OK，此时就可以看见当前目录下生成了共享库 **libsublime-imfix.so** 文件。想试验下效果，可执行如下命令启动 Sublime Text 2 ：

    LD_PRELOAD=./libsublime-imfix.so subl

此时就可以在 Sublime Text 2 编辑器中正常输入中文了，之前 12.04 下的问题也得到了解决。为了在图形化界面启动 Sublime Text 2 时也能自动加载该共享库，我们还需要修改 **/usr/share/applications/** 路径下 Sublime Text 2 的启动文件 **sublime-text-2.desktop**。首先将动态库 **libsublime-imfix.so** 拷贝到 **/usr/lib/**下，然后将 **sublime-text-2.desktop** 中的进程启动行（Exec=）都加上上面定义的 LD_PRELOAD 就行了。以我自己的为例，将如下三行：

    Exec=bash -c /usr/bin/subl %F
    Exec=bash -c /usr/bin/subl -n
    Exec=bash -c /usr/bin/subl --command new_file

替换为：

    Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' %F
    Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' -n
    Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' --command new_file

即可。另外我电脑上 Sublime Text 2 的启动文件（类似于windows下的快捷方式）是 **subl**(/usr/bin/subl)，大家安装的 Sublime Text 2 也有可能是 **sublime—text** 或者 **sublime-text-2** ，需要相应的调整上面的命令。附设置成功后输入中文的截图如下：

{:.post-image}
![image]({{BASE_PATH}}/assets/posts/images/2014-06-21-input-method.png)

此方法由 sublime 论坛上的 [cjacker 提供](http://www.sublimetext.com/forum/viewtopic.php?f=3&t=7006&start=10#p41343)。
