##### SimpleDateFormat &DateTimeFormatter

```kotlin
import java.text.SimpleDateFormat
import java.time.LocalDateTime
import java.time.ZoneId
import java.time.format.DateTimeFormatter

fun main() {
    val timeString = "2024-09-22T16:00:00.000Z"
    val timeStamp = 1726992000000L
    val format = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'"
    run {
        val formatter = SimpleDateFormat(format)
        val date = formatter.parse(timeString)
        val millis = date.time
        println(millis == timeStamp)
    }
    run {
        val formatter = DateTimeFormatter.ofPattern(format)
        val zoneDateTime = LocalDateTime.parse(timeString, formatter).atZone(ZoneId.systemDefault())
        val millis = zoneDateTime.toEpochSecond() * 1000L
        println(millis == timeStamp)
    }
}
```

##### 线程安全问题

SimpleDateFormat这个类不是线程安全的

当有多个线程同时访问时，有概率出现时间解析错误

并且这个概率是相当大的，有几个线程，同时解析多个时间数据，非常容易触发

遇到此类问题，可以新建一个SimpleDateFormat实例

或使用DateTimeFormatter来代替，这个类是线程安全的

```kotlin
const val FORMAT_RFC3339 = "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'"
fun rfcTimeToLong(rftTimeString: String, safeMode: Boolean = true): Long {
    if (safeMode) {
        val formatter = DateTimeFormatter.ofPattern(FORMAT_RFC3339)
        val dateTime = LocalDateTime.parse(rftTimeString, formatter)
        return dateTime.atZone(ZoneId.systemDefault()).toEpochSecond();
    }
    return SimpleDateFormat(FORMAT_RFC3339).parse(rftTimeString).time
}
```

