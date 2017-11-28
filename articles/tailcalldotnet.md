
# Tail call in dotnet #

In the programming as the recursion we call the function which is directly, or indirectly calls itself: A->B / A->B->A.
The recursive calls obviously might be a reason of stack overflowing [StackOverflowException].
On the other hand, the recursive styled functions allow us clearly write our ideas, as Fibonacci, Fractal, Tree-walk function etc.

    private static long Factorial(int n)
    {
         return n== 0 ? 1 : n* Factorial(n - 1);
    }

Because of being aware of the possible failure, there are several well-known approaches to resolve stack overflow issue, they are: looping and inlining the recursive function. From the programmer's perspective the suitable approach might be re-write the function with the loop, inlining is rarely applicable, but with code, readability sacrificing. The inlining and another method of reducing stack calls for the recursive function is the "tail call". Both methods are resolved via the compiler. This is tremendously important both for C# dotnet developers and, moreover, for the functional languages programmers - aka F#. For functional languages, it is natural to express the intention with the recursion.
	
	let rec factorial n =
		match n with
		| 0 | 1 -> 1
		| _ -> n * factorial(n-1)

Thanks to Wiki: "In computer science, a tail call is a subroutine call performed as the final act of a procedure. If a tail call might lead to the same subroutine being called again later in the call chain, the subroutine is said to be tail-recursive, which is a special case of recursion. Tail recursion (or tail-end recursion) is particularly useful, and often easy to handle in implementations."


----------


### Consider the example ###

 of the factorial function:

	private static long Factorial(int n)
	{
		return n== 0 ? 1 : n* Factorial(n - 1);
	}

if we consider the IL code we'll discover the truly recursive self-call:

	.method private hidebysig static
	    int64 FactorialWithoutTailing (
	        int32 n
	    ) cil managed
	{
	    // Method begins at RVA 0x2f78
	    // Code size 18 (0x12)
	    .maxstack 8
	
	    IL_0000: ldarg.0
	    IL_0001: brfalse.s IL_000f
	
	    IL_0003: ldarg.0
	    IL_0004: conv.i8
	    IL_0005: ldarg.0
	    IL_0006: ldc.i4.1
	    IL_0007: sub
	    IL_0008: call int64 Jit_TailCalling::Factorial(int32)
	    IL_000d: mul
	    IL_000e: ret
	
	    IL_000f: ldc.i4.1
	    IL_0010: conv.i8
	    IL_0011: ret
	} // end of method Jit_TailCalling::Factorial

consider the code: the recursive call is not in the tail position, although it is last in the function text. However, automatic optimization does not occur, because, in fact, the last statement is multiplication.
Here is the instructions order:

- comparing
- function call
- subtraction on the stack
- multiplication
- return

To allow the compiler to generate the tail call we have to modify the function to shift the recursive call in the last position.

	private static long FactorialWithTailing(int pos, int depth)
	{
	     return pos == 0 ? depth : FactorialWithTailing(pos - 1, depth * pos);
	}

and the entrypoint function:

     private static long FactorialWithTailing(int depth)
     {
          return FactorialWithTailing(1, depth);
     }

The entrypoint function serves the call with constant 1. Consider the new IL code:

	.method private hidebysig static
	    int64 FactorialWithTailing (
	        int32 depth
	    ) cil managed
	{
	    // Method begins at RVA 0x2f9e
	    // Code size 8 (0x8)
	    .maxstack 8
	
	    IL_0000: ldc.i4.1
	    IL_0001: ldarg.0
	    IL_0002: call int64 Jit_TailCalling::FactorialWithTailing(int32, int32)
	    IL_0007: ret
	} // end of method Jit_TailCalling::FactorialWithTailing

for the entrypoint:

	.method private hidebysig static
	    int64 FactorialWithTailing (
	        int32 pos,
	        int32 depth
	    ) cil managed
	{
	    // Method begins at RVA 0x2f8b
	    // Code size 18 (0x12)
	    .maxstack 8
	
	    IL_0000: ldarg.0
	    IL_0001: brfalse.s IL_000f
	
	    IL_0003: ldarg.0
	    IL_0004: ldc.i4.1
	    IL_0005: sub
	    IL_0006: ldarg.1
	    IL_0007: ldarg.0
	    IL_0008: mul
	    IL_0009: call int64 Jit_TailCalling::FactorialWithTailing(int32, int32)
	    IL_000e: ret
	
	    IL_000f: ldarg.1
	    IL_0010: conv.i8
	    IL_0011: ret
	} // end of method Jit_TailCalling::FactorialWithTailing

Pay attention to recursive call instruction

for the non-modified function:

    IL_0008: call int64 Jit_TailCalling::Factorial(int32)
    IL_000d: mul
    IL_000e: ret

for the modified function:

    IL_0008: mul
    IL_0009: call int64 Jit_TailCalling::FactorialWithTailing(int32, int32)
    IL_000e: ret

the key difference is the second case - the multiplication call is above the function calling that's why it is performed at the stage of computing arguments for the function call. That modification allows CLR automatically within JITting generate non-recursive calls and perform the computation on the single stack.


