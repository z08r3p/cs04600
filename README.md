java c
18-213 /   15-613, Fall 2024
Malloc   Lab:   Writing   a   Dynamic   Storage   Allocator
Assigned:   Monday, October   21, 2024
1 Introduction
In this lab you will write a dynamic memory allocator which will consist of the malloc, free, realloc, and calloc functions. Your goal is to implement an allocator that is correct, efficient, and fast.
We strongly encourage you to start early. The total time you spend designing and debugging can easily eclipse the time you spend coding.
Bugs can be especially pernicious and difficult to track down in an allocator, and you will probably spend a significant amount of time debugging your code. Buggy code will not get any credit.
This lab has been heavily revised from previous versions. Do not rely on advice or information you may find on the Web or from people who have done this lab before. It will most likely be misleading or outright wrong. Be sure to read all of the documentation carefully and especially study the baseline implementation we have provided.
2          Logistics
This is an individual project.   You should do this lab   on   one   of the Shark machines.
To   get   your   lab   materials, click   “Download   Handout” on   Autolab,   enter   your   Andrew   ID,   and   follow   the instructions.   Then, clone your repository on a Shark machine by running:
$      git      clone      https://github.com/18-x13/malloclab-   .gitThe   only   file you will turn in   is   mm   .c.    All the   code   for   your   allocator   must   be   in   this   file.    The rest   of the      provided   code   allows   you to   evaluate your   allocator.    Using the   command make will   generate   four   driver      programs:   mdriver, mdriver-dbg, mdriver-emulate, and mdriver-uninit, as described in section6.   Your final autograded score is computed by driver.pl, as described in section7.1.
To test your code for the checkpoint submission, run mdriver and/or driver.pl with the   -C flag.   To   test your code for the final submission, run mdriver and/or driver.pl with no flags.
These   commands   will   report   accurate   utilization   numbers   for   your   allocator.      They   will   only   report   approximate throughput numbers.   The Autolab servers will generate different throughput numbers, and   the servers’ numbers will determine your actual score.   This is discussed in   more   detail   in   Section   7.
3          Required   Functions
Your allocator must implement the following functions.   They are declared   for you   in mm.hand   you   will   find   starter   definitions   in   mm   .c.   Note   that   you   cannot   alter   mm.hinthis   lab.
bool         mm_init(void);
void      *malloc(size_t      size);   void            free(void      *ptr);
void      *realloc(void      *ptr,      size_t      size);
void      *calloc(size_t      nmemb,      size_t      size);   bool         mm_checkheap(int);
We provide you two versions of memory allocators:
mm   .c:   A   fully-functional   implicit-list   allocator.   We   recommend   that   you   use   this   code   as   your   starting   point.   Note   that   the   provided   code   does   not   implement   block   coalescing.   The   absence   of   this   feature   will   cause    external fragmentation to be very high, so you should implement coalescing.   We   strongly   recommend      considering all cases you need to   implement   before   writing   code   for   coalesce_block;   the   lecture      slides should help you identify and reason about these   cases.
mm-naive.c:   A functional   implementation that runs quickly but gets very poor utilization, because it never   reuses any blocks of memory.
Your   allocator   must   run   correctly   on   a   64-bit   machine. It   must   support   a   full   64-bit   address   space,   even though   current   implementations   of   x86-64 machines   support   only   a   48-bit   address   space.
Your   submitted   mm   .c   must   implement   the   following   functions:
bool   mm_init(void):    Performs any necessary initializations, such as allocating the initial heap area.   The   return value should be   false if there was a problem in performing the initialization, true otherwise.      You   must   reinitialize   all   of   your   data   structures   each   time   this   function   is   called, because   the   drivers call your mm_init function every time they begin a new trace to reset to an empty heap.
void      *malloc(size_t    size):    Returns a pointer to an allocated block payload of at least size bytes. The   entire   allocated   block   should   lie   within   the   heap   region   and   should   not   overlap   with   any   other   allocated block.
Your   malloc   implementation   must   always   return   16-byte   aligned   pointers, even   if   size   is   smaller   than 16.
void      free(void      *ptr)      : If ptr is NULL, does nothing.   Otherwise, ptr must point to the beginning of a   block payload returned by a previous call to malloc, calloc, or realloc and not already freed.   This   block is deallocated.   Returns nothing.
void      *realloc(void      *ptr,      size_t      size):      Changes the size of a previously allocated block.
If   size is nonzero   and ptr is not   NULL,   allocates   a   new   block   with   at   least   size bytes   of payload,   copies as much data from ptr into the new block as will fit   (that is,   copies the   smaller   of   size,   or   the
payload size of ptr, bytes), frees ptr, and returns the new   block.
If   size is nonzero but ptr is NULL, does the same thing as malloc(size).   If   size is zero, does the same thing as   free(ptr) and then returns NULL.
Your realloc implementation will have only minimal impact on measured throughput or   utilization.   A   correct, simple implementation will suffice.
void      *calloc(size_t   nmemb,    size_t    size):   Allocates memory for   an   array   of nmemb elements   of   size   bytes   each, initializes   the   memory   to   all   bytes   zero, and   returns   a   pointer   to   the   allocated   memory.   Your calloc implementation will have only minimal impact on measured throughput or utilization.   A   correct, simple implementation will suffice.
bool   mm_checkheap(int    line):    Scans the entire heap and checks it   for   errors.   This   function   is   called   the heap consistency checker, or simply   heap   checker.A quality heap checker is essential for debugging your malloc implementation.   Many malloc bugs   are too subtle to debug using conventional gdb techniques.   A heap consistency checker   can   help   you   isolate the specific operation that causes your heap to become inconsistent.Because   of   the   importance   of   the   consistency   checker, it   will   be   graded, by   hand; section   7.2   describes the   requirements   for   your   implementation   in   greater   detail.   We   may   also   require   you   to   write   your   heap checker before coming to office hours.The   mm_checkheap   function   takes   a   single   integer   argument   that   you   can   use   any   way   you   want.   One technique is to use this argument to pass in the line number where it was called, using the                  LINE       macro:
mm_checkheap(               LINE               );
This allows you to print the line number where mm_checkheap was called, if you detect a problem with   the   heap.The   driver   will   sometimes   call   mm_checkheap; when   it   does   this   it   will   always   pass   an   argument   of   0.
The semantics of malloc, realloc,   calloc, and   free match the semantics   of the functions   with the   same   names   in   the   C   library.   You   can   type   man      malloc   in   the   shell   for   more   documentation.
4          Support   RoutinesTo   satisfy   allocation   requests, dynamic   memory   allocators   must   themselves   request   memory   from   the   operating system, using   “primitive”   system operations that   are less   flexible than   malloc   and   free.    In this   lab,   you   will use a simulated version of one such primitive.   It is implemented   for you   in memlib   .c and   declared   in   memlib.h.
void      *mem_sbrk(intptr_t    incr):      Expands the heap by incr bytes, and returns a generic pointer to the   first byte of the newly allocated heap area.   If the heap cannot be made any larger, returns   (void    *)   -1.   (Caution:   this   is   different   from   returning   NULL.)
Each time your mm_init function is called, the heap has   just been reset to zero bytes long.
mem_sbrk   cannot   make   the   heap   smaller; it   will   fail   (returning   (void      *)    -1)   if   size   is   negative.
(Data type   intptr_tis defined to be a signed integer large enough to   hold a pointer.   On   our machines   it is the same size   as   size_t, but   signed.)
This function is based on the Unix system callsbrk, but we have simplified   it   by removing   the   ability   to make the heap   smaller.
You   can   also   use   these   helper   functions, declared   in   memlib.h:
void      *mem_heap_lo(void):    Returns   a   generic   pointer   to   the   first   valid   byte   in   the   heap.   void      *mem_heap_hi(void):    Returns   a   generic   pointer   to   the   last   valid   byte   in   the   heap.
Caution:   The   definition   of   “last   valid   byte” may   not   be   intuitive!   If   your   heap   is   8 bytes   large, then   the last   valid   byte   will   be   7 bytes   from   the   start—not   an   aligned   address.
size_t   mem_heapsize(void):    Returns the current size of the heap in bytes.
You   can   also   use   the   following   standard   C   library   functions,   but   only   these:    memcpy,   memset,   printf,   fprintf, and sprintf.
Your   mm.c   code   may   only   call   the   externally-defined   functions   that   are   listed   in   this   section.   Otherwise,   it must be completely self-contained.


