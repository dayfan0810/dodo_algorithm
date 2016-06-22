

# ART Java Method Hook 浅析



最近使用别人的hook技术完成了一个hotfix的框架，之前只对Dalvik的hook有所了解，对art的执行过程不是很熟悉，所以使用GDB跟踪了一下ART启动执行过程。这里整理一下。



##  概述


方法hook技术在Dalvik虚拟机上已经有多种方式实现，成熟的框架和工具也有不少比如`Xposed`,`Cydia Substrate`。在Android4.4中，Google引入了ART虚拟机来应对Dalvik的所面对的性能问题，而针对Art的hook技术也开始发展起来，这里简单介绍一些我使用的一种hook框架中所采用的方法。


首先要明确目的，本篇文章研究的是Java Method的hook，基本原理就是改变代码的执行入口。

本文分析的是4.4的代码。

##  ART 方法执行过程

### OAT File

在讨论这个过程的时候我们先来看一下一个oat文件的格式，祭出这张图


![enter description here][1]


这里简单描述一下Oat文件格式，oat文件本质是一个ELF的文件，具有ELF文件的一般结构，然后在其基础上定义了`oatdata`和`oatexec`两个特殊的数据区域，`oatdata`存储应用原有Dex的相关信息，`oatexec`段是代码区域，用于存储Dalvik字节码预编译生成的本地代码。图中动态符号段`symbol table`中的`oatdata`,`oatexec`,`oatlastword`三个符号分别用于`oatdata`段的标志、`oatexec`段标志、`oatexec`段结束的标志。`oatdata`理论上包含完整的dex文件，并且包含dex类中方法和本地方法`native code` 的映射关系，Art用来查找Dex所对应的本地方法。

在`oatheader`段说明当前oat的一些信息，比如文件标识，版本信息，校验值，指令集，本地代码偏移地址和dex文件个数等，下图是我用oatdump获取的头文件信息，用于参考

![oatheader][2]

其实我们主要关注的是`OatClass`中的方法指针对应的`code_offset`，祭出老罗的一张图就知道这个过程是如何进行的了


![enter description here][3]

在Art执行过程中我们其实操作的是`ArtMethod`，映射到Java里就是我们常见Method，Art通过`OatFile::OatMethod::LinkMethod` 方法将`ArtMethod`和`OatMethod`进行对应，引用老罗文章里的一段话"就是通过`OatMethod`类的成员函数`GetCode`获得`OatMethod`结构体中的`code_offset_`字段，并且通过调用`ArtMethod`类的成员函数`SetEntryPointFromCompiledCode`设置到参`数method`描述的`ArtMethod`对象中去"

![enter description here][4]

这里就是关键所在。


### 方法执行入口
下面我们进入主题，先分析一下整个方法执行的过程。其实在此之前还有很多前置工作要做，这里就不具体讨论OAT文件加载以及OatClass查找和OatMethod查找的过程了，这里只分析与Hook相关的内容。

在此之前我们先看一下ART执行一个method所使用的入口，然后通过这个入口分析code是如何被执行的。

这个过程所涉及的代码在`art/runtime/class_linker.cc`中的`LinkCode`方法中，这里贴出代码方便下面的分析

