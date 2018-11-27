- [ReactiveX คือ](https://github.com/kingland/RxJavaThai/blob/master/README.md)
- [Observer & Observable](https://github.com/kingland/RxJavaThai/blob/master/Observer%26Observable.md)
- [สร้าง Observable](https://github.com/kingland/RxJavaThai/blob/master/Observable.md)
- [Hot Observable Vs Cold Observable](https://github.com/kingland/RxJavaThai/blob/master/Hot%26ColdObservable.md)
- [Observer Vs Subscriber Vs Consumer](https://github.com/kingland/RxJavaThai/blob/master/ObserverSubscriberConsumer.md)
- [observeOn Vs subscribeOn](https://github.com/kingland/RxJavaThai/blob/master/SubscribeOn%26ObserveOn.md)
- [Operator ใน ReactiveX](https://github.com/kingland/RxJavaThai/blob/master/Operators.md)
- [Subject คือ](https://github.com/kingland/RxJavaThai/blob/master/Subject.md)
## What are they look like?
ก่อนที่จะไปดูความแตกต่างและหลักการทำงานของทั้งสามตัวนี้ ก็อยากจะพูดถึง method ที่สำคัญที่เราต้องรู้จักเมื่อใช้งาน Rxjava กันก่อน โดยจะขอยกส่วนหนึ่งจากบทความที่สอง ที่พูดถึงหลักการทำงานของ method ที่อยู่ภายใน Observer มาเพื่อเป็นการทบทวนดังนี้
<br/>
**Observer** คือ interface ที่ประกอบไปด้วย 4 functions หลักที่สำคัญคือ
* onSubscribe(Disposable d): จะถูกเรียกเมื่อมีเหตุการณ์ subcribe เกิดขึ้น และจะส่ง Disposable object เข้ามาทาง parameter ของ function โดยเราสามารถใช้ object นี้เพื่อหยุดการทำงานของการ subscribe ครั้งนั้นๆได้ (ซึ่งจะพูดถึงในตอนท้ายของบนความ)
* onNext(T item): คือช่องทางที่ Observer ไว้ใช้รับข้อมูลจาก Observable โดย Observable จะส่งข้อมูลเข้ามาผ่านทาง parameter ของ onNext()
* onError(Throwable e): จะถูกเรียกโดย Observable เมื่อมีข้อผิดพลาดใดๆเกิดขึ้นระหว่างการทำงาน โดยจะส่ง Throwable เข้ามาผ่านทาง Parameter ของ onError() ซึ่งเมื่อเรียก onError() แล้ว Observable จะหยุดการทำงานทันที โดยจะไม่มีการเรียก onNext() หรือ onComplete() ต่ออีก
* onComplete(): จะถูกเรียกโดย Observable เป็นลำดับสุดท้ายหลังจากที่ไม่มี item ใดๆที่ต้องการปล่อยออกมาผ่านทาง onNext() อีกแล้ว โดยที่ Observable จะหยุดการทำงานทันทีเมื่อ onComplete() ถูกเรียก”
## Observer
```kotlin
interface Disposable {
 void dispose();
}

interface Observer<T> {
 void onNext(T t);
 void onComplete();
 void onError(Throwable t);
 void onSubscribe(Disposable d);
}
```
เริ่มกันที่ Observer ฝั่งรับที่เราคุ้นเคยกันดี จากโค้ดด้านบนจะเห็นว่า Observer คือ Interface ที่มี onNex(), onComplete(), onError() และ onSubscribe() โดยมี Disposable เป็น interface ที่เอาไว้ใช้หยุดการทำงานของ Observable
## Subscriber
```kotlin
interface Subscription {
 void cancel();
 void request(long r);
}

interface Subscriber<T> {
 void onNext(T t);
 void onComplete();
 void onError(Throwable t);
 void onSubscribe(Subscription s);
}
```
Subscriber เป็น interface ที่มีทุกอย่างเหมือน Observer แต่จะแตกต่างกันตรงที่มี Subscription ที่เอาไว้หยุดการทำงานของ Observable แทน
## Consumer
```kotlin
public interface Consumer<T> {
    void accept(@NonNull T t) throws Exception;
}
```
Consumer เป็น Callback Interface ที่รับข้อมูลแค่ประเภทเดียว โดยมี method accept() ทำหน้าที่เป็น callback ที่จะถูกเรียกเมื่อ Observable ต้องการส่งข้อมูลหรือเหตุการณ์ใดๆออกมา
<br/>
เราได้เห็นหน้าตา Interface ของฝั่งรับทั้งสามตัวกันไปแล้ว ซึ่งจะเห็นว่าระหว่าง Subscriber และ Observer นั้น ดูจากหน้าตาแล้วก็ไม่ได้ต่างกันมาก แต่ Consumer ดูค่อนข้างจะแตกต่างจากพวกพอสมควร อย่างไรก็ตามทั้งสามตัวนี้ทำหน้าที่อย่างเดียวกันคือ รับข้อมูลและตอบสนองต่อข้อมูลที่ได้รับมาจาก Observable ซึ่งอาจจะมีรายละเอียดปลีกย่อยที่แตกต่างกันออกไป ขึ้นอยู่กับจุดประสงค์ของการนำไปใช้งานครับ
## Comparison
หลังจากที่ได้เห็นหน้าตา รวมถึงหลักการทำงานของ Method ที่ควรรู้กันไปแล้ว ต่อไปจะเป็นการอธิบายความแตกต่างของตัวรับแต่ละประเภทว่ามีความเหมือนหรือต่างกันอย่างไร โดยจะขออธิบายด้วยการเปรียบเทียบกันเป็นคู่ๆดังนี้
### Subscriber Vs Observer
เริ่มต้นด้วยการเปรียบเทียบเจ้าสองตัวนี้กันก่อน โดยทั้งสองตัวนี้ได้ถูกพูดถึงไปแล้วในบทความที่สอง แต่ก็ยังมีบางส่วนที่ยังไม่ได้พูดถึง ในบทความนี้จึงขอสรุปออกมาเป็นข้อๆ เพื่อให้ง่ายต่อการเข้าใจดังนี้
* Subscriber และ Observer ใช้ในการรับ event หรือ item ที่ปล่อยออกมาจาก Observable เหมือนกัน
* ใน RxJava 1.x Subscriber คือ implementation class ของ Observer แต่ใน RxJava 2.x Subscriber จะเป็นแค่ Interface หนึ่งเหมือนกับ Observer
* ใน RxJava 1.x เราใช้ Subscriber ซึ่งเป็น implementation class ของทั้ง Observer และ Subscription เมื่อต้องการ Subscribe หรือ Unsubscribe กับ Observable
* ใน Rxjava 2.x ทั้ง Subscriber และ Observer สามารถ Unsubscribe ได้ทั้งคู่ โดยการ Unsubscribe ของ Observer ใน Rxjava 2.x จะเป็นการเรียก methoddispose() ของ Observer ที่ implement interface Disposable ส่วน Subscriber จะเป็นการเรียก cancel() ของ Subscriber ที่ implement interface Subscription ตัวอย่างของ Disposable object ได้แก่ ResourceObserver และตัวอย่างของ Subscription object ได้แก่ LambdaSubscriber เป็นต้น
* Subscriber เป็นส่วนหนึ่งของการทำตามข้อกำหนดหนึ่งในโลกของ Reactive (Reactive Stream Specification) โดยข้อกำหนดที่ว่านั้นคือ Observable จะส่งข้อมูลให้กับ Subscriber ครั้งต่อไปได้นั้นก็ต่อเมื่อ Subscriber ทำการร้องขอ ซึ่งในที่นี่คือคำสั่ง Subscription.request(long n) โดยจะแตกต่างจาก Observer ตรงที่ Observer ไม่มีการร้องขอ
* เนื่องจากข้อกำหนดที่ว่า Subscriber จะได้รับข้อมูลก็ต่อเมื่อมีการร้องขอไปที่ Observable ดังนั้นใน RxJava 2.x จึงนำข้อดีของเจ้า Subscriber นี้มาช่วยในการทำ backpressure (ความสามารถในการชะลอการส่งข้อมูลของผู้ผลิต (Producer) ไปสู่ผู้บริโภค (Consumer)) โดยใน RxJava 2.x นี้ เจ้า Subscriber จะถูกนำไปใช้คู่กับ Flowable (Observable แบบหนึ่งที่กล่าวถึงในบทความที่สอง) และ Observer จะถูกใช้กับ Observable ซึ่งไม่ได้มีการจัดการเรื่องของการทำ backpressure
### Observer Vs Consumer
เราเข้าใจความแตกต่างของ Observer และ Subscriber กันไปแล้ว แต่หลายๆคนที่เคยใช้ RxJava 2.x มาบ้างแล้วอาจจะสงสัยว่า บ่อยครั้งที่เราใช้ Consumer เพื่อรับค่าจาก Observable แทนการเรียกใช้ Observer โดยตรง ซึ่งชวนให้สงสัยว่า เจ้า Consumer นั้นต่างยังไงกับ Observer
<br/>
ทั้ง Observer และ Consumer ทำหน้าที่เหมือนกันคือเป็น Callback ที่รับ event หรือ item ที่ถูกส่งออกมาจาก Observable แต่เราใช้ Consumer ในกรณีที่เราต้องการแยกการจัดการกับข้อมูลในแต่ละเหตุการณ์ที่เกิดขึ้นกับ observable นั้นๆ เช่น เราใช้ Consumer ตัวที่หนึ่งเพื่อจัดการกับข้อมูลที่เกิดขึ้นใน onNext() เราใช้ Consumer อีกตัวหนึ่งเพื่อจัดการกับข้อมูลที่เกิดขึ้นใน onError() เป็นต้น ซึ่งถ้าหากเราใช้ Observer เราจะต้องจัดการทุกอย่างภายใน Observer นั้นเพียงตัวเดียว ลองดูตัวอย่างประกอบ
```kotlin
// Using Consumner 
val observable = Observable.just(ValueEntity(1, 1))
val disposable = observable.observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                Consumer { item -> 
                    // do something with item when onNext()
                }, Consumer { error ->
                    // handle error when onError()
                }, Action {
                    // do some action when onComplete()
                }, Consumer { disposable ->
                    // do something when onSubscribe()
                }
            )

...

// Using Observer
val observable = Observable.just(ValueEntity(1, 1))
val disposable = observable.observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : Observer<ValueEntity> {
    
            override fun onNext(item: ValueEntity) {
               // do something with item when onNext()
            }
            override fun onError(error: Throwable) {
                // handle error when onError()
            }
            override fun onComplete() {
                // do some action when onComplete()
            }
            override fun onSubscribe(d: Disposable) {
                // do something when onSubscribe()
            }
})
```
จากโค้ดจะเห็นว่าเราสามารถใช้ Consumer แทน Observer ได้ ซึ่งจริงๆแล้วหากเราลองเข้าไปดูโค้ดข้างในของ Rxjava 2.x แล้ว เราพบว่า Consumer ที่เราใส่เข้าไปนั้น แท้จริงแล้วมันจะถูกห่อด้วย Object ที่ Implement Interface Observer อีกที นั้นหมายความว่า การทำงานที่เกิดขึ้นภายในก็ยังคงเป็น Observable กับ Observer อยู่ดี เพียงแต่มี method subscribe() ที่รับ Consumer เพิ่มขึ้นมาเพื่อช่วยอำนวยความสะดวกในการใช้งานให้มากขึ้นเท่านั้นเอง
## Why do we need Consumer?
ถึงตรงนี้หลายคนก็อาจจะสงสัยว่า แล้วทำไมไม่ใช้ Observer ไปเลยล่ะ จะได้จัดการทุกอย่างในตัวเดียว ไม่ต้องสร้าง Consumer หลายตัว คำตอบคือทำแบบนั้นก็ได้ครับแต่จำเป็นรึป่าว? บางครั้งเราไม่จำเป็นต้องไปจัดการกับเหตุการณ์ทุกเหตุการณ์ เนื่องจากเรารู้อยู่แล้ว่ามันไม่มีทางเกิดเหตุการณ์เหล่านั้นขึ้น ยกตัวอย่างเช่น หากเราสนใจแต่ onNext() ไม่สนใจ onError() และ onSubscribe() เนื่องจากเรารู้อยู่แล้วว่าไม่มีทางที่จะมี error เกิดขึ้น และเราก็ไม่ต้องการที่จะทำอะไรใน onSubscribe() หากเราใช้ Observer เราก็ต้องไล่ overide ทุก method ที่มี ต่างจากการใช้ Consumer ที่เราสามารถเลือกที่จะใส่แค่ Consumer ของ onNext() เพียงอย่างเดียวได้ ลองดูตัวอย่างด้านล่างประกอบครับ
```kotlin
val observable = Observable.just(ValueEntity(1, 1))
val disposable = observable.observeOn(AndroidSchedulers.mainThread())
        .subscribe(Consumer {
            // do something with item when onNext()
        })
```
ซึ่งจะเห็นว่า การเลือกใช้ Consumer จะช่วยให้เกิดความยืดหยุ่นขึ้น และสามารถเลือกสนใจเฉพาะเหตุการณ์ที่เราต้องการได้ โดยที่ไม่จำเป็นต่าง override ทุก method เหมือนกับการใช้ Observer แต่ทั้งนี้ทั้งนั้น ในโค้ดตัวอย่างด้านบนหากเกิด Error ขึ้นจริงๆภายใน Observable โปรแกรมก็จะพังทันทีเหมือนกัน เนื่องจากเราไม่ได้มีการจัดการกับ Error ที่เกิดขึ้น ดังนั้นจะใช้อะไรก็จะควรรู้ว่ากำลังทำอะไรอยู่และระมัดระวังในการใช้งานด้วยครับ
## Action
ไหนๆเราก็พูดถึง Consumer กับไปแล้วเลยอยากจะแถม Action ไปด้วยเลยอีกสักนิด จากโค้ดตัวอย่างการใช้งาน Consumer ด้านบน หลายคนอาจจะสังเกตุเห็นว่า นอกจาก Consumer แล้วเรายังมีการใช้ Action สำหรับเหตุการณ์ onComplete() อีกด้วย ซึ่งจริงๆแล้วหลักการทำงานของ Action ก็คล้ายๆกับ Consumer ที่จะถูกครอบด้วย Observer อีกทีหนึ่งในภายหลัง เพียงแต่ Action จะต่างจาก Consumer ตรงที่ Action ทำหน้าที่คล้ายกับ Runnable ใน Java ซึ่งจะไม่รับค่าใดๆเข้ามาและไม่คืนค่าใดๆออกไป รวมถึงยังอนุญาตให้เรา throw exception ออกไปได้เหมือนกับ Java ดังนั้น Action จึงถูกนำมาใช้กับเหตุการณ์ที่มีลักษณะอย่าง onComplete() ได้
```kotlin
/**
 * A functional interface similar to Runnable but allows throwing a checked exception.
 */
public interface Action {
    /**
     * Runs the action and optionally throws a checked exception.
     * @throws Exception if the implementation wishes to throw a checked exception
     */
    void run() throws Exception;
}
```
## Conclusion
ในบทความนี้เราได้เรียนรู้ความความแตกต่างและลักษณะการใช้งานของตัวรับอย่าง Observer, Subscriber และ Consumer กันไปแล้ว นอกจากนี้ยังได้เรียนรู้การทำงานของ Action ไปด้วย ซึ่งจะเลือกใช้อะไรนั้น ขึ้นอยู่ลักษณะงานที่เราต้องการทำ ซึ่งจากบทความสามารถสรุปเป็นข้อๆได้ดังนี้
* Observer, Subscriber และ Consumer ทำหน้าที่อย่างเดียวกันนั้นคือ รับและตอบสนองต่อข้อมูลที่ได้รับมาจาก Observable
* เราใช้ Interface Disposable ในการหยุดการทำงานระหว่าง Observable และ Observer
* เราใช้ Interface Subscription ในการหยุดการทำงานระหว่าง Flowable และ Subscriber
* Subscriber ถูกสร้างขึ้นจากข้อกำหนดหนึ่งในโลกของ Reactive (Reactive Stream Specification) โดย Observable จะส่งข้อมูลให้กับ Subscriber ครั้งต่อไปได้นั้น ก็ต่อเมื่อ Subscriber ทำการร้องขอ
* Consumer เป็น Callback Interface ที่รับข้อมูลเพียงประเภทเดียว
* Rxjava อำนวยความสะดวกให้เราสามารถใช้ Consumer แทนการเรียกใช้ Observer หรือ Subscriber โดยตรงได้ แต่เบื่องหลังการทำงานแล้ว Consumer เหล่านั้นจะถูกห่อด้วย Observer หรือ Subscriber อีกทีหนึ่ง
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 
