/*
 * ELF executable loading
 * Copyright (c) 2003, Jeffrey K. Hollingsworth <hollings@cs.umd.edu>
 * Copyright (c) 2003, David H. Hovemeyer <daveho@cs.umd.edu>
 * $Revision: 1.29 $
 * 
 * This is free software.  You are permitted to use,
 * redistribute, and modify it as specified in the file "COPYING".
 */

#include <geekos/errno.h>
#include <geekos/kassert.h>
#include <geekos/ktypes.h>
#include <geekos/screen.h>  /* for debug Print() statements */
#include <geekos/pfat.h>
#include <geekos/malloc.h>
#include <geekos/string.h>
#include <geekos/user.h>
#include <geekos/elf.h>


/**
 * From the data of an ELF executable, determine how its segments
 * need to be loaded into memory.该函数的实现要求和项目1相同。
 * @param exeFileData buffer containing the executable file
 * @param exeFileLength length of the executable file in bytes
 * @param exeFormat structure describing the executable's segments
 *   and entry address; to be filled in
 * @return 0 if successful, < 0 on error
 */
int Parse_ELF_Executable(char *exeFileData, ulong_t exeFileLength,
    struct Exe_Format *exeFormat)
{
    int i;
	elfHeader * elfh = (elfHeader*) exeFileData;
	programHeader * proh = (programHeader*) (exeFileData + elfh->phoff);
	struct Exe_Segment * seg = exeFormat->segmentList;
	for(i = elfh->phnum; i > 0; i--)
	{
		seg->offsetInFile = proh->offset;
		seg->lengthInFile = proh->fileSize;
		seg->startAddress = proh->vaddr;
		seg->sizeInMemory = proh->memSize;
		seg->protFlags = proh->flags;
		seg++;
		proh++;
	}
	exeFormat->numSegments = elfh->phnum;
	exeFormat->entryAddr = elfh->entry;
	
	return 0;
}

