#Kotlin, Android��Swift

ƻ���Ѿ���Swift����Objective-C��һ�ֹ��ϵ����ԣ�������iOS�Ŀ����ˡ�����Android����Ҳ��������ơ�

��Ȼ�����Ѿ�����ѡ��Scala����Groovy�Ȼ���JVM�����Կ���AndroidӦ���������ʣ����Ǳ׶�ȴ�Զ��׼���
Ҫ����һ��ȫ�µĿ������ԣ���ô����ζ����Ҫ����������Ե�ȫ��������ʱ�����ֱ����ج�Ρ���Ϊ����
���������Ӧ�ð��Ĵ�С������˵65535�������⡣С��Ӧ�û����ԣ������ⲻ��һ�����ʵ��������Ӧ���е����⡣

##Kotlin
����һ��Kotlin--һ������JVM�����ԡ���JetBrains�����ǿ�����IntelliJ IDEA��һ��ϵ�к�Android Studio��
����������ʥ�˵ñ�������һ��С�����������������Ŀ�Ŀ����ŶӾ������С���칫����2011��Ϊ���
���£����ڼ����ĵڶ�����̱���M2�������˶�Android��֧�֣������ʱ���Ѿ�������1.0��ʽ�棩��

##����
����µ�������ʲô��ɫ�أ�����ܻ��ʡ��кܶ࣡�������ĻἯ�н���JavaӦ���ж�һֱû�е����档

##�����Ϳ�ѡ����
����������һ���ܼ򵥵��������ԣ�����ȴ�ô�������׶�����������Щ�кܶ�����ĺ���ǩ����

����������ӣ�
```java
void circle(int x, int y, int rad, int stroke) {
    ...
}
```
��java���õ�ʱ��������������ӵģ�
```java
circle(15, 40, 20, 1);
```
Ҫ��β鿴����ǩ������֪��ÿ�������ľ������á���Kotlin������������ģ�
```kotlin
fun circle(x: Int, y: Int, rad: Int, stroke: Int) {
    ...
}
```
���þ͸����ˣ�
```kotlin
circle(15, 40, rad = 20, stroke = 1)
```
��������������Ҫ��`stroke`�����趨Ϊ��ѡ�ģ���Java����ֻ��overload����һ���������������
��һ��������Ȼ����������õ�һ��������
```java
public void circle(int x, int y, int rad, int stroke) {
    ...
}

public void circle(int x, int y, int rad) {
    circle(x, y, rad, 1);
}
```
������Kotlin�һ�б�ü򵥣�
```kotlin
fun circle(x: Int, y: Int, rad: Int, stroke: Int = 1) {
    ...
}
```
����ʲô��Ĺ��ܸĽ�������Javaʼ��û�С�

##Lambda���ʽ
�������ʽ����ս����С�KotlinҲ֧��lambda���ʽ��

����������һ���򵥵ġ���������һ�������У�����Ҫɾ��ȫ����������
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

println(numbers.filter { it % 2 != 0 })
// �����[1, 3, 5]
```
���������list��Ԫ������Ϊ����������ͣ�������integer�������һ��boolean�͵Ľ����������ǰ�
filter��lambda���ʽת��Ϊһ�����Եı���д�����ľ͸���������ˣ�
```kotlin
val predicate: (Int) -> Boolean = { it -> it % 2 != 0 }
numbers.filter(predicate)
```
��һ�﷨�������Ķ�֧��lambda���ʽ�����Ե��﷨�������Բ��������͡�

##Null�����Ͱ�ȫ
����Ҫ�������Ƿ�ΪnullҲ��Java��һ���ܷ��˵��¡����������Kotlin�ﱻ���ֽ����

������ô�ͳ�ķ�������һ����������ô�������ǲ�������ֵΪnull�ġ�
```kotlin
var text1: String = "something" // ľ������
var text2: String = null // �޷�����
```
��ô����ı�����ֵ��ҪΪnull��ô���أ���Ҫר�Ŷ���һ��ֵ��Ϊnull�ı�����ֵ��Ϊnull�ı������ñ���
���ͺ�����ʺ�����ȷָ����
```kotlin
var text2: String? = null // ľ������
```
���õ�ʱ��Ҳ��Ҫ�ʺš�
```kotlin
text2?.replace(" ", "-")
```
������˼�ǣ����`text2`Ϊ�գ���ô`replace()`�����ᱻ���Բ��Ҳ�����`NullPointerException`�쳣�׳���

����������һ����ѡ���ͣ�optional type���ı���������԰ٷ�֮�ٵ�ȷ�ϱ�����ֵ����Ϊnull������Adnroid��API��
�ǳ��������Ȼ�������巵��ֵΪ�տɿ����ͣ����Ǵ������᷵��null����Ȼ�����ǿ����Ϊ�ǿյ��÷�����
```kotlin
text2!!.replace(" ", "-")
```
���ʱ�����`text2`Ϊnull�Ļ��ͻ��׳�`NulPointerException`�쳣������С��ʹ�á�

���Ͱ�ȫ����һ�����������ͼ�⡣��������һ��`Context`��ʵ��������Ҫ������ʵ���Ƿ�Ϊ`Activity`���ͣ�Ȼ�����
һЩactivity���еķ�����
```kotlin
val context = getContext()
if (context is Activity) {
    context.finish()
}
```
���ͼ��֮����Ϳ��԰�context��Ϊһ��activity���͵�ʵ����ʹ���ˣ�����Ҫǿ������ת������Java��
`instanceOf`����һ����`is`Ҳ��null��ȫ�ġ���ʹ`getContext()`����һ��null������Ĵ���Ҳ
���������

##���ݶ���Data object��
��д���ݶ����ʱ�򣬺ܶ�����Ҫ�ֶ�ʵ�ֵģ����磺`toString()`, 'hashCode()`�Լ�`equals()`����ʹ
���ںܶ��IDE�Ѿ�������һ���ֹ�����������������µĳ�Ա���ٸ�����Щʵ��Ҳ�Ƿǳ����鷳��

