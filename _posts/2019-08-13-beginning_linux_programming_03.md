---
layout:     post
title:      Linux程序设计 学习笔记 (三)
subtitle:   Linux程序设计 学习笔记 (三)
date:       2019-09-03
author:     王鹏程
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - C/C++
    - Linux
    - 操作系统
    - 程序设计
---

# Linux 程序设计 阅读笔记(三)

## 参考链接：

- [Linux内核文档首页](https://www.kernel.org/doc/Documentation/)
- [Linux文档](https://linux.die.net/)
- [Linux c 开发手册](https://legacy.gitbook.com/book/wizardforcel/linux-c-api-ref/details)
- [Linux Kernel API](https://www.kernel.org/doc/htmldocs/kernel-api/index.html)
- [书中代码地址](http://www.wrox.com/WileyCDA/WroxTitle/Beginning-Linux-Programming-4th-Edition.productCd-0470147628,descCd-DOWNLOAD.html)

## 第6章 使用curses函数库管理基于文本的屏幕

### 6.1 使用curses函数库进行编译

添加头文件`-I/usr/include/nucurses`,添加动态链接库`-lncurses`。

curses中输出字符的过程如下：

- 使用curses函数刷新逻辑屏幕。
- 要求curses使用refresh函数来刷新物理屏幕。

逻辑屏幕的布局通过一个字符数组来实现，它以屏幕的左上角为坐标原点。一般坐标表示(行，列)，y值在前、x值(列号)在后。

![相关函数](https://wangpengcheng.github.io/img/2019-09-04-15-27-49.png)

一个简单示例：

```c++

#include <unistd.h>
#include <stdlib.h>
#include <curses.h>

int main()
{
    //初始化屏幕

    initscr();
    //移动画笔位置
    move(5,15);

    printw("%s","Hello word");
    //刷新屏幕

    refresh();
    sleep(2);
    //结束窗口

    endwin();
    exit(EXIT_SUCCESS);
}
```
### 6.3 屏幕

使用`WINDOW *initscr(void)`进行窗口的初始化。这个函数在一个程序中只能调用一次。当结束窗口时就是使用`endwin()`函数进行相关资源的销毁，当函数成功时返回OK否则返回ERR。

#### 6.3.1 输出到屏幕

curses函数库提供了一些用于刷新屏幕的基本函数，它们是：


- `int addch(const chtype char_to_add)`:。
- `int addchstr(chtype  *const string_to_add)`:。
- `int printw(char* format,...)`:
- `int refresh(void)`
- `int box(WINDOW *win_ptr,chtype vertical_char,chtype horizontal_char)`
- `int insch(chtype char_to_insert)`：插入一个字符，将已有字符向右移动
- `int insertln(void)`
- `int delch(void)`
- `int delectln(void)`
- `int beep(void)`
- `int flash(void)`
- `int erase(void)`:清除屏幕，在每个屏幕位置写上空白字符。
- `int clear(void)`:调用erase后，再使用clearok来强制重现屏幕原文。彻底清除整个屏幕。
- `int clrtobot`函数清除当前光标位置直到屏幕结尾的所有内容。
- `int clrtobot`函数清除当前光标位置直到光标所处行位。
- `int move(int new_y,int new_x)`:将逻辑光标的位置移动到指定地点。希望变化立即显现使用`refresh`函数。
- `int leaveok(WINDOW *window_ptr,bool leave_flag)`；添加bool标志位，控制在屏幕刷新后curses刷新物理光标所放的位置。默认false，刷新后，硬件光标停留在屏幕上逻辑光标所处的位置。true,硬件光标会被随机地防止在屏幕的任意位置之上。

注意：curses拥有自己的字符类型chtype，比标准char类型包含跟多的二进制。

#### 6.3.5 字符属性

- `int attron(chtype attribute)`：启用指定的属性
- `int attroff(chtype attribute)`：关闭指定的属性
- `int attrset(chtype attribute)`:设置curses属性。
- `int standout(void)`:标准的输出
- `int standend(void)`:标准输出

### 6.4 键盘

设置键盘的相关函数

- `int echo(void)`：输出。
- `int noecho(void)`：。
- `int cbreak(void)`:
- `int raw(void)`:
- `int noraw(void)`:

#### 6.4.2 键盘输入

`int getch(void)`、`int getstr(char *string)`、`int getnstr(char* string,int number_of_characters)`、`int scanw(char *format,...)`。

### 6.5 窗口

#### 6.5.1 WINDOW窗口

使用`WINDOW * newwin(int num_of_lines,int num_of_cols,int start_y,int start_x)`来出那个键一个新窗口。`int delwin(WINDOW *window_to_delete)`删除一个窗口

注意：千万不要尝试删除curses自己的窗口和stdscr和curscr。

#### 6.5.3 移动和更新窗口

```c++
#include <curses.h>
int mvwin(WINDOW *window_to_move,int new_y,int new_x);
int wrefresh(WINDOW *window_ptr);
int wclear(WINDOW *window_ptr)
int werase(WINDOW *window_ptr);
int touchwin(WINDOW *window_ptr);
int scrollok(WINDOW *window_ptr,bool scroll_flag);
int scroll(WINDOW *window_ptr);
```

使用示例：

```c++
/*  As usual let's get our definitions sorted first.  */

#include <unistd.h>
#include <stdlib.h>
#include <curses.h>

int main()
{
    WINDOW *new_window_ptr;
    WINDOW *popup_window_ptr;
    int x_loop;
    int y_loop;
    char a_letter = 'a';
    //初始化显示

    initscr();

/*  Then we fill the base window with characters,
    refreshing the actual screen once the logical screen has been filled:

    move(5, 5);
    printw("%s", "Testing multiple windows");
    refresh();

    for (x_loop = 0; x_loop < COLS - 1; x_loop++) {
        for (y_loop = 0; y_loop < LINES - 1; y_loop++) {
            mvwaddch(stdscr, y_loop, x_loop, a_letter);
            a_letter++;
            if (a_letter > 'z') a_letter = 'a';
        }
    }

    refresh();
    sleep(2);
*/
/*  Now we create a new 10x20 window
    and add some text to it before drawing it on the screen.  */

    //创建新窗口

    new_window_ptr = newwin(10, 20, 5, 5);
    //移动窗口并进行写操作
    mvwprintw(new_window_ptr, 2, 2, "%s", "Hello World");

    mvwprintw(new_window_ptr, 5, 2, "%s", "Notice how very long lines wrap inside the window");
    //刷新窗口

    wrefresh(new_window_ptr);
    sleep(2);

/*  We now change the contents of the background window and, when we
refresh the screen, the window pointed to by new_window_ptr is obscured.  */

   a_letter = '0';
     for (x_loop = 0; x_loop < COLS - 1; x_loop++) {
        for (y_loop = 0; y_loop < LINES -1; y_loop++) {
            mvwaddch(stdscr, y_loop, x_loop, a_letter);
            a_letter++;
            if (a_letter > '9') a_letter = '0';
        }
    }

    refresh();
    sleep(2);

/*  If we make a call to refresh the new window, nothing will change,
    because we haven't changed the new window.  */

    wrefresh(new_window_ptr);
    sleep(2);

/*  But if we touch the window first
    and trick curses into thinking that the window has been changed.
    The next call to wrefresh will bring the new window to the front again.  */

    touchwin(new_window_ptr);
    wrefresh(new_window_ptr);
    sleep(2);

/*  Next, we add another overlapping window with a box around it.  */

    popup_window_ptr = newwin(10, 20, 8, 8);
    box(popup_window_ptr, '|', '-');
    mvwprintw(popup_window_ptr, 5, 2, "%s", "Pop Up Window!");
    wrefresh(popup_window_ptr);
    sleep(2);

/*  Then we fiddle with the new and pop-up windows before clearing and deleting them.  */

    touchwin(new_window_ptr);
    wrefresh(new_window_ptr);
    sleep(2);

    wclear(new_window_ptr);
    wrefresh(new_window_ptr);
    sleep(2);
    
    delwin(new_window_ptr);

    touchwin(popup_window_ptr);
    wrefresh(popup_window_ptr);
    sleep(2);
    
    delwin(popup_window_ptr);
    
    touchwin(stdscr);
    refresh();
    sleep(2);

    endwin();
    exit(EXIT_SUCCESS);
}

```

#### 6.5.4 优化屏幕刷新

- `int wnoutrefresh(WINDOW *window_ptr)`：把那些字符发送到屏幕上，实际的发送工作由dpupdate完成
- `int doupdate(void)`：可以进行相关数据的更新工作。
- `WINDOW *subwin(WINDOW *parent,int num_of_lines,int num_of_cols,int start_y,int start_x)`：创建子窗口

#### 6.4 keypad模式

使用`int keypad(WINDOW *window_ptr,bool keypad_on)`设置keypad_on为true来启用keypad模式。让curses接管按键转义序列的处理工作。读取用户按下的键，还将返回与逻辑按键对应的KEY_定义。

注意：识别escape转义序列的过程是与时间相关的，为了能够区分单独按下Escape按键和一个以Escape字符开头的键盘转义序列。不能处理二义性的escape转义序列。如果终端上两个而不同的按键会产生完全相同的专业序列，curses将不会处理这个转义序列，因为它不知道该返回哪个逻辑按键。

下面是简单的使用示例：

```c
#include <unistd.h>
#include <stdlib.h>
#include <curses.h>

#define LOCAL_ESCAPE_KEY    27

int main() 
{
    int key;

    initscr();
    crmode();
    keypad(stdscr, TRUE);

/*  Next, we must turn echo off
    to prevent the cursor being moved when some cursor keys are pressed.
    The screen is cleared and some text displayed.
    The program waits for each key stroke
    and, unless it's q, or produces an error, the key is printed.
    If the key strokes match one of the terminal's keypad sequences, 
    then that sequence is printed instead.  */
    //截断输出

    noecho();
    //清除

    clear();
    //移动输出

    mvprintw(5, 5, "Key pad demonstration. Press 'q' to quit");
    move(7, 5);
    //刷新

    refresh();
    //获取字符

    key = getch();
    while(key != ERR && key != 'q') {
        move(7, 5);
        clrtoeol();

        if ((key >= 'A' && key <= 'Z') ||
            (key >= 'a' && key <= 'z')) {
            printw("Key was %c", (char)key);
        }else {
            switch(key) {
            case LOCAL_ESCAPE_KEY: printw("%s", "Escape key"); break;
            case KEY_END: printw("%s", "END key"); break;
            case KEY_BEG: printw("%s", "BEGINNING key"); break;
            case KEY_RIGHT: printw("%s", "RIGHT key"); break;
            case KEY_LEFT: printw("%s", "LEFT key"); break;
            case KEY_UP: printw("%s", "UP key"); break;
            case KEY_DOWN: printw("%s", "DOWN key"); break;
            default: printw("Unmatched - %d", key); break;
            } /* switch */
        } /* else */

        refresh();
        key = getch();
    } /* end while */

    endwin();
    exit(EXIT_SUCCESS);
}

```

### 6.8 彩色显示

使用has_color()和start_color()可以实现对颜色例程的初始化。

```c++
#include <curses.h>
bool has_colors(void);
int start_color(void);

```
在将颜色作为属性使用之前，你必须首先调用`init_pair()`函数对准备使用的颜色组合进行初始化。对颜色属性的访问是通过COLOR_PAIR函数来完成的。

```c
#include <curses.h>

//定义颜色组合

int init_pair(short pair_number.short foreground,short background);
//

int COLOR_PAIT(int pair_number);
//获取已有的颜色组合

int pair_content(short pair_number,short *foreground,short *background);
//重定义色彩

int init_color(short color_number,short red,short green,short blue); 
```

颜色的调用示例：

```c

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <curses.h>

int main()
{
    int i;

    //初始化curses

    initscr();
    //检查是支持色彩的终端

    if (!has_colors()) {
        endwin();
        fprintf(stderr, "Error - no color support on this terminal\n");
        exit(1);
    }
    //初始化色彩选项

    if (start_color() != OK) {
        endwin();
        fprintf(stderr, "Error - could not initialize colors\n");
        exit(2);
    }

/*  We can now print out the allowed number of colors and color pairs.
    We create seven color pairs and display them one at a time.  */
    //清除屏幕

    clear();
    //输出文字

    mvprintw(5, 5, "There are %d COLORS, and %d COLOR_PAIRS available", 
             COLORS, COLOR_PAIRS);
    //进行刷新

    refresh();
    //初始化颜色

    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_RED, COLOR_GREEN);
    init_pair(3, COLOR_GREEN, COLOR_RED);
    init_pair(4, COLOR_YELLOW, COLOR_BLUE);
    init_pair(5, COLOR_BLACK, COLOR_WHITE);
    init_pair(6, COLOR_MAGENTA, COLOR_BLUE);
    init_pair(7, COLOR_CYAN, COLOR_WHITE);
    //输出显色字符串

    for (i = 1; i <= 7; i++) {
        //开启属性设置

        attroff(A_BOLD);
        //设置颜色属性

        attrset(COLOR_PAIR(i));
        //输出文字

        mvprintw(5 + i, 5, "Color pair %d", i);
        //设置属性

        attrset(COLOR_PAIR(i) | A_BOLD);
        //输出文字

        mvprintw(5 + i, 25, "Bold color pair %d", i);
        //刷新

        refresh();
        //睡眠

        sleep(1);
    }

    endwin();
    exit(EXIT_SUCCESS);
}

```

### 6.9 pad
curses提供了特殊的数据结构pad来控制尺寸大于正常窗口的逻辑屏幕。相关接口如下;

```c
//初始化pad

WINDOW *newpad(int number_of_lines,int number_of_columns);
//执行刷新操作。指定放到屏幕上的pad范围和放置在屏幕上的位置。prefresh函数用于完成这一功能。


int prefresh(WINDOW *pad_ptr,
            int pad_row,
            int pad_column,
            int screen_row_min,//显示区域的坐标范文
            int screen_col_min,
            int screen_row_max,
            int screen_col_max
            )
```
pad的简单示例：

```c
#include <unistd.h>
#include <stdlib.h>
#include <curses.h>

int main() 
{
    WINDOW *pad_ptr;
    int x, y;
    int pad_lines;
    int pad_cols;
    char disp_char;

    initscr();

    pad_lines = LINES + 50;
    pad_cols = COLS + 50;
    //创建pad

    pad_ptr = newpad(pad_lines, pad_cols);

    disp_char = 'a';
    //在内部添加字符串

    for (x = 0; x < pad_lines; x++) {
        for (y = 0; y < pad_cols; y++) {
            mvwaddch(pad_ptr, x, y, disp_char);
            if (disp_char == 'z') disp_char = 'a';
            else disp_char++;
        }
    }

/*  We can now draw different areas of the pad on the screen at different locations before quitting.  */
    //更新局部窗口
    
    prefresh(pad_ptr, 5, 7, 2, 2, 9, 9);
    sleep(1);
    //更新大范围窗口

    prefresh(pad_ptr, LINES + 5, COLS + 7, 5, 5, 21, 19);
    sleep(1);

    delwin(pad_ptr);

    endwin();
    exit(EXIT_SUCCESS);
}

```

## 第七章 数据管理

本章中的主要内容
- 动态内存管理：可以做什么以及Linux不允许做什么。
- 文件锁定：协调锁、共享文件的锁定区域和避免死锁。
- dbm数据库：一个大多数linux系统都提供的、基本的、不基于SQL的数据库函数库。

### 7.1 

linux中的内存管理中一般情况下是265M的堆栈大小。linux中可以使用标准的c语言接口进行内存分配。注意当linux中的内存耗尽的时候，会linux内核会使用交换空间(独立的磁盘空间)。内核会在物理内存和交换空间之间移动数据和程序代码。
每个Linux系统中运行的程序都只能看到属于自己的内存映像，不同的程序看到的内存映像不同。只有操作系统知道物理内存是如何安排的。

Linux可以允许输出空指针，但是不允许空指针写入内存。

**linux中一旦程序调用free释放了一块内存，它就不再属于这个进程。它将由malloc函数库负责管理。在对一块内存调用free之后，就绝不能再对其进行读写操作了**

其它内存释放函数：

- `void *calloc(size_t number_of_elements,size_t element_size);`:结构数组分配内存，需要元素个数和每个元素的大小作为其参数。并且分配的内存全部初始化为0.返回数组中第一个元素的指针。
- `void *realloc(void *existing_memory,size_t new_size);`:释放内存。


### 7.2 文件锁定

文件锁与线程锁类似，都是使用锁来进行的。可以使用原子文件锁，来直接锁定文件，也可以只锁定文件的一部分，从而可以独享对这一部分内容的访问。

创建文件锁后，通常被放在一个特定的位置，linux中通常会在/var/spool目录中创建一个文件。
注意：**文件锁只是建议锁，不是强制锁**

可以贼打开文件时使用锁，想干参数在`fcnt.h`中，在此不做过多叙述。第三章中函数有详细叙述。

文件读取使用fread时，将整个文件都读取到了内存中，再传递给程序，文件内容中未被锁定的部分。当其它部分被更改后，内容锁消失；但是因为程序获取的还是fread上次读取的内容，因此会产生数据的错误。可以使用`read()`和`write()`读取部分内容，避免这个问题的发生。

下面是一个简单的读写锁示例

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>

const char *test_file = "/tmp/test_lock";

int main() {
    int file_desc;
    int byte_count;
    char *byte_to_write = "A";
    struct flock region_1;
    struct flock region_2;
    int res;

        /* open a file descriptor */

    file_desc = open(test_file, O_RDWR | O_CREAT, 0666);
    if (!file_desc) {
        fprintf(stderr, "Unable to open %s for read/write\n", test_file);
        exit(EXIT_FAILURE);
    }

        /* put some data in the file */

    for(byte_count = 0; byte_count < 100; byte_count++) {
        (void)write(file_desc, byte_to_write, 1);
    }

        /* setup region 1, a shared lock, from bytes 10 -> 30 */

    region_1.l_type = F_RDLCK;
    region_1.l_whence = SEEK_SET;
    region_1.l_start = 10;
    region_1.l_len = 20; 
    
        /* setup region 2, an exclusive lock, from bytes 40 -> 50 */

    region_2.l_type = F_WRLCK;
    region_2.l_whence = SEEK_SET;
    region_2.l_start = 40;
    region_2.l_len = 10;

        /* now lock the file */

    printf("Process %d locking file\n", getpid());
    res = fcntl(file_desc, F_SETLK, &region_1);
    if (res == -1) fprintf(stderr, "Failed to lock region 1\n");
    res = fcntl(file_desc, F_SETLK, &region_2);
    if (res == -1) fprintf(stderr, "Failed to lock region 2\n");    

        /* and wait for a while */
        
    sleep(60);

    printf("Process %d closing file\n", getpid());    
    close(file_desc);
    exit(EXIT_SUCCESS);
}
```

下图显示了当程序开始等待时文件锁定的状态

![文件锁定状态](https://wangpengcheng.github.io/img/2019-09-11-22-05-31.png)

#### 7.2.5 其它锁命令

使用lockf函数。通过文件描述符进行操作

```c
#include <unistd.h>
int lockf(int filds,int function ,off_t size_to_lock);
```
function参数取值如下：

- F_ULOCK:解锁
- F_LOCK:设置独占锁
- F_TLOCK:测试并设置独占锁
- F_TEST:特使其它进程设置的锁

size_to_clock参数是操作的字节数，它从文件的当前偏移值开始计算。

### 7.3 数据库

#### 7.3.1 abm数据库

这里主要介绍dbm数据库，这个是linux中数据库自带的基本的版本。其基本文件包含在`ndbm.h`中可以使用`-I/usr/include/gdbm -lgdbm`参数进行链接。

#### 7.3.3 dbm访问函数

```c
#include <ndbm.h>
//打开数据库

DBM *dbm_open(const char* filename,int file_open_flags,mode_t file_mode);
//存储数据库

int dbm_store(DBM *database_descriptor,datum key, datum content ,int store_mode );
//数据库查询函数

datum dbm_fetch(DBM *database_descriptor,datum key);
//关闭数据库

datum dbm_close(DBM *database_descriptor);
```
其它操作函数

```c
//删除数据库

int dbm_delete(DBM *database_descriptor,datum key);
//测试数据库错误

int dbm_error(DBM *database_descriptor);
//清除数据库中所有以被置位的错误条件标志

int dbm_clearerr(DBM *database_descriptor);
//获取第一个关键数据
int dbm_firstkey(DBM *database_descriptor);
//获取第二个关键数据

int dbm_nextkey(DBM *database_descriptor);

```
## 第 八 章 MySQL

### MySQL安装

参考连接:
- [Ubuntu 16.04 mysql安装配置](https://www.jianshu.com/p/3111290b87f4)
- [Ubuntu18.04 安装MySQL](https://blog.csdn.net/weixx3/article/details/80782479)

可以从官网下载，也可以直接使用`sudo apt-get install mysql-server`进行安装。

安装完成后使用`sudo mysql_secure_installation`命令进行初始化设置。

再使用`sudo mysql -uroot -p`进行登录。`use database`使用数据库。

具体的请参考mysql对应文章。

### 8.3 使用c语言访问mysql

使用样例：

```c
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"

MYSQL my_connection;
MYSQL_RES *res_ptr;
MYSQL_ROW sqlrow;

int main(int argc, char *argv[]) {
   int res;

   mysql_init(&my_connection);  
   if (mysql_real_connect(&my_connection, "localhost", "rick", 
                                              "secret", "foo", 0, NULL, 0)) {
   printf("Connection success\n");
   
   res = mysql_query(&my_connection, "SELECT childno, fname, age FROM children WHERE age > 5");

   if (res) {
      printf("SELECT error: %s\n", mysql_error(&my_connection));
   } else {
      res_ptr = mysql_store_result(&my_connection);
      if (res_ptr) {
       printf("Retrieved %lu rows\n", (unsigned long)mysql_num_rows(res_ptr));
       while ((sqlrow = mysql_fetch_row(res_ptr))) {
         printf("Fetched data...\n");
       }
       if (mysql_errno(&my_connection)) {
         fprintf(stderr, "Retrive error: %s\n", mysql_error(&my_connection)); 
       }
       mysql_free_result(res_ptr);
      }

   }
   mysql_close(&my_connection);

   } else {
      fprintf(stderr, "Connection failed\n");
      if (mysql_errno(&my_connection)) {
         fprintf(stderr, "Connection error %d: %s\n",
                  mysql_errno(&my_connection), mysql_error(&my_connection));
      }
   }

   return EXIT_SUCCESS;
}
```

## 第 9 章 开发工具

### 9.2 make命令和makefile

make 选项参数：

- -k:make发生错误时仍然继续执行。
- -n:马克输出将要执行的操作而不进行执行。
- -f:使用那个文件作为makefile文件。

具体参看makefile相关文章

- [跟我一起写makefile](https://wangpengcheng.github.io/2019/07/06/write_makefile_with_me/)

### 9.3 源代码控制

- SCCS:源代码控制系统
- RCS：版本控制系统
- CVS：并发版本控制系统
- 