```cpp

tatic void LinkCode(SirtRef<mirror::ArtMethod>& method, const OatFile::OatClass* oat_class,
                     uint32_t method_index)
    SHARED_LOCKS_REQUIRED(Locks::mutator_lock_) {
  // Method shouldn't have already been linked.
  DCHECK(method->GetEntryPointFromCompiledCode() == NULL);
  // Every kind of method should at least get an invoke stub from the oat_method.
  // non-abstract methods also get their code pointers.
  const OatFile::OatMethod oat_method = oat_class->GetOatMethod(method_index);
  oat_method.LinkMethod(method.get());

  // Install entry point from interpreter.
  Runtime* runtime = Runtime::Current();
  bool enter_interpreter = NeedsInterpreter(method.get(), method->GetEntryPointFromCompiledCode());
  if (enter_interpreter) {
    method->SetEntryPointFromInterpreter(interpreter::artInterpreterToInterpreterBridge);
  } else {
    method->SetEntryPointFromInterpreter(artInterpreterToCompiledCodeBridge);
  }

  if (method->IsAbstract()) {
    method->SetEntryPointFromCompiledCode(GetCompiledCodeToInterpreterBridge());
    return;
  }

  if (method->IsStatic() && !method->IsConstructor()) {
    // For static methods excluding the class initializer, install the trampoline.
    // It will be replaced by the proper entry point by ClassLinker::FixupStaticTrampolines
    // after initializing class (see ClassLinker::InitializeClass method).
    method->SetEntryPointFromCompiledCode(GetResolutionTrampoline(runtime->GetClassLinker()));
  } else if (enter_interpreter) {
    // Set entry point from compiled code if there's no code or in interpreter only mode.
    method->SetEntryPointFromCompiledCode(GetCompiledCodeToInterpreterBridge());
  }

  if (method->IsNative()) {
    // Unregistering restores the dlsym lookup stub.
    method->UnregisterNative(Thread::Current());
  }

  // Allow instrumentation its chance to hijack code.
  runtime->GetInstrumentation()->UpdateMethodsCode(method.get(),
                                                   method->GetEntryPointFromCompiledCode());
}


```


相对来说ART执行过程要比Dalvik要复杂的多，对于方法执行的入口大概有以下四种方式：

1. 存在本地代码的方法
![enter description here][5]

对于存在nativecode的方法入口设置为`artInterpreterToCompiledCodeBridge`,从本地代码进入的入口设置为方法对应的本地代码偏移地址，也就是之前说的`code_offset`,art会执行到`artInterpreterToCompiledCodeBridge`函数首先从解释器的`shadow_frame`栈帧中获取目标方法的ArtMethod对象，ArtMethod里保存这本地方法的入口也就是前面所设置的`code_offset`值，然后调用`Invoke`方法来执行本地方法代码。
具体本地方法如何执行的将在下面单独分析。

2. 没有对应本地代码的方法
这种情况指的是在方法的`CODE`段是空的的情况。如下图
![enter description here][6]
对于没有对应本地代码的方法只能通过解释器执行，此时将 `artInterpreterToInterpreterBridge`(art/runtime/interpreter/interpreter.cc)设置为解释器的入口，并且将`GetCompiledCodeToInterpreterBridge`设置为本地代码入口，`artInterpreterToInterpreterBridge`是从解释器到解释器的跳转代码，只需要找到目标方法的字节码然后解释执行，而`GetCompiledCodeToInterpreterBridge`是从本地代码进入解释器的入口。

3. 非构造方法的静态方法
本地代码的入口是`art_quick_resolution_trampoline`,也就是`GetResolutionTrampoline`的最终返回值。
从注释上看是静态方法在对应的类没有初始化的时候，该方法会初始化该类，然后再调用该方法的本地方法。
(我还没分析清楚这里=_=)


4. JNI方法
暂时不分析，只知道入口是名为`art_jni_dlsym_lookup_stub`(`runtime/arch/arm/jni_entrypoints_arm.S`)的汇编函数，调用`artFindNativeMethod`查找本地函数，查找到后就执行，大概这样

```x86asm
 blx    artFindNativeMethod
    mov    r12, r0                        @ 将执行结果赋值给r12寄存器
    add    sp, #12                        @ restore stack pointer
.....
    bx     r12                            @ 如果不是空的，跳转到所指地址执行
```


### Native Code执行过程

这个过程通过`Method::Invoke`方法调起