----------


### F# workflow ###

For the recursive function

	let rec factorial n =
	     match n with
	     | 0 | 1 -> 1
	     | _ -> n * factorial(n-1)

the IL code:

	.method assembly hidebysig
	    instance int32 factorial (
	        int32 n
	    ) cil managed
	{
	    .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = (
	        01 00 00 00
	    )
	    // Method begins at RVA 0x2338
	    // Code size 28 (0x1c)
	    .maxstack 8
	
	    IL_0000: ldarg.1
	    IL_0001: switch (IL_001a, IL_001a)
	
	    IL_000e: ldarg.1
	    IL_000f: ldarg.0
	    IL_0010: ldarg.1
	    IL_0011: ldc.i4.1
	    IL_0012: sub
	    IL_0013: callvirt instance int32 Program/TailCallDetector::factorial(int32)
	    IL_0018: mul
	    IL_0019: ret
	
	    IL_001a: ldc.i4.1
	    IL_001b: ret
	} // end of method TailCallDetector::factorial

looks the same as for the previous example. The function modification will reveal the *difference* how C# and F# compilers work.

	let factorial1 n =
		let rec loop i acc =
			match i with
			| 0 | 1 -> acc
			| _ -> loop (i-1) (acc * i)
		loop n 1

the IL code:

	.method assembly hidebysig
	    instance int32 factorial1 (
	        int32 n
	    ) cil managed
	{
	    .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = (
	        01 00 00 00
	    )
	    // Method begins at RVA 0x2358
	    // Code size 8 (0x8)
	    .maxstack 8
	
	    IL_0000: ldarg.1
	    IL_0001: ldc.i4.1
	    IL_0002: call int32 Program::loop@47(int32, int32)
	    IL_0007: ret
	} // end of method TailCallDetector::factorial1

the difference is compiler has generated the loop for this function at the compile time: `IL_0002: call int32 Program::loop@47(int32, int32)`

	.method assembly static
	    int32 loop@47 (
	        int32 i,
	        int32 acc
	    ) cil managed
	{
	    // Method begins at RVA 0x20fc
	    // Code size 28 (0x1c)
	    .maxstack 8
	
	    // loop start
	        IL_0000: ldarg.0
	        IL_0001: switch (IL_001a, IL_001a)
	
	        IL_000e: ldarg.0
	        IL_000f: ldc.i4.1
	        IL_0010: sub
	        IL_0011: ldarg.1
	        IL_0012: ldarg.0
	        IL_0013: mul
	        IL_0014: starg.s acc
	        IL_0016: starg.s i
	        IL_0018: br.s IL_0000
	    // end loop
	
	    IL_001a: ldarg.1
	    IL_001b: ret
	} // end of method Program::loop@47

This solution solves the problem but with another option: to generate new method to calculate the recursion into a loop. But to gain the tail call at JITting one has to change the function and extract the tail call to separate one.

	let factorial2 n =
	let rec tailCall n f =
	if n <= 1 then
	     f()
	else
	     tailCall (n - 1) (fun () -> n * f())
	
	tailCall n (fun () -> 1)

as the result:

	.method assembly hidebysig
	    instance int32 factorial2 (
	        int32 n
	    ) cil managed
	{
	    .custom instance void [mscorlib]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = (
	        01 00 00 00
	    )
	    // Method begins at RVA 0x2364
	    // Code size 14 (0xe)
	    .maxstack 8
	
	    IL_0000: ldarg.1
	    IL_0001: newobj instance void Program/clo@60::.ctor()
	    IL_0006: tail.
	    IL_0008: call int32 Program::tailCall@54(int32, class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32>)
	    IL_000d: ret
	} // end of method TailCallDetector::factorial2

here is the need call just above the return from the method, and for the entrypoint function:
	
	IL_0008: call int32 Program::tailCall@54(int32, class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32>)

which is obviously with the tail call:  `tailCall n (fun () -> 1)`

	.method assembly static
	    int32 tailCall@54 (
	        int32 n,
	        class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32> f
	    ) cil managed
	{
	    // Method begins at RVA 0x211c
	    // Code size 30 (0x1e)
	    .maxstack 8
	
	    // loop start
	        IL_0000: ldarg.0
	        IL_0001: ldc.i4.1
	        IL_0002: bgt.s IL_000e
	
	        IL_0004: ldarg.1
	        IL_0005: ldnull
	        IL_0006: tail.
	        IL_0008: callvirt instance !1 class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32>::Invoke(!0)
	        IL_000d: ret
	
	        IL_000e: ldarg.0
	        IL_000f: ldc.i4.1
	        IL_0010: sub
	        IL_0011: ldarg.0
	        IL_0012: ldarg.1
	        IL_0013: newobj instance void Program/'tailCall@58-1'::.ctor(int32, class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32>)
	        IL_0018: starg.s f
	        IL_001a: starg.s n
	        IL_001c: br.s IL_0000
	    // end loop
	} // end of method Program::tailCall@54

