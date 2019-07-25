# advanced ultimate go

## goroutines

data transfers

go will ask the compiler to check which is the most efficient integer for the platform
address integer and word

every function or goroutine go will create a new frame on the stack and execute within the frame

eveyrthing is pass by value

when the frame changes it needs input and output

data semantics are super important, go does a good job with both value and pointer semantics

every piece of code operates on its own copy of the data

as a go developer you need to know when you use value or pointer semantics

you should ask this when you define the data, instead of in the middle of writing

when youre in pointer semantic mode, be careful.

your program can deal with the value itself or the address it lives, go is value semantic first at the cost of inefficiency

pointer semantics can cause side effects, take note of what semantic is in play

every value is constructed by a function (as long as theres no globals)

compiler can perform escape analysis. if the compiler sees that a value needs to be shared up the stack

it will allocate it on the heap

the cost of things on the stack is free-ish

go abstracted away from a developer needing to think about if something lives on the stack or the heap

simplicity is about hiding complexity without hiding the cost

dont make things easy to do - make them easy to understand

use the & to show the value of sharing

go build has -gcflags and -m=2 to perform escape analysis

if the compiler doesnt know the size of a variable at compile time, it doesnt know the size to allocate on the heap

an example of this is using make on a slice using a variable for the size

if youre run out of space on the stack (maybe recursive func) then a new stack is created at double the size

go doesnt share pointers between stacks, if values need to be shared between stacks - it lives on the heap then

## garbage collection

concurrent, tri-color, non compacting garbage collection, means things on the heap dont move

garbage collection in go only deals with the marking phase, not the sweeping

the heap is an open field of memory. once 4 megs of memory are allocated on the heap a gc is run

gc has 3 phases:

1 - stop the world - mark start
10-30 microseconds, rough estimate

2 - concurrent - marking. checks to see whats in use and what isnt

3 - stop the world - mark termination
30-90 microseconds

when the first and third collection times are added up, it tries to equal 100 microseconds

when theres very tight loops, its hard to stop the world without a function call

write barrier, scans the stack from the top down to the active frame - looks for pointer vars that point to the heap

walk the heap, paint it white

when you find it, paint it grey, put it in a queue

when its used paint it black

white values are unused, black are used

gc says it can take up to 25% of the cpu and tries to finish depending on a memory limit

if it starts looking like it wont meet the gc goal, so it slows allocation on a goroutine and

recruits another core to help with gc

at stage three, resets the write barrier during a stop the world

dont touch GOGC default. it defaults at 100, you can turn it off for a small lambda or cron

check out ardanlabs blog post

garbage collection will start early, to avoid mark assists

less gc's = less gc slowdown hit

reduce the amount of allocations

## data

if you dont understand the data, you dont understand the problem

benchmarks lie all the time, where you run the benchmark matters

machine must be idle when you run a benchmark

benchmarking = b.N means find a value for n where the problem takes a certain amount of seconds which is benchtime

use go test -bench . -run none -benchtime 3s

gos model is based on the machine

current industry standard is 64 byte cache line

slices arrays and maps, arrays are the most important in go because they are the fastest on the hardware

readability make a good developer, a piece of code will be read way more than it is written

if you declare or initialize a var to a zero value - use var

dont use empty literals, use the short assignment for non zero declaration

when it comes to built in types, int string bool, use value semantics

even in structs, one exception, null. to represent null using a pointer is ok

have engineering conversation around cost of decisions

any reference types at their zero value is considered nil

reference types is array slice map

reference types should get passed around by value semantics - but read and writes use pointer semantics

empty struct: struct{}

an api can choose the data it needs and the data it returns but does not choose how

append is a warning sign if the backing array gets replaced it could cause a problem

does the mutation change the data is an indication of what semantics to use

situations where its ok to switch to pointer semantics:
decoding and unmarshalling require pointer semantics
or a leaf function - no further calls are being made afterwards

never ever go from pointer to value semantics

polymorphism means that a piece of code changes its behavior depending upon concrete data it is operating

interfaces are not real just method sets

they just define behavior around concrete data

do not design with interfaces, you discover it

start with concrete types

anytime a concrete value is stored in an interface, its an allocation

make things easy to understand not easy to do

define api

## multi thread, concurrency

its hard, do it slowly

scheduler period, all threads run in the period

thread has 3 states: running, waiting

minimal time slice = minimum time a thread can operate in a period

context switching is not free

cpu bound workload, never goes into a waiting state, context switches hurt

io bound workload, maybe goes into a waiting state, context switches help

concurrency means out of order execution

parallelism means doing two things at once

sometimes concurrency and parallelism blend together and sometimes they dont

cpu bound workloads are the easiest to do on the os

yields during go, gc, syscalls, blocks

if a thread hasnt executed an instruction in 20nanoseconds, its considered blocked and a new thread is brought in

## channels

dont think of a channel as a queue

channel serves one purpose, signaling

does the goroutine sending the signal need to guarantee the signal has been recevied

the recevie happens before the send

channels are slow

can only use make for a channel, cant do it with a literal to pre populate data

runtime.NumCPU() to figure out how many goroutines can run on the machine

if order matters do not use concurrency

## tooling

run b.ResetTimer() before benchmarking if work is being done during setup

-memprofile

go tool pprof -http :3000

webList, if you include the binary you can see the assembly

benchmark to see allocations -memprofile

look in compiler to see why -gcflags

GODEBUG=gctrace=1 ./project > /dev/null

if you import pprof \_, it adds a route for you to inspect at /debug/pprof
