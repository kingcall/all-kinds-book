## 概述

之所以想写这篇文章，是因为YGC过程对我们来说太过于黑盒，如果对YGC过程不是很熟悉，这类问题基本很难定位，我们就算开了GC日志，也最多能看到类似下面的日志

```
[GC (Allocation Failure) [ParNew: 91807K->10240K(92160K), 0.0538384 secs] 91807K->21262K(2086912K), 0.0538680 secs] [Times: user=0.16 sys=0.06, real=0.06 secs]  
```

只知道耗了多长时间，但是具体耗在了哪个阶段，是基本看不出来的，所以要么就是靠经验来定位，要么就是对代码相当熟悉，脑袋里过一遍整个过程，看哪个阶段最可能，今天要讲的这个大家可以当做今后排查这类问题的一个经验来使，这个当然不是唯一导致YGC过长的一个原因。

## Demo

先上一个demo，来描述下问题的情况，代码很简单，就是不断创建UUID，其实就是一个字符串，并将这个字符串调用下intern方法

```
import java.util.UUID;

public class StringTableTest {
  public static void main(String args[]) {
      for (int i = 0; i < 10000000; i++) {
          uuid();
      }
  }

  public static void uuid() {
      UUID.randomUUID().toString().intern();
  }
}

```

我们使用的JVM参数如下：

```
-XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -Xmx2G -Xms2G -Xmn100M
```

这里特意将新生代设置比较小，老生代设置比较大，让代码在执行过程中更容易突出问题来，大量做ygc，期间不做CMS GC，于是我们得到的输出结果类似下面的

