# RxJava
บทความนี้ เป็นส่วนหนึ่งของชุดบทความที่พูดถึง RxJava โดยคุณ [nutron](https://medium.com/@nutron) ต้องขอขอบคุณมา ณ.ที่นี่ด้วย ใครอยากอยากบทความอื่นของพี่เค้าไปติดตามได้
ก่อนที่เราจะไปรู้จักกับ RxJava กันในบทความชุดนี้ บทความนี้จึงอยากจะปูพื้นฐานเบื้องต้นเกี่ยวกับคำศัพท์และเทคโนยีที่ใช้ใน RxJava กันก่อน โดยพื้นฐานเหล่านี้จะเกี่ยวข้องกับตระกูล ReactiveX ทั้งหมด ทั้ง RxJava RxJs หรือ RxSwift เป็นต้น ซึ่งจะช่วยให้เราเข้าใจหลักการและความเป็นมา รวมถึงยังเป็นพื้นฐานให้กับเนื้อหาในบทความหน้าที่เราจะพูดถึงอีกด้วย แต่เนื่องจากบทความนี้จะไม่ลงลึกถึงรายละเอียดมากนัก ซึ่งหากใครที่ต้องการรู้รายละเอียดมากขึ้น ก็สามารถตามไปอ่านต่อได้ตามลิ้งค์ที่แนบไว้ให้ครับ
- [ReactiveX คือ](https://github.com/kingland/RxJavaThai/blob/master/README.md)
- [Observer & Observable](https://github.com/kingland/RxJavaThai/blob/master/Observer%26Observable.md)
- [สร้าง Observable](https://github.com/kingland/RxJavaThai/blob/master/Observable.md)
- [Hot Observable Vs Cold Observable](https://github.com/kingland/RxJavaThai/blob/master/Hot%26ColdObservable.md)
- [Observer Vs Subscriber Vs Consumer](https://github.com/kingland/RxJavaThai/blob/master/ObserverSubscriberConsumer.md)
- [observeOn Vs subscribeOn](https://github.com/kingland/RxJavaThai/blob/master/SubscribeOn%26ObserveOn.md)
- [Operator ใน ReactiveX](https://github.com/kingland/RxJavaThai/blob/master/Operators.md)
- [Subject คือ](https://github.com/kingland/RxJavaThai/blob/master/Subject.md)
## TL;DR
เนื่องจากบทความนี้ค่อนข้างจะยาวหน่อยเลยอยากเอาส่วนสรุปมาไว้ตอนต้นซะเลย จะได้ประหยัดเวลาสำหรับผู้อ่านที่มีพื้นอยู่บ้างแล้ว ส่วนใครที่ยังเพิ่งเริ่มต้นก็ scroll กันยาวๆไปครับ ซึ่งจากบทความสามารถสรุปประเด็นที่สำคัญเป็นข้อๆได้ดังนี้
* **`Imperative programming`**  คือลักษณะการเขียนโปรแกรมแบบบอกรายละเอียดทุกขั้นตอน เน้นอธิบายการทำเพื่อให้ได้มาซึ่งผลลัพธ์
* **`Declarative programming`** คือลักษณะการเขียนโปรแกรมที่ไม่ลงรายละเอียดทุกขั้นตอน เพื่อทำให้อ่านเข้าใจง่ายขึ้นและลดความซับซ้อนของโค้ด โดยเน้นการได้มาซึ่งผลลัพธ์มากกว่าการวิธีการ
* **`Functional programming`** คือรูปแบบหนึ่งของ Declarative programming โดยมีลักษณะที่สำคัญ 3 ประการที่คือ First-class function, Higher-order function และ Pure function
* **`Observer Pattern`** คือ design pattern รูปแบบหนึ่งที่ไว้เฝ้าดูเหตุการณ์ที่เกิดขึ้นกับ object ใดๆที่เราสนใจ โดยจะแจ้งเตือนไปยังผู้ฟัง (observer) ทุกครั้ง หากมีเหตุการณ์ที่เราสนใจเกิดขึ้น
* **`Producer-Consumer Pattern`** คือ design pattern แบบหนึ่ง ที่ producer สามารถผลิตข้อมูลออกมาเก็บไว้ใน buffer ก่อนที่จะส่งข้อมูลให้กับ consumer เมื่อได้รับการร้องขอ โดย consumer สามารถร้องขอเพื่อรับข้อมูลได้มากกว่าหนึ่งข้อมูลในแต่ละครั้ง
* **`Reactive programming`** เป็นการนำหลักการของ functional programming และ Observer pattern มาช่วยให้การเขียนโค้ดเป็นไปในลักษณะของการตอบสนองต่อค่าที่เปลี่ยนแปลงไป หรือตอบสนองต่อเหตุการณ์ที่เราสนใจ
* **`ReactiveX`** เป็น library ตัวหนึ่งที่สร้างอยู่บนพื้นฐานของ Reactive programming ซึ่งเน้นการจัดการกับเหตุการณ์หรือค่าที่เปลี่ยนแปลงไปใน object ที่เราสนใจ และช่วยให้เราจัดการกับเรื่อง thread ในการทำงานได้ง่าย รวมถึงมี Operator ต่างๆที่ช่วยอำนวยความสะดวกในการจัดกับข้อมูล
* **`Observable`**  และ **`Observer`** (หรือ subscriber) คือสองสิ่งหลักๆที่จะเจอเมื่อเราใช้ ReactiveX

## Imperative programming
เริ่มต้นด้วย Imperative programming ซึ่งเป็นรูปแบบการเขียนโค้ดที่หลายๆคนคุ้นเคยกันดีอยู่แล้ว เพียงแต่อาจยังไม่รู้จักชื่อ คำว่า imperative แปลว่า จำเป็นอย่างยิ่ง ดังนั้น Imperative programming จึงเป็นการเขียนโปรแกรมในลักษณะที่จำเป็นอย่างยิ่งในการระบุทุกรายละเอียดทุกขั้นตอนในการทำงานเพื่อให้ได้มาซึ่งผลลัพธ์ ซึ่งกว่าจะได้ผลลัพธ์มานั้นต้องแลกกับจำนวนโค้ดที่บานเบอะอย่างที่เป็นกันอยู่ในปัจจุบันนี้
## Declarative programming
เมื่อการเขียนโค้ดในลักษณะที่ต้องระบุทุกขั้นตอนทำให้โค้ดดูเยอะและเข้าใจยาก จึงมีการพัฒนารูปแบบของการเขียนโค้ดที่เขียนได้กระชับขึ้นและอ่านเข้าใจง่ายขึ้น ซึ่งเรียกว่า Declarative programming นั้นเอง โดยการเขียนโปรแกรมในลักษณะนี้ จะให้ความสนใจกับผลลัพธ์มากกว่าวิธีการที่ได้มา ซึ่งทำให้เราไม่จำเป็นต้องลงรายละเอียดในทุกขั้นตอนของการทำงาน
## Imperative programming
```kotlin
val array = arrayOf(1, 2, 3, 4, 5)
val oddList = ArrayList<Int>()
for (i in array.indices) {
    if (array[i] % 2 != 0)
        oddList.add(array[i])
}
Logv("oddList: $oddList")
```
## Declarative programming
```kotlin
val list = listOf(1, 2, 3, 4, 5)
val oddList = list.filter { it % 2 != 0 }
Logv("oddList: $oddList")
```
จากโค้ดด้านบนจะเห็นว่าลักษณะโปรแกรมแบบ Declarative programming จะสั้นและเข้าใจง่ายกว่าแบบ Imperative programming โดยเราไม่จำเป็นต้องระบุทุกขั้นตอนของการได้มาซึ่งผลลัพธ์ แต่ถึงอย่างไรก็ตาม หากเข้าไปดูการทำงานของ function filter ก็คงหนีไม่พ้นการวนลูปหาค่าเหมือนกับ Imperative programming อยู่ดี เพียงแต่ขั้นตอนเหล่านั้นเราไม่จำเป็นต้องเขียนมันขึ้นมาเอง
## Functional programming
Functional programming คือรูปแบบหนึ่งของการเขียนโปรแกรมแบบ declearative programming โดยเป็นรูปแบบของการเขียนโปรแกรมในลักษณะที่หลีกเลี่ยงการแชร์ข้อมูลระหว่างกัน (avoiding shared state) หลีกเลี่ยงการยุ่งเกี่ยวกับข้อมูลที่สามารถเปลี่ยนแปลงค่าได้ (mutable data) เพื่อไม่ให้เกิดผลกระทบข้างเคียง (side effect) ที่อาจจะเกิดขึ้นจากการใช้ข้อมูลเหล่านั้น โดยคุณลักษณะหลักที่สำคัญของ Functional programmimg มีอยู่ด้วยกัน 3 อย่างคือ
* **`First-class function`** คือ การที่มองว่า function เป็นเสมือน object ก้อนหนึ่งที่สามารถเก็บในรูปแบบของตัวแปลได้ และส่งต่อให้กันได้ในรูปแบบของ argument ซึ่งจะแตกต่างจาก imperative programing ตรงที่ function ไม่ได้เป็น object และไม่สามารถส่งต่อ function ไปให้ฟังชั่นอื่นได้
* **`Higher-order function`** เนื่องจากการที่ function เป็นแค่ object ก้อนนึง จึงทำให้เกิด Higher-order function ขึ้น ซึ่ง Higher-order function คือ function ที่รับ argument เป็น function หรือ คืนค่ากลับไปเป็น function หรือ ทำทั้งสองอย่าง
* **`Pure Function`** คือฟังชั่นที่ไม่ก่อให้เกิด side effect ต่างๆ หรือพูดให้เข้าใจง่ายขึ้นกว่านี้คือ function ที่ไม่ยุ่งเกี่ยวกับค่าอื่นใดที่อยู่นอกเหนือ function และ agument ที่ส่งเข้ามา นั้นหมายความว่า ผลลัพธ์ของการรันด้วย function นี้ ไม่ว่าจะรันกี่ครั้งด้วย agument แบบเดิมจะได้ค่าเดิมเสมอ ตัวอย่างเช่น
```kotlin
fun plusAndMultiplyByTwo(a: Int, b: Int): Int {
  val factor = 2
  return (a + b) * factor
}
...
plusAndMultiplyByTwo(1, 2) //output: 6
plusAndMultiplyByTwo(1, 2) //output: 6
plusAndMultiplyByTwo(1, 2) //output: 6
```
## Observer Pattern
reactive programming ถูกสร้างขึ้นมาอยู่บนพื้นฐานของ Observer pattern ซึ่งเจ้า Observer pattern นั้นเป็น design pattern รูปแบบหนึ่งที่ไว้ใช้เพื่อเฝ้าดูเหตุการณ์ที่เกิดขึ้นกับ object ใดๆที่เราสนใจ โดยเมื่อมีการเปลี่ยนแปลงเกิดขึ้นกับ object นั้น คนที่คอยฟังเหตุการณ์นั้นอยู่ (observer) ก็จะได้รับการแจ้งเตือนอัตโนมัติทันที ลองดูตัวอย่างด้านล่างประกอบครับ
```kotlin
interface ObserverListener {
    fun onValueChange(value: Int)
}

class IntObserver {

    private val listener =  mutableListOf<ObserverListener>()

    var value : Int = 0
        set(value) {
            field = value
            notifyObserver(field)
        }

    fun addObserver(observer: ObserverListener) {
        listener.add(observer)
    }

    fun notifyObserver(value: Int) {
        listener.forEach { it.onValueChange(value) }
    }
}

fun main(args: Array<String>) {
    val intObserver = IntObserver()
    val observer1 = object : ObserverListener {
        override fun onValueChange(value: Int) {
                println("observe on observer1 got: $value")
            }
        }
    val observer2 = object : ObserverListener {
        override fun onValueChange(value: Int) {
                println("observe on observer2 got: $value")
            }
        }
    intObserver.addObserver(observer1)
    intObserver.addObserver(observer2)
    intObserver.value = 1
    intObserver.value = 5
    intObserver.value = 9
    
}
```
```text
Output:

observe on observer1 got: 1
observe on observer2 got: 1
observe on observer1 got: 5
observe on observer2 got: 5
observe on observer1 got: 9
observe on observer2 got: 9
```
จากโค้ดจะเห็นว่า เมื่อมีการเปลี่ยนแปลงค่าของ value ใน IntObserver เหตุการณ์นั้นก็จะถูกแจ้งเตือนไปยัง observer1 และ observer2 ที่คอยฟังอยู่ทันที นอกจาก Observer pattern ก็ยังมีอีกหนึ่ง design pattern ที่อยากจะพูดถึง นั้นคือ Producer-Consumer pattern เนื่องจากเป็นรูปแบบการทำงานที่คล้าย อีกทั้งยังมีบางส่วนของชุดของบนความนี้ที่จะกล่าวถึง design pattern นี้จึงอยากขอยกมาอธิบายกันสักหน่อย
## Producer-Consumer pattern (PC-Pattern)
หากเราเข้าใจการทำงานของ observer pattern แล้ว การทำงานของ PC-pattern ก็จะคล้ายๆกัน แต่จะต่างกันตรงที่ observer pattern นั้น หากมีการเปลี่ยนแปลงค่าของ object ที่เราสนใจ มันจะทำการแจ้งเตือนกลับไปให้ผู้ฟัง หรือ observer ทันที ในขณะที่ PC-Pattern นั้น ตัว producer จะเก็บข้อมูลเหล่านั้นไว้ใน buffer ก่อน หลังจากนั้นคอยส่งข้อมูลให้กับ consumer เมื่อมีการร้องขอ โดย consumer สามารถร้องขอเพื่อรับข้อมูลได้มากกว่าหนึ่งข้อมูลในแต่ละครั้ง ขึ้นอยู่กับความสามารถในการจัดการข้อมูลของ consumer PC-pattern จึงเป็น pattern ที่ช่วยแก้ปัญหาในเรื่องของการจัดการกับข้อมูลที่ต้องใช้เวลานานในฝั่งผู้รับ โดยต่างจาก observer pattern ที่ไม่สนใจว่าผู้รับจะทำงานนั้นทันหรือไม่ซึ่งอาจก่อให้เกิดการทำงานที่ผิดพลาดขึ้นได้ ซึ่งหากใครสนใจเพิ่มเติมสามารถตามไปอ่านต่อได้ที่ท้ายบทความครับ
## ReactiveX (Rx)
ปูพื้นกันมาตั้งนาน ก็ได้เวลามาทำความรู้จักกับเจ้า ReativeX กันสักที โดย ReativeX ย่อมาจากคำว่า Reactive Extensions ซึ่งเป็น Library ที่ถูกพัฒนาบนพื้นฐานของ Reactive programming โดยจะเน้นไปที่การจัดการกับเหตุการณ์หรือค่าที่เปลี่ยนแปลงไปใน object ที่เราสนใจ นอกจากนี้ยังอำนวยความสะดวกให้ผู้ใช้ในเรื่องของ thread โดยสามารถสลับการทำงานระหว่าง thread ไปมาได้อย่างง่ายดาย อีกทั้งยังมี operator ต่างๆที่ช่วยในการจัดการกับข้อมูล เพื่อให้ผู้ใช้สามารถเลือกรับเฉพาะข้อมูลที่สนใจได้ และเมื่อกล่าวถึง ReactiveX จะมีสองอย่างที่ถูกพูดถึงและถูกเรียกใช้งานอยู่เป็นประจำนั้นคือ Observable และ Observer (หรือที่บางคนเรียกว่า Subscriber) ซึ่งเมื่อดูจากชื่อแล้วก็น่าจะเดาออกได้ไม่ยากว่ามาจาก Observer pattern นั้นเอง โดย Observable คือ ฝั่งที่คอยส่งข้อมูลออกมา (producer) ส่วน Observer คือ ฝั่งที่คอยรับข้อมูลที่ถูกส่งออกมานั้นเอง (consumer)
## ทำไมต้องใช้ ReactiveX
ด้วยการเขียนโค้ดในรูปแบบ imperative programming แบบเดิมๆ ทำให้การทำงานบางอย่างค่อนข้างยุ่งยากและซับซ้อน จนในหลายๆครั้งก็สร้างปัญหาให้กับเรา ตัวอย่างของงานเหล่านี้ได้แก่ การเรียก API ที่ต่อเนื่องกัน ซึ่งแน่นอนว่าเราจำเป็นที่จะต้องสร้าง callback ขึ้นมาเพื่อบอกอีก API หนึ่งว่าทำงานเสร็จแล้ว จึงจะสามารถเรียก API ที่สองต่อได้ และถ้าหากมีมากกว่า 2 API ก็คงหนีไม่พ้น callback ซ้อน callback กันไปเรื่อยๆ เกิดเป็นสิ่งที่เรียกว่า Callback Hell ซึ่งหากมี Error เกิดขึ้น การจัดการกับ Error นั้นจะทำได้ยาก หรืออีกกรณีคือ หากต้องการทำหน้าค้นหาข้อมูล โดยให้ระบบค้นหาข้อมูลแบบอัตโนมัติขณะที่ผู้ใช้พิมพ์ ซึ่งหากยิง API ทุกครั้งที่ผู้ใช้พิมพ์ก็คงจะไม่ดีแน่ ดีขึ้นมาหน่อยก็อาจจะสร้าง timer เพื่อหาช่วงเวลาที่ user หยุดพิมพ์แล้วค่อยยิง API แต่ก็ทำให้การเขียนโค้ดดูซับซ้อนและยุ่งยากมากขึ้น กรณีตัวอย่างเหล่านี้ หากเรานำ Rx เข้ามาช่วย ก็จะสามารถทำได้เพียงแค่เขียนโค้ดไม่กี่บรรทัด แต่ถึงอย่างไรก็ตาม ก็ไม่ได้แปลว่า Rx จะเป็นยาวิเศษรักษาทุกโรค ไม่ใช่เปลี่ยนทุกอย่างมาใช้ Rx หมด แต่อยากให้มองว่าถ้าใช้ Rx แล้วทำให้ชีวิตมันง่ายขึ้นก็ใช้มันเถอะครับ แต่หากทำด้วยวิธีเดิมแล้วมันง่ายกว่าก็กลับไปใช้วิธีเดิมนั้นแหละครับ แต่ทั้งนี้ทั้งนั้นก็ต้องระวังเรื่องของโครงสร้างของโค้ดด้วย ไม่ใช่เขียนปนไปปนมาจนทำให้ชีวิตยากขึ้นกว่าเดิม
## Conclusion
จากบทความข้างต้นก็ทำให้เราได้เข้าใจว่า ReactiveX นั้นมีความเป็นมาอย่างไร และมีประโยชน์อย่างไร หากเราเข้าใจหลักการและทฤษฎีต่างๆเหล่านี้ จะทำให้เราเข้าใจ ReactiveX มากขึ้นและเข้าใจสิ่งที่จะกล่าวถึงในบนความหน้าได้ง่ายขึ้นอีกด้วย
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 
