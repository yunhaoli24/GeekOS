/*
 * Common user mode functions
 * Copyright (c) 2001,2003,2004 David H. Hovemeyer <daveho@cs.umd.edu>
 * $Revision: 1.50 $
 * 
 * This is free software.  You are permitted to use,
 * redistribute, and modify it as specified in the file "COPYING".
 */

#include <errno.h>
#include <geekos/ktypes.h>
#include <geekos/kassert.h>
#include <geekos/int.h>
#include <geekos/mem.h>
#include <geekos/malloc.h>
#include <geekos/kthread.h>
#include <geekos/vfs.h>
#include <geekos/tss.h>
#include <geekos/user.h>
#include <geekos/elf.h>

/*
 * This module contains common functions for implementation of user
 * mode processes.
 */

/*
 * Associate the given user context with a kernel thread.
 * This makes the thread a user process.
 */
void Attach_User_Context(struct Kernel_Thread* kthread, struct User_Context* context)
{
    KASSERT(context != 0);
    kthread->userContext = context;

    Disable_Interrupts();

    /*
     * We don't actually allow multiple threads
     * to share a user context (yet)
     */
    KASSERT(context->refCount == 0);

    ++context->refCount;
    Enable_Interrupts();
}

/*
 * If the given thread has a user context, detach it
 * and destroy it.  This is called when a thread is
 * being destroyed.
 */
void Detach_User_Context(struct Kernel_Thread* kthread)
{
    struct User_Context* old = kthread->userContext;

    kthread->userContext = 0;

    if (old != 0) {
	int refCount;

	Disable_Interrupts();
        --old->refCount;
	refCount = old->refCount;
	Enable_Interrupts();

	/*Print("User context refcount == %d\n", refCount);*/
        if (refCount == 0)
            Destroy_User_Context(old);
    }
}

/*
 * Spawn a user process.文件中的函数Spawn（），其功能是生成一个新的用户级进程；
 * Params:
 *   program - the full path of the program executable file
 *   command - the command, including name of program and arguments
 *   pThread - reference to Kernel_Thread pointer where a pointer to
 *     the newly created user mode thread (process) should be
 *     stored
 * Returns:
 *   The process id (pid) of the new process, or an error code
 *   if the process couldn't be created.  Note that this function
 *   should return ENOTFOUND if the reason for failure is that
 *   the executable file doesn't exist.
 */
int Spawn(const char *program, const char *command, struct Kernel_Thread **pThread)
{
    /*
     * Hints:
     * - Call Read_Fully() to load the entire executable into a memory buffer
	 调用Read_Fully（）来加载整个可执行文件到内存缓冲区
     * - Call Parse_ELF_Executable() to verify that the executable is
     *   valid, and to populate an Exe_Format data structure describing
     *   how the executable should be loaded
	 调用Parse_ELF_Executable（）以验证该可执行文件是有效的，并填充描述如何可执行应加载一个Exe_Format数据结构
     * - Call Load_User_Program() to create a User_Context with the loaded
     *   program
	 调用Load_User_Program（）创建User_Context与加载的程序
     * - Call Start_User_Thread() with the new User_Context
     *
     * If all goes well, store the pointer to the new thread in
     * pThread and return 0.  Otherwise, return an error code.
	 如果一切正常，指针存储在pThread并返回pid
     */
	int result;//如果创建进程成功，返回用户态进程pid；如果创建失败，返回ENOMEM
	char *exeFileData = 0;//文件指针
	ulong_t exeFileLength;//文件长度
	struct User_Context *UserContext = 0;
	struct Exe_Format exeFormat;
	struct Kernel_Thread * thread;
	if ((result = Read_Fully(program, (void **)&exeFileData, &exeFileLength)) != 0)//Read_Fully函数是系统已实现函数，其功能是加载整个可执行文件到内存缓冲区
	{//void **是指向void*的指针。但void*是不完全的指针,无法进行++,--,+=,-=,-等操作，而void**是个合格的指针, 可以进行上述操作。void*是所有指针的"隐式基类",任何指针都可以直接赋值给void*
		Print("Oh,mygod!Failed to read file %s\n", program);
		goto fail;
	}
	if ((result = Parse_ELF_Executable(exeFileData, exeFileLength, &exeFormat)) != 0)//Parse_ELF_Executable在project1中实现
	{//其主要功能是将Read_Fully打开elf文件的内容赋给Exe_Format
		Print("Oh,mygod!Failed to parse ELF file \n");
		goto fail;
	}
	if ((result = Load_User_Program(exeFileData, exeFileLength, &exeFormat, command, &UserContext)) != 0)
	{
		Print("Oh,mygod!Failed to Load User Program\n");
		goto fail;
	}

	Free(exeFileData);
	exeFileData = 0;
	thread = Start_User_Thread(UserContext, false);
	if (thread != 0)
	{
		*pThread = thread;
		result = thread->pid;
	}
	else
	{
		result = ENOMEM;
	}
	return result;
fail:
	if (exeFileData != 0)
		Free(exeFileData);
	if (UserContext != 0)
		Destroy_User_Context(UserContext);
	return result;
}

/*
 * If the given thread has a User_Context,
 * switch to its memory space.调度程序在执行一个新的进程前调用该函数，它负责检测当前进程是否为用户进程，如果是用户进程，就切换到用户进程空间。
 *
 * Params:
 *   kthread - the thread that is about to execute
 *   state - saved processor registers describing the state when
 *      the thread was interrupted
 */
void Switch_To_User_Context(struct Kernel_Thread* kthread, struct Interrupt_State* state)
{
    /*
     * Hint: Before executing in user mode, you will need to call
     * the Set_Kernel_Stack_Pointer() and Switch_To_Address_Space()
     * functions.
	 执行在用户模式之前，您将需要调用Set_Kernel_Stack_Pointer（）tss.c设置TSS段（任务状态段）的栈指针。
	 Switch_To_Address_Space（）函数用来设置用户进程地址空间
     */

    //TODO("Switch to a new user address space, if necessary");
	static struct User_Context* s_currentUserContext;  //该变量用来保存被切换的用户态进程的User_Context指针
	//extern int userDebug;
	struct User_Context* userContext = kthread->userContext;//获取正在运行进程的User_Context
	KASSERT(!Interrupts_Enabled());
	if (userContext == 0) {    // 内核态进程，无需改变地址空间. 
		return;
	}
	if (userContext != s_currentUserContext) {	//userContext为用户态进程且没有被切换
		ulong_t esp0;
		//if (userDebug) Print("A[%p]\n", kthread);
		Switch_To_Address_Space(userContext);
		esp0 = ((ulong_t)kthread->stackPage) + PAGE_SIZE;//新进程的栈地址为原进程的栈地址加上页的大小
		//if (userDebug) Print("S[%lx]\n", esp0);
		/* 新进程的核心栈. */
		Set_Kernel_Stack_Pointer(esp0);//设置新进程的TSS段的栈指针
		/* 将被切换进程指针保存在s_currentUserContext变量 */
		s_currentUserContext = userContext;
	}
}