```
[GC (Allocation Failure) [ParNew: 81920K->9887K(92160K), 0.0096027 secs] 81920K->9887K(2086912K), 0.0096428 secs] [Times: user=0.06 sys=0.01, real=0.01 secs] 
[GC (Allocation Failure) [ParNew: 91807K->10240K(92160K), 0.0538384 secs] 91807K->21262K(2086912K), 0.0538680 secs] [Times: user=0.16 sys=0.06, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0190535 secs] 103182K->32655K(2086912K), 0.0190965 secs] [Times: user=0.12 sys=0.01, real=0.02 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0198259 secs] 114575K->44124K(2086912K), 0.0198558 secs] [Times: user=0.13 sys=0.01, real=0.02 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0213643 secs] 126044K->55592K(2086912K), 0.0213930 secs] [Times: user=0.14 sys=0.01, real=0.02 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0234291 secs] 137512K->67061K(2086912K), 0.0234625 secs] [Times: user=0.16 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10238K(92160K), 0.0243691 secs] 148981K->78548K(2086912K), 0.0244041 secs] [Times: user=0.15 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0235310 secs] 160468K->89998K(2086912K), 0.0235587 secs] [Times: user=0.17 sys=0.01, real=0.02 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0255960 secs] 171918K->101466K(2086912K), 0.0256264 secs] [Times: user=0.18 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10238K(92160K), 0.0287876 secs] 183386K->113770K(2086912K), 0.0288188 secs] [Times: user=0.20 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0298405 secs] 195690K->125267K(2086912K), 0.0298823 secs] [Times: user=0.20 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0310182 secs] 207187K->136742K(2086912K), 0.0311156 secs] [Times: user=0.22 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0321647 secs] 218662K->148210K(2086912K), 0.0321938 secs] [Times: user=0.22 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0338090 secs] 230130K->159686K(2086912K), 0.0338446 secs] [Times: user=0.24 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0326612 secs] 241606K->171159K(2086912K), 0.0326912 secs] [Times: user=0.23 sys=0.01, real=0.03 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0350578 secs] 253079K->182627K(2086912K), 0.0351077 secs] [Times: user=0.26 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0346946 secs] 264547K->194096K(2086912K), 0.0347274 secs] [Times: user=0.25 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0384091 secs] 276016K->205567K(2086912K), 0.0384401 secs] [Times: user=0.27 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0394017 secs] 287487K->217035K(2086912K), 0.0394312 secs] [Times: user=0.29 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0411447 secs] 298955K->228504K(2086912K), 0.0411748 secs] [Times: user=0.30 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0393449 secs] 310424K->239972K(2086912K), 0.0393743 secs] [Times: user=0.29 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0444541 secs] 321892K->251441K(2086912K), 0.0444887 secs] [Times: user=0.32 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0449196 secs] 333361K->262910K(2086912K), 0.0449557 secs] [Times: user=0.33 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0497517 secs] 344830K->274382K(2086912K), 0.0497946 secs] [Times: user=0.34 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0475741 secs] 356302K->285851K(2086912K), 0.0476130 secs] [Times: user=0.35 sys=0.01, real=0.04 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0461098 secs] 367771K->297320K(2086912K), 0.0461421 secs] [Times: user=0.34 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0508071 secs] 379240K->308788K(2086912K), 0.0508428 secs] [Times: user=0.38 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0494472 secs] 390708K->320257K(2086912K), 0.0494938 secs] [Times: user=0.36 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0531527 secs] 402177K->331725K(2086912K), 0.0531845 secs] [Times: user=0.39 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0543701 secs] 413645K->343194K(2086912K), 0.0544025 secs] [Times: user=0.41 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0528003 secs] 425114K->354663K(2086912K), 0.0528283 secs] [Times: user=0.39 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0565080 secs] 436583K->366131K(2086912K), 0.0565394 secs] [Times: user=0.42 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0597181 secs] 448051K->377600K(2086912K), 0.0597653 secs] [Times: user=0.44 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0606671 secs] 459520K->389068K(2086912K), 0.0607423 secs] [Times: user=0.46 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0590389 secs] 470988K->400539K(2086912K), 0.0590679 secs] [Times: user=0.43 sys=0.01, real=0.05 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0600462 secs] 482459K->412008K(2086912K), 0.0600757 secs] [Times: user=0.44 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0608772 secs] 493928K->423476K(2086912K), 0.0609170 secs] [Times: user=0.45 sys=0.01, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0622107 secs] 505396K->434945K(2086912K), 0.0622391 secs] [Times: user=0.46 sys=0.00, real=0.06 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0626555 secs] 516865K->446413K(2086912K), 0.0626872 secs] [Times: user=0.47 sys=0.01, real=0.07 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0647713 secs] 528333K->457882K(2086912K), 0.0648013 secs] [Times: user=0.47 sys=0.00, real=0.07 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0747113 secs] 539802K->469353K(2086912K), 0.0747446 secs] [Times: user=0.51 sys=0.01, real=0.07 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0727498 secs] 551273K->480832K(2086912K), 0.0727899 secs] [Times: user=0.52 sys=0.01, real=0.07 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0734084 secs] 562752K->492300K(2086912K), 0.0734402 secs] [Times: user=0.54 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0766368 secs] 574220K->503769K(2086912K), 0.0766673 secs] [Times: user=0.55 sys=0.01, real=0.07 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0778940 secs] 585689K->515237K(2086912K), 0.0779250 secs] [Times: user=0.56 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0815513 secs] 597157K->526712K(2086912K), 0.0815824 secs] [Times: user=0.57 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0812080 secs] 608632K->538181K(2086912K), 0.0812406 secs] [Times: user=0.58 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0818790 secs] 620101K->549651K(2086912K), 0.0819155 secs] [Times: user=0.60 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0840677 secs] 631571K->561122K(2086912K), 0.0841000 secs] [Times: user=0.61 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0842462 secs] 643042K->572593K(2086912K), 0.0842785 secs] [Times: user=0.61 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0875011 secs] 654513K->584076K(2086912K), 0.0875416 secs] [Times: user=0.62 sys=0.01, real=0.08 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0887645 secs] 665996K->595532K(2086912K), 0.0887956 secs] [Times: user=0.64 sys=0.01, real=0.09 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10240K(92160K), 0.0921844 secs] 677452K->607001K(2086912K), 0.0922153 secs] [Times: user=0.65 sys=0.01, real=0.09 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10238K(92160K), 0.0930053 secs] 688921K->618471K(2086912K), 0.0930380 secs] [Times: user=0.67 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0955379 secs] 700391K->629942K(2086912K), 0.0955873 secs] [Times: user=0.69 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0919127 secs] 711862K->641411K(2086912K), 0.0919528 secs] [Times: user=0.68 sys=0.01, real=0.09 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0942291 secs] 723331K->652879K(2086912K), 0.0942611 secs] [Times: user=0.70 sys=0.00, real=0.09 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.0951904 secs] 734799K->664348K(2086912K), 0.0952265 secs] [Times: user=0.71 sys=0.00, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.0963585 secs] 746268K->675816K(2086912K), 0.0963909 secs] [Times: user=0.72 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0969504 secs] 757736K->687285K(2086912K), 0.0969843 secs] [Times: user=0.72 sys=0.01, real=0.09 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.0999066 secs] 769205K->698753K(2086912K), 0.0999376 secs] [Times: user=0.75 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10240K(92160K), 0.0998371 secs] 780673K->710222K(2086912K), 0.0998835 secs] [Times: user=0.75 sys=0.00, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92160K->10239K(92160K), 0.1024616 secs] 792142K->721691K(2086912K), 0.1024927 secs] [Times: user=0.77 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92159K->10238K(92160K), 0.1015041 secs] 803611K->733159K(2086912K), 0.1015378 secs] [Times: user=0.76 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.1058214 secs] 815079K->744628K(2086912K), 0.1058532 secs] [Times: user=0.80 sys=0.01, real=0.10 secs] 
[GC (Allocation Failure) [ParNew: 92158K->10238K(92160K), 0.1061273 secs] 826548K->756096K(2086912K), 0.1061620 secs] [Times: user=0.80 sys=0.01, real=0.10 secs] 

```

