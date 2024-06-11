
# Just In Time Hooking #

_The :ru: video from the DotNet meetup can be found [here](https://youtu.be/39fOc4Jr8lE) with the :uk: [slides](https://speakerdeck.com/dotnetru/gieorghii-plotnikov-just-in-time-hooking)_

At the time the [Clr](https://github.com/dotnet/coreclr) became open source, tons of opportunities and ways to improve and analyze the code came alive.
One of the ways I'd like to show you was developed when I was working on an enhancement  for the [BDN](https://github.com/dotnet/BenchmarkDotNet) project.

The question was: *how can I add the annotations for the JIT disassembly output?*
And the answer was: *hey, take a look at the [JIT dumps](https://github.com/dotnet/coreclr/blob/master/Documentation/building/viewing-jit-dumps.md).*

While I've been analyzing the JIT dumps, like

```
; Total bytes of code 23, prolog size 4 for method jitdumptest.Program:Main(ref)
; ============================================================
; Assembly listing for method jitdumptest.Program:Calc():ref
; Emitting BLENDED_CODE for X64 CPU with AVX
; optimized code
; rsp based frame
; partially interruptible
; Final local variable assignments
;
;  V00 tmp0         [V00,T00] (  4,  8   )     ref  ->  rsi         class-hnd exact
;  V01 tmp1         [V01,T01] (  4,  8   )     ref  ->  rdi         class-hnd exact
;  V02 OutArgs      [V02    ] (  1,  1   )  lclBlk (32) [rsp+0x00]
;
; Lcl frame size = 40
```

I was thinking: what do all these guys (BLENDED_CODE, optimized code, ...) mean and
*is it possible for me, to know how the JIT analyze and tries to optimize my code?*

Describing all this in a separate topic, for now, let's try to extract as much information from the JIT as it's possible.
To do it, it's reasonable to use one technique which is applicable because the source is open - [hooking](https://en.wikipedia.org/wiki/Hooking).

## Hooking ##

To get the information about my function interested to me the easiest way is to treat this information right from the compiler, I believe.
From the different techniques of hooking, I selected the runtime one by the reason for hooking JIT, and I want to get the current, real information
from my system preferable compiler, which I can expect on most computers instead of the developer version.

*That's just a brief reminder of how to hook the external code.*

To do this I have to find the method which is suitable for hooking and can expose to me its argument I can handle in the way I want.

It should:
1. be marked as `external`, otherwise, I won't be able to see it from the assembly
2. should return something virtual, so I could interact with its VTable in late binding
3. arguments mostly should give me the information I'm looking for

Lucky, In the CoreClr there is an entrypoint that suits perfectly, it's `extern "C" ICorJitCompiler* __stdcall getJit()` from
[here](https://github.com/dotnet/coreclr/blob/8db8ec135fb5e797b6bdc6e68545bb4661f43881/src/inc/corjit.h#L241).

Before digging deep into the internals it makes sense to describe *the common hooking workflow*.

1. *Determine the function's signature.* Okay, I've got one, `getJit()` and it returns an implementation of `ICorJitCompiler`.
This interface has a great method which I definitely want to hook `compileMethod`.

```
// compileMethod is the main routine to ask the JIT Compiler to create native code for a method. The
    // method to be compiled is passed in the 'info' parameter, and the code:ICorJitInfo is used to allow the
    // JIT to resolve tokens, and make any other callbacks needed to create the code. nativeEntry, and
    // nativeSizeOfCode are just for convenience because the JIT asks the EE for the memory to emit code into
    // (see code:ICorJitInfo.allocMem), so really the EE already knows where the method starts and how big
    // it is (in fact, it could be in more than one chunk).
    // 
    // * In the 32 bit jit this is implemented by code:CILJit.compileMethod
    // * For the 64 bit jit this is implemented by code:PreJit.compileMethod
    // 
    // Note: Obfuscators that are hacking the JIT depend on this method having __stdcall calling convention
    virtual CorJitResult __stdcall compileMethod (
            ICorJitInfo                 *comp,               /* IN */
            struct CORINFO_METHOD_INFO  *info,               /* IN */
            unsigned /* code:CorJitFlag */   flags,          /* IN */
            BYTE                        **nativeEntry,       /* OUT */
            ULONG                       *nativeSizeOfCode    /* OUT */
            ) = 0;
```

to use it in the C# code necessary to wrap these functions like:

```
    [DllImport(
#if _TARGET_X64_
            "Clrjit.dll"
#else
            "Mscorjit.dll"
#endif
            , CallingConvention = CallingConvention.StdCall, SetLastError = true, EntryPoint = "getJit", BestFitMapping = true)]
        public static extern IntPtr GetJit();
```

as an entrypoint and to get the pointer to ICorJitCompiler

and the `compileMethod` function:

```
[UnmanagedFunctionPointer(CallingConvention.StdCall, SetLastError = true)]
public unsafe delegate CorJitResult CompileMethodDel(IntPtr thisPtr, [In] IntPtr corJitInfo, [In] CorInfo* methodInfo, CorJitFlag flags, [Out] IntPtr nativeEntry, [Out] IntPtr nativeSizeOfCode);
```

or in the C++ definition:

```
#define CompileFunctionSig CorJitResult(*compileMethod)(void*, struct CORINFO_METHOD_INFO*, unsigned flags, BYTE**, ULONG*)
```

2. After gathering the pointer to the function to re-write it, it's necessary to mark its memory as re-writable.
It can be done via [VirtualProtect](https://msdn.microsoft.com/ru-ru/library/windows/desktop/aa366898%28v=vs.85%29.aspx) win32 function.

To call it from the C# we need to interop wrap:

```
[DllImport("kernel32.dll", BestFitMapping = true, CallingConvention = CallingConvention.Winapi, SetLastError = true, ExactSpelling = true)]
public static extern bool VirtualProtect(IntPtr lpAddress, UInt32 dwSize, MemoryProtectionConstants flNewProtect, out UInt32 lpflOldProtect);
```

3. Precompile the delegate's function before hooking. It must be done to prevent SOE, the otherwise hooked compileMethod function will be
JITted with itself infinitely. `PrepareDelegate` [method](https://github.com/dotnet/coreclr/issues/15522) is doing this kind required the job.
It's jitting the provided method and keeps it alive from the GC via its fixed pointer.

4. Essentially replace the original function pointer in Vtable with the new one.

5. Return the previous `VitrualProtect` level for the used memory.

## Altogether ##

_the concept of described above_

```
[DllImport("kernel32.dll", BestFitMapping = true, CallingConvention = CallingConvention.Winapi, SetLastError = true, ExactSpelling = true)]
public static extern bool VirtualProtect(IntPtr lpAddress, UInt32 dwSize, MemoryProtectionConstants flNewProtect, out UInt32 lpflOldProtect);

[DllImport(
#if _TARGET_X64_
            "Clrjit.dll"
#else
            "Mscorjit.dll"
#endif
            , CallingConvention = CallingConvention.StdCall, SetLastError = true, EntryPoint = "getJit", BestFitMapping = true)]
public static extern IntPtr GetJit();

namespace ClrAnalyzer.Core.Hooks
{
    public unsafe class CompilerHook
    {
        public CompileMethodDel Compile = null;

        private IntPtr pJit;
        private IntPtr pVTable;
        private bool isHooked = false;
        private readonly CorJitCompilerNative compiler;
        private uint lpflOldProtect;

        public CompilerHook()
        {
            if (pJit == IntPtr.Zero) pJit = GetJit();
            Debug.Assert(pJit != null);
            compiler = Marshal.PtrToStructure<CorJitCompilerNative>(Marshal.ReadIntPtr(pJit));
            Debug.Assert(compiler.CompileMethod != null);
            pVTable = Marshal.ReadIntPtr(pJit);
            
            RuntimeHelpers.PrepareMethod(GetType().GetMethod("RemoveHook").MethodHandle);
            RuntimeHelpers.PrepareMethod(GetType().GetMethod("LockpVTable", System.Reflection.BindingFlags.Instance|System.Reflection.BindingFlags.NonPublic).MethodHandle);            
        }
        
        private bool UnlockpVTable()
        {
            if (!Win32MemoryUtils.VirtualProtect(pVTable, (uint)IntPtr.Size, Win32MemoryUtils.MemoryProtectionConstants.PAGE_EXECUTE_READWRITE, out lpflOldProtect))
            {
                Console.WriteLine(new Win32Exception(Marshal.GetLastWin32Error()).Message);
                return false;
            }
            return true;
        }

        private bool LockpVTable()
        {
            return Win32MemoryUtils.VirtualProtect(pVTable, (uint)IntPtr.Size, (Win32MemoryUtils.MemoryProtectionConstants)lpflOldProtect, out lpflOldProtect);
        }

        public bool Hook(CompileMethodDel hook)
        {
            if (!UnlockpVTable()) return false;

            Compile = compiler.CompileMethod;
            Debug.Assert(Compile != null);

            RuntimeHelpers.PrepareDelegate(hook);
            RuntimeHelpers.PrepareDelegate(Compile);

            Marshal.WriteIntPtr(pVTable, Marshal.GetFunctionPointerForDelegate(hook));

            return isHooked = LockpVTable();
        }

        public bool RemoveHook()
        {
            if (!isHooked) throw new InvalidOperationException("Impossible unhook not hooked compiler");
            if (!UnlockpVTable()) return false;

            Marshal.WriteIntPtr(pVTable, Marshal.GetFunctionPointerForDelegate(Compile));

            return LockpVTable();
        }
    }
}

```

## Internals ##

The 

```
virtual CorJitResult __stdcall compileMethod (
            ICorJitInfo                 *comp,               /* IN */
            struct CORINFO_METHOD_INFO  *info,               /* IN */
            unsigned /* code:CorJitFlag */   flags,          /* IN */
            BYTE                        **nativeEntry,       /* OUT */
            ULONG                       *nativeSizeOfCode    /* OUT */
            ) = 0;
            
            
[UnmanagedFunctionPointer(CallingConvention.StdCall, SetLastError = true)]
public unsafe delegate CorJitResult CompileMethodDel(IntPtr thisPtr, [In] IntPtr corJitInfo, [In] CorInfo* methodInfo, CorJitFlag flags, [Out] IntPtr nativeEntry, [Out] IntPtr nativeSizeOfCode);
```

contains the parameters I want to describe

1. `ICorJitInfo`. From the [code](https://github.com/dotnet/coreclr/blob/9f1dc4444478ccbac2476d53949c471583876ad7/src/inc/corjit.h#L318): "ICorJitInfo is the main interface that the JIT uses to call back to the EE and get information.".
As described, this interface has many useful functions to get the information of your code. Like `getJitFlags`
2. `CORINFO_METHOD_INFO`. The structure contains interesting and useful information like

* ILCode
* ILCodeSize
* maxStack
* ...

the C# wrapper is:

```
[StructLayout(layoutKind: LayoutKind.Sequential, Pack = 1, Size = 0x88)]
public unsafe struct CorInfo
{
    //ftn CORINFO_METHOD_HANDLE
    public IntPtr methodHandle;
    //scope CORINFO_MODULE_HANDLE
    public IntPtr moduleHandle;
    //BYTE*
    public IntPtr ILCode;
    public UInt32 ILCodeSize;
    public UInt16 maxStack;
    public UInt16 EHcount;
    //options CorInfoOptions
    public CorInfoOptions options;
    //regionKind CorInfoRegionKind
    public CorInfoRegionKind regionKind;
    //CORINFO_SIG_INFO
    public CorInfoSigInfo args;
    //CORINFO_SIG_INFO
    public CorInfoSigInfo locals;
}
```

3. `corJitFlag` is a bunch of flags that indicate broad attributes and parameters of the code for different compiler optimization approaches.
Its data we need, for sure [Here](https://github.com/GeorgePlotnikov/ClrAnalyzer/blob/master/ClrAnalyzer.Core/Compiler/CorJitFlags.cs) is a C# wrapper for using this.

4. `nativeEntry`

5. `nativeSizeOfCode`

The result of it is me [repository](https://github.com/GeorgePlotnikov/ClrAnalyzer) where you can find the working code. There are 2 versions of how to do it. First is the total C#
wrapping and interacting with interop interfaces. The second is the creation of a separate "C" library with only one entrypoint where all
magic should be done. The second approach is more clear because all datatypes and structures you can use as they are...almost :-).

For the convenient and safe allocation of unsafe structures there is the 3s steps way to do it:

```
public unsafe interface ICorJitCompiler
{
  CorJitResult CompileMethod(IntPtr thisPtr, [In] IntPtr corJitInfo, [In] CorInfo* methodInfo, CorJitFlag flags, [Out] IntPtr nativeEntry, [Out] IntPtr nativeSizeOfCode);
  void ProcessShutdownWork(IntPtr thisPtr, [In] IntPtr corStaticInfo);
}

public unsafe class CorJitCompiler
{
  public unsafe struct CorJitCompilerNative
  {
      public CompileMethodDel CompileMethod;
      public ProcessShutdownWorkDel ProcessShutdownWork;
      public isCacheCleanupRequiredDel isCacheCleanupRequired;
  }

  public static ICorJitCompiler GetCorJitCompilerInterface()
  {
      var pJit = GetJit();
      var nativeCompiler = Marshal.PtrToStructure<CorJitCompilerNative>(pJit);
      return new CorJitCompilerNativeWrapper(pJit, nativeCompiler.CompileMethod, nativeCompiler.ProcessShutdownWork);
  }

  private sealed class CorJitCompilerNativeWrapper : ICorJitCompiler
  {
    private IntPtr _pThis;
    private CompileMethodDel _compileMethod;
    private ProcessShutdownWorkDel _processShutdownWork;

    public CorJitCompilerNativeWrapper(IntPtr pThis, CompileMethodDel compileMethodDel, ProcessShutdownWorkDel processShutdownWork)
    {
        _pThis = pThis;
        _compileMethod = compileMethodDel;
        _processShutdownWork = processShutdownWork;
    }

    public CorJitResult CompileMethod(IntPtr thisPtr, [In] IntPtr corJitInfo, [In] CorInfo* methodInfo, CorJitFlag flags, [Out] IntPtr nativeEntry, [Out] IntPtr nativeSizeOfCode)
    {
        return _compileMethod(thisPtr, corJitInfo, methodInfo, flags, nativeEntry, nativeSizeOfCode);
    }

    public void ProcessShutdownWork(IntPtr thisPtr, [In] IntPtr corStaticInfo)
    {
        _processShutdownWork(thisPtr, corStaticInfo);
    }
}
```

To redirect the output it's necessary to source output target. For that, you can use the following function.

```
public const int STD_OUTPUT_HANDLE = -11;

[System.Runtime.InteropServices.DllImport("Kernel32.dll", SetLastError = true)]
public static extern int SetStdHandle(int device, IntPtr handle);

using (var fs = new FileStream("./Result.txt", FileMode.OpenOrCreate, FileAccess.ReadWrite))
using (var sw = new StreamWriter(fs))
{
    Win32IOUtils.SetStdHandle(Win32IOUtils.STD_OUTPUT_HANDLE, fs.SafeFileHandle.DangerousGetHandle());
}
```

## Result ##

As an example of what can be gathered with this approach is dump for the following function:

```
[MethodImpl(MethodImplOptions.AggressiveInlining)]
static int Calc(int x, int y)
{
    var r = Math.Asin((double)x);
    return (int)r * y;
}
```

the result:

```
native size of code: 12
IL code: 00000165EDB2D600
method attribs: 4010008
```

here we see the size of code, literally the code which is and attributes for this method. It's just a simple illustration how it works.

## Conclusion ## 

Feel free to contribute to [the ClrAnalyzer project](https://github.com/GeorgePlotnikov/ClrAnalyzer) and provide your PR and issues.
I will improve and provide the new features when I'm having the time.

## Links  ##

* [https://github.com/dotnet/coreclr](https://github.com/dotnet/coreclr)
* [https://github.com/dotnet/BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet)
* [https://github.com/dotnet/coreclr/blob/master/Documentation/building/viewing-jit-dumps.md](https://github.com/dotnet/coreclr/blob/master/Documentation/building/viewing-jit-dumps.md)
* [https://en.wikipedia.org/wiki/Hooking](https://en.wikipedia.org/wiki/Hooking)
* [https://msdn.microsoft.com/ru-ru/library/windows/desktop/aa366898%28v=vs.85%29.aspx](https://msdn.microsoft.com/ru-ru/library/windows/desktop/aa366898%28v=vs.85%29.aspx)
* [https://github.com/GeorgePlotnikov/ClrAnalyzer](https://github.com/GeorgePlotnikov/ClrAnalyzer)