5          Programming   Rules
•   Any   allocator that   attempts to detect which   trace   is   running   will   receive   a penalty   of   20   points.    On   the other hand, you should feel free to write an adaptive   allocator—one that   dynamically   tunes   itself   according to the general characteristics of the different traces.
•   You may not change any of the interfaces in mm.h, or any of   the other C   source files   and   headers besides mm   .c.   (Autolab only 代 写Malloc Lab: Writing a Dynamic Storage Allocator Fall 2024R
代做程序编程语言processes your mm.c; it will not see changes   you   make   to   any   other   file.)   However,   we strongly encourage you to use   static helper functions in mm   .c to break up your code into small,   easy-to-understand segments.
•   You may not change the Makefile (again, Autolab will not see   any   changes   you   make   there)   and   your   code must compile with no warnings using the warnings flags we   selected.
•   You   are   not   allowed   to   declare   large   global   data   structures   such   as   large   arrays,   trees,   or   lists   in   mm   .c.   You   are   allowed   to   declare   small   global   arrays,   structs,   and   scalar   variables,   and   you   may   have   as   much    constant data (defined with the const qualifier) as you like.   Specifically, you may   declare   no   more      than   128 bytes of writable global variables,   total.    This is   checked   automatically,   as   described   in      Section   7.1.4.The   reason   for   this   restriction   is   that   global   variables   are   not   accounted   for   when   calculating   your   memory utilization.   If you need a large data structure for some   reason,   you   should   allocate   space   for   it   within the heap, where it will count toward external fragmentation.
•   Dynamic memory allocators cannot avoid doing operations   that   the   C   standard   labels   as   “undefined   behavior.”   They need to treat the heap as   a   single   huge   array   of bytes   and reinterpret   those   bytes   as   different data types at different times.   It is rarely appropriate to write   code   in   this   style, but   in   this   lab   it   is   necessary.We   ask   you   to   minimize   the   amount   of undefined   behavior   in   your   code.    For   example,   instead   of   directly casting between pointer types, you should explicitly alias memory through the use of   unions.   Additionally, you   should   confine   the   pointer   arithmetic   to   a   few   short   helper   functions, as   we   have   tried    to do in the handout   code.
•   In the provided baseline code, we use a zero-length array to declare a payload element   in   the block   struct.   This is a non-standard compiler extension, which, in general, we discourage the use   of, but in   this   lab   we   feel   it   is   better   than   any   available   alternative.A   zero-length   array   is   not   the   same   as   a   C99   “flexible   array   member;”   it   can   be   used   in   places   where   a flexible array member cannot.   For example, a zero-length   array   can   be   a   member   of   a   union.   Using   zero-length arrays this way is our recommended strategy for   declaring   a   block   struct   that   might   contain   payload data, or might contain something else (such   as   free   list pointers).
•   The practice of using macros instead of function definitions is now obsolete.   Modern compilers can   perform. inline substitution of small functions, eliminating the overhead   of function calls.   Use   of inline   functions provides better type checking and debugging support.In this lab, you may only use #define to define constants (macros   with   no parameters)   and   debugging   macros that are enabled or disabled at compile time.   Debugging macros must have names   that begin   with   the   prefix   “dbg_” and   they   must   have   no   effect   when   the   macro-constant   DEBUG   is   not   defined.
Here are some examples of allowed and disallowed macro   definitions:
#define DEBUG 1       OK Defines a constant
#define CHUNKSIZE (1<<12)       OK Defines a constant
#define WSIZE sizeof(uint64_t)       OK Defines a constant
#define dbg_printf(...) printf(__VA_ARGS__)       OK Debugging support
#define GET(p) (*(unsigned int *)(p))       Not OK Has parameters
#define PACK(size, alloc) ((size)|(alloc))       Not OK Has parameters