```cpp
    if (GetEntryPointFromCompiledCode() != NULL) {
      if (kLogInvocationStartAndReturn) {
        LOG(INFO) << StringPrintf("Invoking '%s' code=%p", PrettyMethod(this).c_str(), GetEntryPointFromCompiledCode());
      }
#ifdef ART_USE_PORTABLE_COMPILER
      (*art_portable_invoke_stub)(this, args, args_size, self, result, result_type);
#else
      (*art_quick_invoke_stub)(this, args, args_size, self, result, result_type);
#endif
      if (UNLIKELY(reinterpret_cast<int32_t>(self->GetException(NULL)) == -1)) {
        // Unusual case where we were running LLVM generated code and an
        // exception was thrown to force the activations to be removed from the
        // stack. Continue execution in the interpreter.
        self->ClearException();
        ShadowFrame* shadow_frame = self->GetAndClearDeoptimizationShadowFrame(result);
        self->SetTopOfStack(NULL, 0);
        self->SetTopOfShadowStack(shadow_frame);
        interpreter::EnterInterpreterFromDeoptimize(self, shadow_frame, result);
      }
      if (kLogInvocationStartAndReturn) {
        LOG(INFO) << StringPrintf("Returned '%s' code=%p", PrettyMethod(this).c_str(), GetEntryPointFromCompiledCode());
      }
    } 

```


这个执行过程根据平台(arm,x86,mips)不同所实现的方法都不一样，这里只分析ARM架构的实现`art_quick_invoke_stub``(/art/runtime/arch/arm/quick_entrypoints_arm.S)`。

```x86asm

   /*
     * Quick invocation stub.
     * On entry: 参数对应上面所传递的值
     *   r0 = 方法指针
     *   r1 = 参数数组指针
     *   r2 = 参数数组大小
     *   r3 = 当前线程指针
     *   [sp] = JValue* result 返回值结果指针
     *   [sp + 4] = result type char 返回值类型
     */
ENTRY art_quick_invoke_stub
    push   {r0, r4, r5, r9, r11, lr}       @ spill regs 
    .save  {r0, r4, r5, r9, r11, lr}
    .pad #24
    .cfi_adjust_cfa_offset 24
    .cfi_rel_offset r0, 0
    .cfi_rel_offset r4, 4
    .cfi_rel_offset r5, 8
    .cfi_rel_offset r9, 12
    .cfi_rel_offset r11, 16
    .cfi_rel_offset lr, 20
    mov    r11, sp                         @ 保存sp到r11
    .cfi_def_cfa_register r11
    mov    r9, r3                          @ 保存当前线程指针到r9
    mov    r4, #SUSPEND_CHECK_INTERVAL     @ reset r4 to suspend check interval
    add    r5, r2, #16                     @ 给参数分配空间
    and    r5, #0xFFFFFFF0                 @ 对齐16个字节
    sub    sp, r5                          @ reserve stack space for argument array
    add    r0, sp, #4                      @ pass stack pointer + method ptr as dest for memcpy
    bl     memcpy                          @ memcpy (dest, src, bytes) 
    ldr    r0, [r11]                       @ r0=方法指针
    ldr    r1, [sp, #4]                    @ arg0
    ldr    r2, [sp, #8]                    @ arg1
    ldr    r3, [sp, #12]                   @ arg2
    mov    ip, #0                          @ set ip to 0
    str    ip, [sp]                        @ store NULL for method* at bottom of frame
    ldr    ip, [r0, #METHOD_CODE_OFFSET]   @ 方法指针+METHOD_CODE_OFFSET 注意这个值在不同版本上不同
    blx    ip                              @ 调用方法
    mov    sp, r11                         @ restore the stack pointer
    ldr    ip, [sp, #24]                   @ load the result pointer
    strd   r0, [ip]                        @ 将返回值写入r0 r1
    pop    {r0, r4, r5, r9, r11, lr}       @ restore spill regs
    .cfi_adjust_cfa_offset -24
    bx     lr
END art_quick_invoke_stub

```

