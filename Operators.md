## Operators
Operators คือ methods ที่มีให้ใช้ใน RxJava เพื่อช่วยอำนวยความสะดวกหรือช่วยให้เราจัดการกับข้อมูลที่ถูกส่งออกมาจาก Observable ได้ง่ายขึ้น Operators ส่วนใหญ่ที่มีอยู่ใน RxJava นั้น มักช่วยเราจัดการ 3 เรื่องหลักๆดังต่อไปนี้
* ช่วยในการรวมข้อมูลเข้าด้วยกัน (CombineLastest(), merge() etc.)
* ช่วยในจัดการเกี่ยวกับ thread (observeOn(), subscribeOn() etc.)
* ช่วยในการจัดการเกี่ยวกับการปล่อยข้อมูลออกมาจาก Observable (map(), flatMap() etc.)
นอกจากนี้ยังมี Operators ที่ช่วยจัดการเกี่ยวกับ Error, การรสร้าง Observable และอื่นๆอีกมากมาย ซึ่งมี Operator ให้เราเลือกใช้เยอะมากจนผมไม่สามารถพูดถึงได้หมดภายในบทความนี้ ซึ่งหากใครที่สนใจอยากรู้ว่ามีอะไรบ้างก็สามารถเข้าไปดูได้ตาม link ที่แนบไว้ให้ในท้ายบทความครับ

ใน RxJava หากเราเรียกใช้ operators ใดๆก็ตาม ผลลัพธ์ที่ได้กลับมาจะคืนค่าเป็น Observable ตัวใหม่ให้เราเสมอ (Builder Pattern) ทั้งนี้ก็เพื่อประโยชน์ในการทำ chain operators หรือพูดง่ายๆคือ ทำให้เราสามารถนำผลลัพธ์ของ operator แรกไปใช้ใน operator ที่สอง สาม สี่ ได้ต่อไปเรื่อยๆนั่นเอง ทำให้เราสามารถที่จะปรับเสริมเติมแต่งข้อมูลที่ถูกส่งออกมาจาก Observable ให้เป็นข้อมูลที่เราต้องการได้ก่อนที่จะถูกส่งออกไปให้กับ Observer