When you run make, it will run a program that checks for disallowed macro definitions   in   your   code.   This checker is overly strict—it cannot determine when   a macro   definition   is   embedded   in   a   comment      or in some part of the code that has been disabled by conditional-compilation directives.   Nonetheless,   your code must pass this checker without any warning messages.
•   The code shown in the textbook (Section 9.9.12, and   available from the   CS:APP   website)   is   a useful      source of inspiration for the lab, but it does not meet the required coding standards.   It   does   not   handle      64-bit allocations, it makes extensive use of macros instead of functions, and it relies heavily on low-   level pointer arithmetic.   Similarly, the code shown in KR does not   satisfy the   coding requirements.   You   should   use   the   provided   mm   .c   as   your   starting   point.
•   It is okay to look at any high-level descriptions of algorithms found in the   textbook   or   elsewhere, but   it is not acceptable to copy   or look   at   any   code   of malloc implementations found   online   or   in   other   sources, except   for   the   allocators   described   in   the   textbook, in   KR,   or   as   part   of   the   provided   code.
•   It   is   okay   to   adapt   code   for   useful   generic   data   structures   and   algorithms   (e.g.   linked   lists, hash   tables,   search   trees, and   priority   queues) from   any   external   source   (e.g.   KR, Wikipedia, The   Art   of   Computer Programming) as long as it was not already part of   a memory allocator.   You must include (as a comment)   an attribution of the origins of any borrowed code.
•   Your   allocator   must   always   return   pointers   that   are   aligned   to   16-byte   boundaries,   even   if   the   allocation is   smaller   than   16 bytes.   The   driver   will   check   this   requirement.