上面的每行注释已经将过程写的很清楚了，这里通过栈帧分析一下这个过程。


![enter description here][7]

代码到 `bl memcpy`处，所做的操作基本等于如下操作

```cpp
r5 = (r2 + 16) & 0xFFFFFF0;
sp = r5;
r0 = sp + 4;
memcpy(r0, r1, r2);

```

然后下面的操作基本上就是将`ip`设置到本地方法中然后`blx`一下，最后保存返回值

对ARM的汇编理解不是很到位，如果有问题请指出。


这里需要说明一下`METHOD_CODE_OFFSET`这个值，在不同的Android平台下这个值也是不同的

在4.4中是40

在5.0中是44

在6.0中是36

在不同版本中这段执行代码也是不同的，6.0的代码相对要复杂，而且名称变为`art_quick_invoke_stub_internal` 以后有时间在重新分析一下6.0的执行逻辑。


### Hook实现


从上面的分析看，如果只考虑情况1的条件下，我们只需要重置`code_offset`到我们hook方法的`code_offset`中即可。

首先我们要获取到被Hook方法对应的ArtMethod所对应的指针，也就是我们要像虚拟机查找方法那样拿到一个对象。

这里用的方法很简单，用过JNIEnv的FromReflectedMethod 获取当前方法指针。

```cpp

jlong getMethodAddress(JNIEnv *env, jclass clazz, jobject method) {
    return (jlong) env->FromReflectedMethod(method);
}


```

然后我们就可以通过这个地址还原出整个ArtMethod的对象内容

以4.4的结构为例
```java

  @StructMapping(offset = 0)
    private StructMember klass_;

    @StructMapping(offset = 4)
    private StructMember monitor_;

    @StructMapping(offset = 8)
    private StructMember declaring_class_;

    @StructMapping(offset = 12)
    private StructMember dex_cache_initialized_static_storage_;

    @StructMapping(offset = 16)
    private StructMember dex_cache_resolved_methods_;

    @StructMapping(offset = 20)
    private StructMember dex_cache_resolved_types_;

    @StructMapping(offset = 24)
    private StructMember dex_cache_strings_;

    @StructMapping(offset = 28)
    private StructMember access_flags_;

    @StructMapping(offset = 32)
    private StructMember code_item_offset_;

    @StructMapping(offset = 36)
    private StructMember core_spill_mask_;

    @StructMapping(offset = 40)
    private StructMember entry_point_from_compiled_code_;

    @StructMapping(offset = 44)
    private StructMember entry_point_from_interpreter_;

    @StructMapping(offset = 48)
    private StructMember fp_spill_mask_;

    @StructMapping(offset = 52)
    private StructMember frame_size_in_bytes_;

    @StructMapping(offset = 56)
    private StructMember gc_map_;

    @StructMapping(offset = 60)
    private StructMember mapping_table_;

    @StructMapping(offset = 64)
    private StructMember method_dex_index_;

    @StructMapping(offset = 68)
    private StructMember method_index_;

    @StructMapping(offset = 72)
    private StructMember native_method_;

    @StructMapping(offset = 76)
    private StructMember vmap_table_;


```

通过C语言的指针操作按照该`内存地址+偏移量+长度`的方法将指定区域内容从内存中读取出来，然后通过JNI返回到Java对象中


```cpp
//src是要读取的源地址，length是长度

jbyteArray android_memget(JNIEnv *env, jclass _cls, jlong src, jint length) {
    jbyteArray dest = env->NewByteArray(length);
    if (dest == NULL) {
        return NULL;
    }
    unsigned char *destPnt = (unsigned char *) env->GetByteArrayElements(dest, 0);
    unsigned char *srcPnt = (unsigned char *) src;
    for (int i = 0; i < length; ++i) {
        destPnt[i] = srcPnt[i];
    }
    env->ReleaseByteArrayElements(dest, (jbyte *) destPnt, 0);
    return dest;
}


```