有没有发现YGC不断发生，并且发生的时间不断在增长，从10ms慢慢增长到了100ms，甚至还会继续涨下去

## String.intern方法

从上面的demo我们能挖掘到的可能就是intern这个方法了，那我们先来了解下intern方法的实现，这是String提供的一个方法，jvm提供这个方法的目的是希望对于某个同名字符串使用非常多的场景，在jvm里只保留一份，比如我们不断new String(“a”)，其实在java heap里会有多个String的对象，并且值都是a，如果我们只希望内存里只保留一个a，或者希望我接下来用到的地方都返回同一个a，那就可以用String.intern这个方法了，用法如下

```
String a = "a".intern();
...
String b = a.intern();
```

这样b和a都是指向内存里的同一个String对象，那JVM里到底怎么做到的呢？

我们看到intern这个方法其实是一个native方法，具体对应到JVM里的逻辑是

```
oop StringTable::intern(oop string, TRAPS)
{
  if (string == NULL) return NULL;
  ResourceMark rm(THREAD);
  int length;
  Handle h_string (THREAD, string);
  jchar* chars = java_lang_String::as_unicode_string(string, length);
  oop result = intern(h_string, chars, length, CHECK_NULL);
  return result;
}

oop StringTable::intern(Handle string_or_null, jchar* name,
                        int len, TRAPS) {
  unsigned int hashValue = hash_string(name, len);
  int index = the_table()->hash_to_index(hashValue);
  oop found_string = the_table()->lookup(index, name, len, hashValue);

  // Found
  if (found_string != NULL) return found_string;

  debug_only(StableMemoryChecker smc(name, len * sizeof(name[0])));
  assert(!Universe::heap()->is_in_reserved(name) || GC_locker::is_active(),
         "proposed name of symbol must be stable");

  Handle string;
  // try to reuse the string if possible
  if (!string_or_null.is_null() && (!JavaObjectsInPerm || string_or_null()->is_perm())) {
    string = string_or_null;
  } else {
    string = java_lang_String::create_tenured_from_unicode(name, len, CHECK_NULL);
  }

  // Grab the StringTable_lock before getting the_table() because it could
  // change at safepoint.
  MutexLocker ml(StringTable_lock, THREAD);

  // Otherwise, add to symbol to table
  return the_table()->basic_add(index, string, name, len,
                                hashValue, CHECK_NULL);
}

```

