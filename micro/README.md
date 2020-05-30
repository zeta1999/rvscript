# micro

An implementation of syscall-accelerated C++ binaries for the RISC-V scripting backend. CMake is used to make it easy to add new binaries.

Check the CMakeLists.txt for build options, although the only really relevant one is `RTTI_EXCEPT` which enabled C++ RTTI and exceptions should you want that.

## Getting started

You need the RISC-V compiler specified in the main README to be able to compile this. It's set up so that if the compiler is installed into `~/riscv` it will just work.

## Troubleshooting

- I'm unable to execute a specific function on a remote machine.
	- You have to remotely execute a function that exists on that machine, and not the machine you are executing from.
- After using `vmcall()` on a machine it stops working
	- If that machine was in the middle of something when getting called into, then the registers would be clobbered and so when you try to resume the machine later it would be in an exited state. That is, it would be at the point where the vmcall ended, instead of the point where it left off the last time. You can use `preempt()` to call into a machine which is running some long-term operation.
- Why can't I block the main thread?
	- Many reasons. Most importantly because the main thread is going to get clobbered by other functions calling into the machine. It was a design decision (that I have revisited many times) to only allow blocking other threads. It makes everything simple in both the implementation and for the user too, when you think about it.
- Why don't you have real coroutines?
	- Well, they are arriving now in all relevant compilers, so you can probably use them right now. But. They are not good for driving script in game engines, even though they are compiler-assisted and insanely performant. The reason is that only the game engine knows when events happen, and because of this the game engine needs the ability to resume any coroutine at any point in time, by suspending anything that was happening before and then waking up the right thread. Yes, we need threads for that, and there are native micro-threads that are written for this task.
- I'm unable to throw a bog-standard C++ exception
	- The C++ standard library is not linked in, since it requires basically a standard C library, which we don't want to provide. It would be slow and inefficient for a myriad of reasons. If you want a fast C++ library use EASTL. I have a fork of it where I change 4 lines to make it build on RISC-V. You can still use most C++ containers that are inline-able, such as std::vector.
- Do you support any sanitizers?
	- No, but I could drop in support for ubsan. However, personally I think it's not going to provide much value in this particular environment. See my barebones repository for how it's done.
- Backtraces?
	- Possible. I have read that when linking in libgcc you should have access to `__builtin_return_address (N > 0)`, which would make it possible to implement backtraces inside the environment. You have some limited access to backtraces from outside the machine, which automatically gets printed any time a CPU exception happens.
- I can't enable `-fstack-protector`?
	- Unfortunately there is a compiler bug preventing using this for now. Maybe there is a work-around.
- Should I enable `-gc-sections`?
	- Probably not, it prevents some optimizations, and the size of the binaries matters much less than the locality of the code being executes. If you are good with hot/cold attributes and other things that make the code stay inside the same page as much as possible, you will not lose any performance.
- Should I enable the RISC-V compressed instructions (C-extension)?
	- Maybe. I've seen it increase performance overall, but I've also seen the opposite more often. It's disabled by default.
- Can I build these binaries on Windows?
	- With Docker I don't see a problem at all. I would not try to use this repository with a docker container that compiles the code, rather put everything in the container and build the executables from within. Then share the output directories so that you build them where the engine already finds them. I'm probably simplifying somewhat, but it should be doable.
	- With the WSL2 it should be even easier, however I have no experience with that. Definitely something I want to try, and I'll add some scripts for that when I try it.