### filter
![Filter image](https://cdn-images-1.medium.com/max/800/0*MeVEh1Yf92_hryNt.)

เริ่มต้นกันที่ filter() เป็น Operator ที่ชื่อค่อนข้างชัดเจนอยู่แล้วว่า เราใช้ filter() เพื่อกรองข้อมูลเฉพาะที่เราต้องการ filter() รับ parameter ที่เป็นประเภท Predicate ซึ่งมีไว้เพื่อให้เราสร้างเงื่อนไขในการกรองข้อมูล (Item) ลองดูตัวอย่างการใช้ filter() เพื่อกรองตัวเลขเฉพาะเลขคู่ออกมาครับ
```kotlin
val obs = Observable.just(1, 2, 3, 4, 5).filter { n -> n%2 == 0 }

//output: 2, 4
```
### map
![Map image](https://cdn-images-1.medium.com/max/800/0*MeVEh1Yf92_hryNt.)

map() คือการ Operator ที่เอาไว้ใช้เปลี่ยนข้อมูลหรือชนิดของข้อมูลให้เป็นข้อมูลใหม่ที่เราต้องการ (map Data) โดย map() จะรับ parameter เป็น Function ที่เรากำหนดให้ ซึ่ง Function นี้จะถูกนำไป apply กับทุกๆ Item ที่ถูกปล่อยออกมา ลองดูตัวอย่างการใช้ map() ด้านล่างเพื่อเปลี่ยนตัวเลขให้กลายเป็นข้อความว่า “emit number X” กันดูครับ
```kotlin
val obs = Observable.just(1, 2, 3).map { number -> “emit number: $number” }

//outputs:
emit number: 1
emit number: 2
emit number: 3
```
### flatMap
![FlatMap image](https://cdn-images-1.medium.com/max/800/0*BSh0_fZ0jA6Bzn79.)

flatMap() คือ Operator ที่ใช้เปลี่ยนแต่ละ item ให้กลายเป็น Observable จากนั้นจึงรวบทุก Observable ของแต่ละ Item เหล่านั้นให้เหลือแค่ Observable เดียว โดย parameter ที่ flatmap() รับ คือ Fuction ที่เอาไว้ใช้เปลี่ยน Item เหล่านั้นให้กลายเป็น Observable อ่านดูแล้วอาจจะเข้าใจยากหน่อย ดังนั้นจึงขออธิบายเป็นตัวอย่างดังนี้ครับ
<br/>
สมมุติว่าเรามี method หนึ่งที่คืนค่ากลับมาเป็น Observable<List<String>> ดังนี้
```kotlin
data class Customer(val name: String, val email: String)
```
ซึ่งหากว่าเราไม่ทำอะไรเลยกับ Observable ที่ได้มานี้ ผลลัพธ์ที่ได้จากการ subscribe Observable นี้ คือเราจะได้ก้อนของ List<String> ออกมา แทนที่จะเป็น String แต่ละตัวที่อยู่ใน List ซึ่งถ้าหากเราต้องการเปลี่ยนให้ Observable ปล่อยข้อมูลออกมาเป็น String แต่ละตัวแทน เราสามารถใช้ flatMap() เข้ามาช่วยได้ดังนี้
```kotlin
val searchObservable = search("Trump wife").flatMap { item -> 
    Observable.fromIterable(item) 
}

searchObservable.subscribe { girlName -> 
    logd("Wife's name: $girlName")
}
```
```text
// Outputs: 
Wife's name: Melania Trump
Wife's name: Marla Maples
Wife's name: Ivana Trump
```
สิ่งที่อยากให้ สังเกตุคือ ใน flatMap() เราครอบแต่ละ Item ด้วย Observable.fromIterable() และคืนค่า Observable ใหม่นี้ออกไป ซึ่งผลลัพธ์ที่ได้จากการใช้ flatMap() คือ Observable ที่ปล่อยข้อมูลออกมาเป็นค่าของ String ที่อยู่ภายใน List นั้นเอง

ถึงตรงนี้หลายคนคงสงสัยว่าแล้ว map() กับ flatMap() มันต่างกันยังไง เพราะทั้งคู่ก็ดูเหมือนจะเปลี่ยนข้อมูลเหมือนกัน เราจะพิสูจน์ให้ดูว่า หากเราเปลี่ยน Operator จาก flatMap() เป็น map() แทน ผลลัพธ์ที่ได้จะหน้าตาออกมาอย่างไร
```kotlin
val searchObservable = search("Trump wife").map { item ->
    Observable.fromIterable(item)
}

searchObservable.subscribe { girlName ->
    logd("Girl's name: $girlName")
}
```
```text
Output:

Girl's name: io.reactivex.internal.operators.observable.ObservableFromIterable@d260085
```
จากโค้ดด้านบนจะเห็นว่าเราเปลี่ยนจาก flatMap() เป็น map() โดยคงโค้ดส่วนอื่นไว้เหมือนเดิม แต่ผลลัพธ์ที่ได้กลับเป็นก้อน ObservableFromIterable ที่ครอบ List<String> ออกมาแทน ทั้งนี้เนื่องจากผลลัพธ์ที่ได้เป็นไปตามหน้าที่ของ map() ที่ทำหน้าที่เปลี่ยน Item ชนิดหนึ่งให้เป็นอีกชนิดหนึ่ง ซึ่ง Item ใหม่ที่ได้จากโค้ดตัวอย่างนี้ก็คือก้อน ObservableFromIterable นั้นเองครับ
<br/>
จะเห็นว่า map() กับ flatMap() นั้นมีความแตกต่างกัน ดังนั้นควรเลือกใช้ให้ถูกกับงานที่จะทำ เรามักใช้ flatMap() กับกรณีที่ต้องการเปลี่ยนจาก Observable หนึ่งไปเป็นอีก Observable หนึ่ง ซึ่งตัวอย่างเหตุการณ์จริงที่ใช้ flatMap() ได้แก่ กรณีที่ที่เราเรียก API หนึ่งด้วย Observable จากนั้นนำผลลัพธ์ของ API ที่หนึ่งไปใช้เรียก API ที่สองต่อ อย่างนี้เป็นต้น
```kotlin
gitHubApi.getUser("Nukatron") //call API to get 'user' data
            .flatMap { user -> gitHubApi.getRepository(user) } //call API to get repositories by using 'user'
            .subscribe { repositories ->
                //do something with Nukatron's repositories 
}
```
ถ้าใครยังงงๆอยู่ลองเข้าไปดูอีกตัวอย่างจากเว็บนี้เพิ่มเติมได้ครับ เค้ายกตัวอย่างไว้เคสหนึ่งที่ค่อนข้างเห็นความแตกต่างระหว่าง map() กับ flatMap() ได้ดีเหมือนกัน
### Merge
![Merge image](https://cdn-images-1.medium.com/max/800/0*7uaekMyxaeE5Ry_6.)

merge() คือ operator ที่ช่วยให้การรวม Observable หลายๆตัวให้กลายเป็น Observable เดียว ซึ่งหากเปรียบ Observable เป็นแม่น้ำ และ Item เป็นเรือแล้วละก้อ การใช้ merge() คือการรวมแม่น้ำสายต่างๆให้กลายเป็นแม่น้ำสายเดียว เหมือนที่ แม่น้ำปิง วัง ยม น่าน ไหลมาบรรจบกันกลายเป็นแม่น้ำเจ้าพระยา ดังนั้น เรือ (Item) ที่ลอยมาตามแม่น้ำ (Observable) สายต่างๆจะมาเจอกันที่แม่น้ำเจ้าพระยา (Observable ใหม่) และไหลออกสู่ทะเลทางอ่าวไทยทางเดียวกัน ลองดูตัวอย่างการใช้งาน merge() ได้จากโค้ดด้านล่าง
```kotlin
val ping = Observable.fromCallable { getKayak() }
val wang = Observable.fromCallable { getCanoe() }
val yom = Observable.fromCallable { getFerry() }
val nan = Observable.fromCallable { getCatamaran() }

val jaoPrayaRiver = Observable.merge<String>(ping, wang, yom, nan)

jaoPrayaRiver.subscribe { boat ->
    logd("Boat: $boat")
}
```
```text
// output: 
Boat: Canoe     //from wang
Boat: Kayak     //from ping
Boat: Catamaran //from nan
Boat: Ferry     //from yom
```
### Concat
![Concat image](https://cdn-images-1.medium.com/max/800/0*V7RJ1fNy006LW8EU.)
หากเราเข้าใจการทำงานของ merge() แล้ว concat() ก็จะคล้ายกันครับ เพียงแต่ concat() จะแตกต่างจาก merge() ตรงที่ concat() จะทำทีละ Observable โดยเมื่อทำ Observable ที่หนึ่งเสร็จแล้วก็จะไปทำ Observable ที่สอง สาม สี่ ต่อ เป็นอย่างนี้ต่อไปเรื่อยๆ จนครบทุกตัว ผลลัพธ์ที่ได้จึงเป็น Item ที่ถูกปล่อยออกมาเรียงตาม Observable ที่ใส่เข้าไป ซึ่งผิดกับ merge() ที่ Observable ไหนเสร็จก่อนก็สามารถปล่อยข้อมูลออกมาได้เลย ลองดูตัวอย่างผลลัพธ์ด้านล่างของการใช้ concat() เทียบกับ merge() ดูครับ

```kotlin
val ping = Observable.fromCallable { getKayak() }
val wang = Observable.fromCallable { getCanoe() }
val yom = Observable.fromCallable { getFerry() }
val nan = Observable.fromCallable { getCatamaran() }

val jaoPrayaRiver = Observable.concatArray(ping, wang, yom, nan)

jaoPrayaRiver.subscribe { boat ->
    logd("Boat: $boat")
}
```
```text
// output: 
Boat: Kayak     //from ping
Boat: Canoe     //from wang
Boat: Ferry     //from yom
Boat: Catamaran //from nan
```
### CombineLastest
![CombineLastest image](https://cdn-images-1.medium.com/max/800/0*rLdMwX_O8nfIWzOi.)

combineLastest() เป็น operator ที่ใช้ในการรวมผลลัพธ์ของสอง Observable (หรือมากกว่า) เข้าไว้ด้วยกัน โดย Item ที่จะถูกปล่อยออกมาหลังจากใช้ combineLastest() แล้วนั้น จะเป็น Item ที่เกิดจากผลลัพธ์ของ Fuction ที่เราส่งเข้าไป โดยมี input เป็น Item ล่าสุดของทั้งสอง Observable และทุกๆครั้งที่ Observable ใด Observable หนึ่งปล่อย Item ออกมา จะเป็นการ Trigger ให้ Function นี้ทำงานทันที ลองดูตัวอย่างการใช้งานด้านล่างครับ
```kotlin
val interval1 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "First: $it" }
val interval2 = Observable.interval(150, TimeUnit.MILLISECONDS).map { "Second: $it" }
val observable = Observable.combineLatest(interval1, interval2, BiFunction { item1: String, item2: String ->
    "$item1 - $item2"
}).take(5) //emit 5 items

observable.subscribe { item ->
    logd(item)
}
```
```text
// Output: 
First: 0 - Second: 0
First: 1 - Second: 0
First: 2 - Second: 0
First: 2 - Second: 1
First: 3 - Second: 1
```
จากโค้ดด้านบนเรากำหนดให้ interval1 ปล่อยข้อมูลออกมาทุกๆ 100 Miliseconds และ interval2 ปล่อย ข้อมูลออกมาทุกๆ 150 Miliseconds ผลลัพธ์ของ Observable ใหม่ที่เกิดจากการใช้ combineLatest() คือการนำ Item ล่าสุดของ interval1 และ interval2 มารวมกันผ่าน Fuction ที่เรากำหนดให้ และเมื่อดูจากผลลัพธ์ที่ได้ออกมาจะเห็นว่า item ที่ปล่อยออกมาจากทั้ง interval1 และ interval2 จะเป็นตัว trigger ให้ combineLatest() ทำงานเสมอ
### withLastestFrom
![WithLastestFrom image](https://cdn-images-1.medium.com/max/800/0*YmddVlEmMpjzzx-E.)

withLastestFrom() จะทำงานคล้ายกับ combineLastest() แต่จะต่างกันตรงที่การปล่อย Item ของ Observable source ตัวที่หนึ่งจะเป็นตัว Trigger ให้เกิดการสร้าง Item ของ Observable ใหม่แทน ซึ่งต่างจาก combineLastest() ตรงที่ Observable source ทุกตัวของ combineLastest() เป็นตัว Trigger ให้เกิดการสร้าง Item ใหม่
```kotlin
val interval1 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "First: $it" }
val interval2 = Observable.interval(150, TimeUnit.MILLISECONDS).map { "Second: $it" }
val observable = interval1.withLatestFrom(interval2, BiFunction { item1: String, item2:String ->
    "$item1 - $item2"
}).take(5) //emit 5 items

observable.subscribe { item ->
    logd(item)
}
```
```text
// Output:
First: 1 - Second: 0
First: 2 - Second: 1
First: 3 - Second: 1
First: 4 - Second: 2
First: 5 - Second: 3
```
จากโค้ดด้านบนเราเปลี่ยนจากการใช้ combineLastest() มาเป็น withLastestFrom() เมื่อดูที่ผลลัพธ์จะเห็นว่า Item ของ Observable ที่เกิดจากการใช้ withLastestFrom() เกิดจากการนำ Item ของ interval1 และ Item ของ interval2 มาผ่าน Function ที่เรากำหนดให้ โดยทุกครั้งที่มี item1 เกิดขึ้นจะเป็นการ Trigger ให้ withLastestFrom() ทำงาน
<br/>
อีกสิ่งหนึ่งที่น่าสังเหตุคือ ผลลัพธ์ที่ได้ไม่มี First: 0 - Second: 0 เนื่องจากจังหวะที่ interval1 ปล่อย First: 0 ออกมา interval2 ยังไม่ได้มีการปล่อย Item ใดๆ จึงทำให้ไม่เกิดการสร้าง Item ขึ้น จากนั้นเมื่อ interval1 ปล่อย First: 1 ออกมาและจังหวะนั้น interval2 มีข้อมูล Second: 0 พร้อมอยู่แล้ว จึงทำให้ผลลัพธ์ที่ได้ออกมาเป็นดังที่แสดงด้านบน
### Zip
![Zip image](https://cdn-images-1.medium.com/max/800/0*ky0RLxWD6GV3WW3Q.)

zip() อยู่กึ่งกลางระหว่าง combineLastest() และ withLastestFrom() โดยหลักการทำงานก็ยังคงเหมือนกัน นั้นคือมี Function ที่ทำหน้าที่นำ Item ที่ได้จาก Observable ทั้งสองตัว (หรือมากกว่า) มาสร้างเป็น Item ของ Observable ใหม่ แต่จะต่างจาก combineLastest() และ withLastestFrom() ตรงที่การ Trigger เพื่อสร้าง Item ของ zip() นั้น จะเป็นการจับคู่กันแบบ 1 ต่อ 1 ตามลำดับของข้อมูลที่ส่งออกมา ลองดูตัวอย่างประกอบครับ
```kotlin
val interval1 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "First: $it" }
val interval2 = Observable.interval(150, TimeUnit.MILLISECONDS).map { "Second: $it" }
val observable = Observable.zip(interval1, interval2, BiFunction { item1: String, item2:String ->
    "$item1 - $item2"
}).take(5) //emit 5 items

observable.subscribe { item ->
    logd(item)
}
```
```text
// output:
First: 0 - Second: 0
First: 1 - Second: 1
First: 2 - Second: 2
First: 3 - Second: 3
First: 4 - Second: 4
```
จากผลลัพธ์จะเห็นว่า Item ที่ปล่อยออกมาจะเป็นการจับคู่กันระหว่าง 1 ต่อ 1 ของ Item ที่เกิดจาก inteval1 และ interval2 ดังนั้น zip() จึงเหมาะกับงานที่เราต้องการรอให้เสร็จพร้อมกันจึงจะนำข้อมูลไปทำงานต่อได้ ตัวอย่างเช่น กรณีที่เราต้องการผลลัพธ์จากการเรียก API มากกว่าหนึ่งตัว โดยการเรียกทุก API พร้อมกัน จากนั้นนำผลลัพธ์ของ API เหล่านั้นมาทำอะไรบางอย่าง ก่อนที่จะนำผลลัพธ์ใหม่ที่ได้ไปใช้ในงานต่อไป ซึ่งถ้าเป็นการเขียนแบบ imperative programming แบบเดิมอาจจะต้องมีการเก็บ flag เพื่อบอกสถานะว่า API ทั้งหมดทำงานเสร็จแล้ว จึงจะสามารถทำงานต่อได้ ซึ่งยุ่งยากกว่าการใช้ Observable.zip() แน่นอน

## ข้อควรระวัง ในการใช้ Operators
อย่างที่ได้กล่าวไปตั้งแต่ตอนต้นว่า Operators ส่วนใหญ่ที่มีให้ใช้ใน Rxjava มักจะคืนค่ากลับมาเป็น Observable ตัวใหม่ เพื่อประโยชน์ในการทำ chain operators ซึ่งทำให้ในบางครั้งเราอาจจะเผลอใช้ Observable ตัวเก่า หรือตัวที่เราไม่ได้ใส่ Operators ที่เราต้องการเข้าไป ทำให้เกิดความผิดพลาดได้ ตัวอย่างเช่น
```kotlin
val observable = Observable.just(1, 2, 3)
observable.map { number -> number*2 }
observable.subscribe { newNumber -> 
    logd("output : $newNumber")
}
```
```text
Output:
output : 1
output : 2
output : 3
```
จากโค้ดตัวอย่างด้านบน แทนที่ผลลัพธ์ที่ได้ควรจะเป็น 2, 4, 6 เนื่องจากเราได้ใช้ map() เพื่อคูณสองเข้าไปในทุก Item แต่ผลลัพธ์ที่ได้กลับเป็น 1, 2, 3 ซึ่งสาเหตุที่เป็นอย่างนี้เพราะตอนที่เราสั่ง observable.map { number -> number*2 } นั้นเราได้ Observable ตัวใหม่กลับมา แต่เรากลับไม่ได้ใช้ Observable ตัวใหม่นั้น เรายังคงใช้ Observable ตัวเก่าอยู่ ผลลัพธ์ที่ได้จึงออกมาเป็นเช่นนี้นั้นเอง ซึ่งโค้ดด้านบนหากจะแก้ไขให้ถูกต้อง สามารถทำได้ดังนี้
```kotlin
val observable = Observable.just(1, 2, 3)
val newObservable = observable.map { number -> number*2 }
newObservable.subscribe { newNumber ->
    logd("output : $newNumber")
}
   
// Or
     
val newObservable =  Observable.just(1, 2, 3).map { number -> number*2 }
newObservable.subscribe { newNumber ->
    logd("output : $newNumber")
}
```
```text
Output:
output : 2
output : 4
output : 6
```
ข้อควรระวังอีกอย่างหนึ่งคือ Operator ที่ใช้เพื่อการรวมสาย Observable อื่นๆเข้าไว้ด้วยกัน เช่น merge(), concat() หรือ zip() หาก Observable สายใดสายหนึ่งเกิด Error ขึ้นมา Observable ที่สร้างขึ้นมาใหม่จากการใช้ Operator เหล่านั้น จะหยุดการทำงานทันทีและเรียก onError() ลองดูตัวอย่างครับ
```kotlin
val ping = Observable.fromCallable { getKayak() }
val wang = Observable.fromCallable { getCanoe() }
val yom = Observable.fromCallable { throw Exception("run out of boat!") }
val nan = Observable.fromCallable { getCatamaran() }

val jaoPrayaRiver = Observable.merge(ping, wang, yom, nan)

jaoPrayaRiver.subscribe { boat ->
    logd("Boat: $boat")
}
```
```text
Output:
Boat: Kayak               //from ping
Boat: Canoe               //from wang
error: run out of boat!   //from yom
```
## Conclusion
ในบทความนี้เราได้เรียนรู้ Operator พื้นฐานแต่ละตัวที่เรามักใช้กันบ่อยๆใน RxJava โดยจะเห็นว่าแต่ละตัวก็มีหน้าที่ที่แตกต่างกันออกไปขึ้นอยู่กับลักษณะงานที่ใช้ รวมถึงเรายังได้เรียนรู้ข้อควรระวังเมื่อใช้งาน Operator เหล่านี้ด้วย เมื่อเราศึกษาและใช้ operator ได้อย่างคล่องแคล่วแล้ว เราสามารถสร้าง Chain ของ Operator ที่ซับซ้อนเพื่อจัดการกับงานบางอย่างให้ง่ายขึ้นได้ ซึ่งยิ่งเราใช้มันมากเท่าไร ก็จะยิ่งเปลี่ยนวิธีคิดของเราให้เป็นแบบในโลกของ Reactive programming มากขึ้นเท่านั้น
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 