也就是说是其实在JVM里存在一个叫做StringTable的数据结构，这个数据结构是一个Hashtable，在我们调用String.intern的时候其实就是先去这个StringTable里查找是否存在一个同名的项，如果存在就直接返回对应的对象，否则就往这个table里插入一项，指向这个String对象，那么再下次通过intern再来访问同名的String对象的时候，就会返回上次插入的这一项指向的String对象

至此大家应该知道其原理了，另外我这里还想说个题外话，记得几年前tomcat里爆发的一个HashMap导致的hash碰撞的问题，这里其实也是一个Hashtable，所以也还是存在类似的风险，不过JVM里提供一个参数专门来控制这个table的size，`-XX:StringTableSize`，这个参数的默认值如下

```
product(uintx, StringTableSize, NOT_LP64(1009) LP64_ONLY(60013),          \
          "Number of buckets in the interned String table")        
```

另外JVM还会根据hash碰撞的情况来决定是否做rehash，比如你从这个StringTable里查找某个字符串是否存在，如果对其对应的桶挨个遍历，超过了100个还是没有找到对应的同名的项，那就会设置一个flag，让下次进入到safepoint的时候做一次rehash动作，尽量减少碰撞的发生，但是当恶化到一定程度的时候，其实也没啥办法啦，因为你的数据量实在太大，桶子数就那么多，那每个桶再怎么均匀也会带着一个很长的链表，所以此时我们通过修改上面的StringTableSize将桶数变大，可能会一定程度上缓解，但是如果是java代码的问题导致泄露，那就只能定位到具体的代码进行改造了。

## StringTable为什么会影响YGC

YGC的过程我不打算再这篇文章里细说，因为我希望尽量保持每篇文章的内容不过于臃肿，有机会可以单独写篇文章来介绍，我这里将列出ygc过程里StringTable这块的具体代码

```
 if (!_process_strong_tasks->is_task_claimed(SH_PS_StringTable_oops_do)) {
    if (so & SO_Strings || (!collecting_perm_gen && !JavaObjectsInPerm)) {
      StringTable::oops_do(roots);
    }
    if (JavaObjectsInPerm) {
      // Verify the string table contents are in the perm gen
      NOT_PRODUCT(StringTable::oops_do(&assert_is_perm_closure));
    }
  }
```

因为YGC过程不涉及到对perm做回收，因此`collecting_perm_gen`是false，而`JavaObjectsInPerm`默认情况下也是false，表示String.intern返回的字符串是不是在perm里分配，如果是false，表示是在heap里分配的，因此StringTable指向的字符串是在heap里分配的，所以ygc过程需要对StringTable做扫描，以保证处于新生代的String代码不会被回收掉

至此大家应该明白了为什么YGC过程会对StringTable扫描

有了这一层意思之后，YGC的时间长短和扫描StringTable有关也可以理解了，设想一下如果StringTable非常庞大，那是不是意味着YGC过程扫描的时间也会变长呢

## YGC过程扫描StringTable对CPU影响大吗

这个问题其实是我写这文章的时候突然问自己的一个问题，于是稍微想了下来跟大家解释下，因为大家也可能会问这么个问题

要回答这个问题我首先得问你们的机器到底有多少个核，如果核数很多的话，其实影响不是很大，因为这个扫描的过程是单个GC线程来做的，所以最多消耗一个核，因此看起来对于核数很多的情况，基本不算什么

## StringTable什么时候清理

YGC过程不会对StringTable做清理，这也就是我们demo里的情况会让Stringtable越来越大，因为到目前为止还只看到YGC过程，但是在Full GC或者CMS GC过程会对StringTable做清理，具体验证很简单，执行下`jmap -histo:live <pid>`，你将会发现YGC的时候又降下去了