6          Driver   Programs
Four   driver   programs   are   generated   when   you   run   make.
mdriver   is used by Autolab to test your allocator’s correctness, utilization, and throughput on   a   standard   set   of   benchmark traces.
mdriver-emulate   is used by Autolab to test your allocator with a heap spanning the entire 64-bit   address   space.   In   addition   to   the   standard   benchmark   traces, it   will   run   a   set   of   giant   traces   that   make   very   large allocation requests.As   the   name   implies,   this   test   is   an   emulation:    it   does   not   actually   allocate   exabytes   of   memory.   However, it verifies that your allocator could handle allocations that large, if the hardware permitted      them.   Failing the checks performed by mdriver-emulate leads to grade   penalties,   as   described   in      section   7.1.4.
mdriver-dbg   is for you to use when debugging your   allocator.   It   is   the   same   program   as mdriver,   with   three notable differences:
1.    It is compiled with DEBUG defined, which   enables the   dbg_ macros   at   the   top   of   mm   .c.   Without   this defined, functions like dbg_printf and dbg_assert will not have any effect.
2.    It   is   compiled   with   optimization   level   -O0,   which   allows   GDB   to   display   more   meaningful   debugging information.
3.    It uses the AddressSanitizer instrumentation tool   to detect several classes of errors that are   easy   to make when writing an   allocator.
mdriver-uninit   is also for you to use when debugging.   It uses the MemorySanitizer instrumentation tool   to detect uses of uninitialized memory.mdriver-dbg,   mdriver-emulate,   and   mdriver-uninit   are   much   slower   than   mdriver,   so   they   only   report   correctness   and   the   utilization   score   for   each   trace.   All   four   programs   should   report   the   same   utilization scores for each trace that they all run (only mdriver-emulate runs the giant traces).
6.1            Trace   files
The driver programs are controlled by a set of trace   files that are   included   in   the   traces subdirectory.   Each
trace file contains a sequence of commands that instruct the driver to call your malloc, realloc, and   free   routines in some sequence.   Autolab will use the same   trace   files   to   grade   your   work.When the driver programs are run, they   will   process   each   trace   file   multiple   times:   once   to   make   sure   your implementation is correct, once to determine the space utilization,   and between   3   and   20 more   times   to   determine the throughput.Some   of the traces   are   short traces that   are included mainly   for   detecting   errors   and   debugging.    Your   utilization and performance scores on these traces do not   count toward your   grade.   The   traces   that   do   count   are marked with a ‘*’ in the output   of mdriver.




6.2          Command-line   arguments
The drivers accept the following command-line arguments.
-C:   Apply   the   scoring   standards   for   the   checkpoint, rather   than   for   the   final   submission.
-f    tracefile:    Only run the trace   tracefile.   Correctness, utilization, and performance are all tested.
-c    tracefile:    Only run the trace   tracefile,   and only   test   it   for   correctness.    This   still runs the   trace   twice, to verify that mm_init correctly resets your heap.
-v    level:    Set the verbosity level to the specified   value.   The   level   can   be   0,   1,   or   2;   the   default   level   is   1.   Raising the verbosity level causes additional diagnostic information to be printed as each   trace   file   is      processed.   This can help you determine which trace file is causing your   allocator to   fail.
-d    level:    Controls the   amount   of validity   checking performed by   the   driver.    This   is   separate   from   the   DEBUG compile-time define.
At debug level 0, very little checking is done, which is   useful   when   testing performance   only.
At   debug   level   1,   the   driver   checks   allocation   payloads   to   ensure   that   they   are   not   overwritten   by   unrelated calls into your code.   This is the default.
At   debug   level   2, the   driver   will   also   call   your   implementation   of   mm_checkheap   after   each   operation.   This   mode   is   slow, but   it   can   help   identify   the   exact   point   at   which   an   error   occurs.
Additional arguments can be listed by running mdriver    -h.





         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