构造好Hook method和原始method的内容后，将`entry_point_from_compiled_code_`的值进行替换即可完成Hook

其实做到这里我们只能hook住同方法签名的方法，相对于Hotfix的功能来说已经足够了。

如果要Hook的方法和被Hook的方法签名不一致那么在大部分手机上就会出现问题。


最后也是重要的一步就是我们需要在调用完Hook代码后，提供调用原来的方法的功能。

这里采用的方式是在替换指针之前将原来的方法保存一份


```java
 public ArtMethod backup() {
            Class<?> abstractMethodClass = Class.forName("java.lang.reflect.AbstractMethod");
            if (Build.VERSION.SDK_INT < 23) {
                Class<?> artMethodClass = Class.forName("java.lang.reflect.ArtMethod");
                //Get the original artMethod field
                Field artMethodField = abstractMethodClass.getDeclaredField("artMethod");
                if (!artMethodField.isAccessible()) {
                    artMethodField.setAccessible(true);
                }
                Object srcArtMethod = artMethodField.get(method);

                Constructor<?> constructor = artMethodClass.getDeclaredConstructor();
                constructor.setAccessible(true);
                Object destArtMethod = constructor.newInstance();

                //Fill the fields to the new method we created
                for (Field field : artMethodClass.getDeclaredFields()) {
                    if (!field.isAccessible()) {
                        field.setAccessible(true);
                    }
                    field.set(destArtMethod, field.get(srcArtMethod));
                }
                Method newMethod = Method.class.getConstructor(artMethodClass).newInstance(destArtMethod);
                newMethod.setAccessible(true);
                ArtMethod artMethod = ArtMethod.of(newMethod);
                artMethod.setEntryPointFromInterpreter(getEntryPointFromInterpreter());
                artMethod.setEntryPointFromJni(getEntryPointFromJni());
                artMethod.setEntryPointFromQuickCompiledCode(getEntryPointFromQuickCompiledCode());
                //NOTICE: The clone method must set the access flags to private.
                int accessFlags = getAccessFlags();
                accessFlags &= ~Modifier.PUBLIC;
                accessFlags |= Modifier.PRIVATE;
                artMethod.setAccessFlags(accessFlags);
                return artMethod;
}
```

当用户需要调用原来的方法时候只需要通过反射invoke原来的方法即可

```java
    private <T> T callSuperArt(Method method, Object who, Object... args) throws Throwable {
        return (T) method.invoke(who, args);
    }
```








## 测试







## ART VM启动过程

