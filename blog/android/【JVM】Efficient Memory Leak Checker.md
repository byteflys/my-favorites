##### Theory

in java we can receive a gc notification by `ReferenceQueue` 

when an object is recycled by jvm, `Reference` that hold it will be added to `ReferenceQueue` at the same time

so we can use `Reference` and `ReferenceQueue` to check whether an object is correctly released

##### Choose Solution

we choose `WeakReference` as the impl class, as it tells jvm to recycle object as soon as possible

we can call `System.gc()` to ask jvm for a gc execution, but this is not guranteed, it is just an advice

if you want trigger gc immediately, you can run a java command like this

```groovy
jcmd <pid> GC.run
```

if you are running on an android platform, you can force a gc through `Profiler` tool

```groovy
Profiler > New Session > Open Memory > Force Garbage Collection
```

##### Leak Check Tool

now let us achieve this tool

```kotlin
import java.lang.ref.ReferenceQueue
import java.lang.ref.WeakReference
import kotlin.concurrent.thread

object Leaker {

    class NamedWeakReference(obj: Any, val name: String) : WeakReference<Any>(obj, queue)

    private val queue = ReferenceQueue<Any>()
    private val refs = mutableListOf<NamedWeakReference>()

    init {
        thread {
            while (true) {
                Thread.sleep(1000L)
                val ref = queue.poll() as? NamedWeakReference
                if (ref != null) {
                    refs.remove(ref)
                    println("Leaker : ${ref.name} Recycled")
                }
            }
        }
    }

    fun add(obj: Any, name: String = obj.name()) {
        val ref = NamedWeakReference(obj, name)
        refs.add(ref)
    }

    private fun Any.name() = "${javaClass.simpleName}@${hashCode()}"
}
```

##### Appreciate The Masterpiece

<img src="03.png" align="left"/>