Kotlin�ﲻ����Ϊ��Щ�����ˡ�����Ҫ����ֻ�����ඨ���ǰ���һ��`data`�����Σ�����ķ������Ѿ���ʽ�������ˡ�
```kotlin
data class Island(val name: String? = null)
```
������ʼ��������࣬���Զ�����һ��������Զ�����`toString()`������
```
val island = Island("Kotlin")
println(island.toString())  // �����Island(name=Kotlin)
```
ͬ�����������ͬһ�����ִ�������һ��ʵ�������ʵ����`equal`֮ǰ������ʵ�����������ǵ�`hashCode()`ֵҲһ����
```kotlin
val island = Island("Kotlin")
val island2 = Island("Kotlin")

assert(island.equals(island2))
assert(island.hashCode() == island2.hashCode())
```
�����Ҫ�������ϵķ�������ô����Ȼ���㻹��Ҫ��д�ġ�

##����
������һ���ǳ����õķ�����Kotlinʡȥ�˴���������ʱ����Ҫ�ľ�̬`getInstance()`������˽�еĹ��캯����
Kotlinʹ��`object`������
```kotlin
object ApiConfig {
    val baseUrl: String = "https://github.com"
}
```
`object`����Ҳ��������������̬������
```kotlin
open class MyFragment : Fragment() {
    companion object {
        fun newInstance(): MyFragment = MyFragment()
    }
}
```
����ķ���������������`val fragment = MyFragment.newInstance()`���ͺ�Java�ľ�̬����һ����

##�ӿ�
��ȻKotlin��֧�ֶ�̳У� ���ǻ���֧�ֽӿڵġ�ֻ��������ӿں�`Trait`�ȽϽӽ��������ڽӿ��а���Ĭ��ʵ�֡�
```kotlin
interface SessionCloseable {
    fun closeSession() {
        Log.d(SessionCloseable::class.java.simpleName, "Closing...")
    }
}
```
���Ͻӿڶ�����һ�����Թر�session�ķ���������������һ��activity��ʵ�����������
```kotlin
class KotlinActivity : Activity(), SessionCloseable {
    override fun onStop() {
        super.onStop()
        closeSession()
    }
}
```
��һ����Ҫע�����Kotlin��û��extend��implement�����֣�ʲôʱ���Ƕ��ŷָ���

##��չ��extension��
������ǿ�һ�����ӣ�����Ը�һ���Ѿ����ڵ�����ӷ�����

������Ҫ����һ������������������԰��ַ����Ŀհ��滻Ϊ�»��ߡ���Java������Ҫ����һ��Utility�ķ�������
ԭʼ�ַ���Ϊ������
```java
public class StringUtils {
    public static String encodeString(String str) {
        return str.replaceAll(" ", "_");
    }
}
```
��Kotlin�����Դ���һ����չ������extension function������ʹԭ����`final`�ġ�
```kotlin
fun String.encodeSpaces(): String {
    return this.replace(" ", "_")
}
```

##���
Ϊ�˹�ƽ���������˵���Ĵ󲿷�������Java8�ﶼ�Ѿ����ˡ�������Щʲôʱ�������Android���õ�����
��ңң�����ˣ���ʵ��������Ѿ�������2014��Google I/O���ʹ��ˣ����Ǹ���Xavier Ducrohet��˵������û�������ļƻ�����
����������Kotlin�ˡ�

���⣬��������Ѿ��ǳ��ĳ��졣Ψһ����������ĵ�������ȫ���һ������ǹ��ڵģ�������Ϊ�﷨��ͬʱJVM���Ե�Scala����
��Щ�ӽ������ٿ��Բ³�����š����ԣ���ֻ��һ����С��覴á�

�������Kotlin�������ң����ߣ�������Android�����о����ܵ�ʹ�á���Ȼ�������еĴ�����ת����̫���ǡ�
����Kotlin���Ժ�Java�Ĵ����޷컥��������������ʲô̫������⡣Kotlin��ֵ��һ�ԣ�

ԭ��[��ַ](https://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/),������[����](https://github.com/future-challenger/okhttp-retrofit-rxandroid)