```bash
(gdb) p *argv@argc
$9 = {0xbfcebc00 "dalvikvm", 0xbfcebc09 "-cp", 0xbfcebc0d "/mnt/foo.jar", 
  0xbfcebc1a "Foo"}

执行到277行的变量值

(gdb) p init_args
$13 = {version = 65542, nOptions = 2, options = 0xb73cc020, 
  ignoreUnrecognized = 0 '\000'}
(gdb) p *init_args.options
$17 = {optionString = 0xbfcebc09 "-cp", extraInfo = 0x0}


进入到CreateJavaVM流程中

(gdb) s
JNI_CreateJavaVM (p_vm=0xbfceb0f0, p_env=0x2, vm_args=0x801)
    at libnativehelper/JniInvocation.cpp:175
175	  return JniInvocation::GetJniInvocation().JNI_CreateJavaVM(p_vm, p_env, vm_args);

(gdb) s
JNI_CreateJavaVM (this=0xbfceb0f0, p_vm=<optimized out>, 
    p_env=<optimized out>, vm_args=0xbfceb100)
    at libnativehelper/JniInvocation.cpp:146
146	  return JNI_CreateJavaVM_(p_vm, p_env, vm_args);

调用JNI_CreateJavaVM_ 正式进入虚拟机创建流程

(gdb) s
art::JNI_CreateJavaVM (p_vm=0xbfceb0e8, p_env=0xbfceb0ec, vm_args=0xbfceb100)
    at art/runtime/java_vm_ext.cc:791
791	extern "C" jint JNI_CreateJavaVM(JavaVM** p_vm, JNIEnv** p_env, void* vm_args) {

进入Runtime::Create流程

Breakpoint 2, art::JNI_CreateJavaVM (p_vm=0xbfc6e9d8, p_env=0xbfc6e9dc, 
    vm_args=0xbfc6e9f0) at art/runtime/java_vm_ext.cc:805
805	  if (!Runtime::Create(options, ignore_unrecognized)) {

art::Runtime::Create (options=..., ignore_unrecognized=false)
    at art/runtime/runtime.cc:410
410	bool Runtime::Create(const RuntimeOptions& options, bool ignore_unrecognized) {


进入Runtime::Init流程


art::Runtime::Init (this=this@entry=0xb7493000, raw_options=..., 
    ignore_unrecognized=ignore_unrecognized@entry=false)
    at art/runtime/runtime.cc:782


art::Thread::Startup () at art/runtime/thread.cc:1241
1241	void Thread::Startup() {


art::Thread::Attach (thread_name=thread_name@entry=0xb739739b "main", 
    as_daemon=as_daemon@entry=false, thread_group=thread_group@entry=0x0, 
    create_peer=create_peer@entry=false) at art/runtime/thread.cc:514
    
执行完Create返回到JNI_CreateJavaVM流程中
art::JNI_CreateJavaVM (p_vm=0xbfe42958, p_env=0xbfe4295c, vm_args=0xbfe42970)
    at art/runtime/java_vm_ext.cc:809
809	  Runtime* runtime = Runtime::Current();


接着执行 Runtime::Start流程中

art::Runtime::Start (this=this@entry=0xb7453000) at art/runtime/runtime.cc:485
485	bool Runtime::Start() {



执行完成后返回 

dalvikvm (argv=0xbfe42a18, argc=3) at art/dalvikvm/dalvikvm.cc:185
185	  if (arg_idx == argc) {


进入到InvokeMain流程中


InvokeMain (argv=0xbfe42a20, env=0xb6be2000) at art/dalvikvm/dalvikvm.cc:63
63	  ScopedLocalRef<jobjectArray> args(env, toStringArray(env, argv + 1));


_JNIEnv::CallStaticVoidMethod (this=0xb6be2000, clazz=0x5, methodID=0xb34b5098)
    at libnativehelper/include/nativehelper/jni.h:776
776	    void CallStaticVoidMethod(jclass clazz, jmethodID methodID, ...)


art::JNI::CallStaticVoidMethodV (env=0xb6be2000, mid=0xb34b5098, 
    args=0xbfc6331c "\001") at art/runtime/jni_internal.cc:1621
1621	  static void CallStaticVoidMethodV(JNIEnv* env, jclass, jmethodID mid, va_list args) {

art::InvokeWithVarArgs (soa=..., obj=obj@entry=0x0, mid=mid@entry=0xb34b5098, 
    args=args@entry=0xbfc6331c "\001") at art/runtime/reflection.cc:443
443	  if (UNLIKELY(__builtin_frame_address(0) < soa.Self()->GetStackEnd())) {


Breakpoint 1, InvokeWithArgArray (shorty=0x71103a40 "VL", result=0xbfeb7c74, 
    arg_array=0xbfeb7c7c, method=0x70b8b520, soa=...)
    at art/runtime/reflection.cc:434
    
art::ArtMethod::Invoke (this=this@entry=0x70b8b520, self=0xb6bcd000, 
    args=args@entry=0xbfeb7c8c, args_size=4, result=result@entry=0xbfeb7c74, 
    shorty=shorty@entry=0x71103a40 "VL") at art/runtime/art_method.cc:369
369	                       const char* shorty) {

```