here is the call we are interested in:

	IL_0006: tail.
	IL_0008: callvirt instance !1 class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<class [FSharp.Core]Microsoft.FSharp.Core.Unit, int32>::Invoke(!0)
	IL_000d: ret


----------

### Check the code ###

To detect JIT tail calls and analyze functions there are two ETW events:

*MethodJitInliningSucceeded Event*
*MethodJITTailCallFailed Event*

These events are used in the TailCallDiagnoser that I've recently contributed to the BanchmarkDotNet project.

Here is the examples of the diagnoser demo and output:

The C# demo:

	  [Diagnostics.Windows.Configs.TailCallDiagnoser(false)]
	  [LegacyJitX86Job, LegacyJitX64Job, RyuJitX64Job]
	  public class Jit_TailCalling
	  {
	  [Benchmark]
	  public long Calc()
	  {
	  return FactorialWithoutTailing(7) - FactorialWithTailing(7);
	  }
	
	  private static long FactorialWithoutTailing(int depth)
	  {
	  return depth == 0 ? 1 : depth * FactorialWithoutTailing(depth - 1);
	  }
	
	  private static long FactorialWithTailing(int pos, int depth)
	  {
	  return pos == 0 ? depth : FactorialWithTailing(pos - 1, depth * pos);
	  }
	
	  private static long FactorialWithTailing(int depth)
	  {
	  return FactorialWithTailing(1, depth);
	  }
	  }

output:
	
	--------------------
	Jit_TailCalling.Calc: LegacyJitX64(Jit=LegacyJit, Platform=X64, Runtime=Clr)
	--------------------
	
	--------------------
	Jit_TailCalling.Calc: LegacyJitX86(Jit=LegacyJit, Platform=X86, Runtime=Clr)
	--------------------
	
	--------------------
	Jit_TailCalling.Calc: RyuJitX64(Jit=RyuJit, Platform=X64)
	--------------------
	Caller: <null>.<null> - <null>
	Callee: BenchmarkDotNet.Samples.JIT.Jit_TailCalling.FactorialWithTailing - int64 (int32,int32)
	Tail prefix: False
	Tail call type: RecursiveLoop
	--------------------

the F# demo:

	[<TailCallDiagnoser>]
	type TailCallDetector () =
	
	  let rec factorial n =
		  match n with
		  | 0 | 1 -> 1
		  | _ -> n * factorial(n-1)
	
	  let factorial1 n =
		  let rec loop i acc =
			  match i with
			  | 0 | 1 -> acc
			  | _ -> loop (i-1) (acc * i)
		  loop n 1
	
	  let factorial2 n =
		  let rec tailCall n f =
		  if n <= 1 then
			  f()
		  else
			  tailCall (n - 1) (fun () -> n * f())
	
	  tailCall n (fun () -> 1)
	
	  [<Params (7)>]
	
	  member val public facRank = 0 with get, set
	
	  [<Benchmark>]
	  member self.test () =
	  factorial self.facRank
	
	  [<Benchmark>]
	  member self.test1 () =
	  factorial1 self.facRank
	
	  [<Benchmark>]
	  member self.test2 () =
	  factorial2 self.facRank

output:
	
	--------------------
	TailCallDetector.test: DefaultJob [facRank=7]
	--------------------
	
	--------------------
	TailCallDetector.test1: DefaultJob [facRank=7]
	--------------------
	
	--------------------
	TailCallDetector.test2: DefaultJob [facRank=7]
	--------------------
	Caller: <null>.<null> - <null>
	Callee: Program+TailCallDetector.factorial2 - instance int32 (int32)
	Tail prefix: True
	Tail call type: HelperAssistedTailCall
	--------------------

**Limitations:**

 - Windows only 
 - x64 


----------


**Useful links**:
 - [https://github.com/dotnet/BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet)
 - [https://github.com/dotnet/roslyn/issues/1235](https://github.com/dotnet/roslyn/issues/1235)
 - [https://github.com/fsharp/fslang-design/blob/master/RFCs/FS-1011-warn-on-recursive-without-tail-call.md](https://github.com/fsharp/fslang-design/blob/master/RFCs/FS-1011-warn-on-recursive-without-tail-call.md)
 - [https://blogs.msdn.microsoft.com/fsharpteam/2011/07/08/tail-calls-in-f/](https://blogs.msdn.microsoft.com/fsharpteam/2011/07/08/tail-calls-in-f/)
 - [https://blogs.msdn.microsoft.com/clrcodegeneration/2010/05/07/jit-etw-tail-call-event-fail-reasons/](https://blogs.msdn.microsoft.com/clrcodegeneration/2010/05/07/jit-etw-tail-call-event-fail-reasons/)
 - [https://docs.microsoft.com/en-us/dotnet/framework/performance/jit-tracing-etw-events](https://docs.microsoft.com/en-us/dotnet/framework/performance/jit-tracing-etw-events)
 - [https://blogs.msdn.microsoft.com/clrcodegeneration/2009/05/11/jit-etw-tracing-in-net-framework-4/](https://blogs.msdn.microsoft.com/clrcodegeneration/2009/05/11/jit-etw-tracing-in-net-framework-4/)
