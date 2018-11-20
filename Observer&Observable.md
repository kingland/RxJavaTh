## Observable
Observable คือ ตัวที่เชื่อมระหว่างผู้ผลิตข้อมูล (Producer) และผู้รับข้อมูล (Consumer) เพื่อทำหน้าที่จัดการกระบวนการปล่อยข้อมูลหรือชุดของข้อมูลออกมา (emit item or sequence of items) โดยมี Observer ที่เป็นเหมือน Consumer ทำหน้าที่รับข้อมูลและตอบสนองต่อข้อมูลที่ถูกส่งออกมาเหล่านั้น Observable ยังอนุญาตให้เราสามารถที่จะทำการเสริมเติมแต่งในสายข้อมูล (allow to compose stream) ก่อนที่ข้อมูลเหล่านั้นจะถูกปล่อยออกมาให้กับ Observer เพื่อที่จะให้ Observer สามารถนำไปข้อมูลที่ได้นั้นไปใช้งานได้ทันที นอกจากนี้ Observable ยังช่วยให้การทำงานแบบ asynchronous เป็นเรื่องที่ง่ายขึ้น โดยเราสามารถสลับ thread การทำงานไปมาได้อย่างง่ายดาย และยังช่วยลดปัญหาเรื่อง callback hell ทำให้ง่ายต่อการอ่านโค้ดและง่ายต่อการจัดการกับโค้ดอีกด้วย
<br/>
![Observable image](https://cdn-images-1.medium.com/max/800/1*xen7ToAKzdoDwiqK1BxXMg.png)
<br/>
## Observer
เรารู้จักฝั่งส่งกันไปแล้ว ที่นี้เรามารู้จักกับฝั่งรับกันบ้าง อยากที่ได้เกริ่นไปแล้วว่า Observer ทำหน้าที่รับข้อมูลและตอบสนองต่อข้อมูลที่ได้รับมาจาก observable โดยขั้นตอนการเชื่อมต่อระหว่าง Observer และ Observable เรียกว่าการ subcribe ซึ่งเป็นขั้นตอนที่บอกว่า Observer กำลังรอฟังเหตุการณ์ต่างๆจาก Observable อยู่ ถ้าหากเปรียบเป็นเหตุการณ์ก็จะคล้ายๆกับตอนที่เรา subscribe channel ใน Youtube เพื่อรอรับการแจ้งเตือนเมื่อมีการอัพโหลดวิดีโอใหม่ๆนั้นเอง เมื่อเราเข้าไปดูโค้ดการทำงานของ Observer เราจะเห็นว่าจริงๆแล้ว Observer เป็นแค่ interface ที่ประกอบไปด้วย 4 functions หลักที่สำคัญคือ
```java
public interface Observer<T> {
    void onSubscribe(@NonNull Disposable d);
    void onNext(@NonNull T t);
    void onError(@NonNull Throwable e);
    void onComplete();
}
```
* **`onSubscribe(Disposable d)`**: จะถูกเรียกเมื่อมีเหตุการณ์ subcribe เกิดขึ้น และจะส่ง Disposable object เข้ามาทาง parameter ของ function โดยเราสามารถใช้ object นี้เพื่อหยุดการทำงานของการ subscribe ครั้งนั้นๆได้ (ซึ่งจะพูดถึงในตอนท้ายของบนความ)
* **`onNext(T item)`**: คือช่องทางที่ Observer ไว้ใช้รับข้อมูลจาก Observable โดย Observable จะส่งข้อมูลเข้ามาผ่านทาง parameter ของ onNext()
* **`onError(Throwable e)`**: จะถูกเรียกโดย Observable เมื่อมีข้อผิดพลาดใดๆเกิดขึ้นระหว่างการทำงาน โดยจะส่ง Throwable เข้ามาผ่านทาง Parameter ของ onError() ซึ่งเมื่อเรียก onError() แล้ว Observable จะหยุดการทำงานทันที โดยจะไม่มีการเรียก onNext() หรือ onComplete() ต่ออีก
* **`onComplete()`**: จะถูกเรียกโดย Observable เป็นลำดับสุดท้ายหลังจากที่ไม่มี item ใดๆที่ต้องการปล่อยออกมาผ่านทาง onNext() อีกแล้ว โดยที่ Observable จะหยุดการทำงานทันทีเมื่อ onComplete() ถูกเรียก
และเพื่อให้เข้าใจและเห็นภาพได้ชัดเจนมากยิ่งขึ้นเราลองมาดูภาพอธิบายการทำงานของ Observable จากเว็บ ReactiveX ประกอบครับ
<br/>
![ReactiveX image](https://cdn-images-1.medium.com/max/800/0*EVHjFNGDhPjT6TbU.)
<br/>
จากภาพคือแผนภาพ Mable (marble diagram) ที่อธิบายการทำงานของ Observable ที่มี Operator ขั้นตรงกลาง โดย Operator ทำหน้าที่ปรับเสริมเติมแต่งข้อมูลก่อนที่ข้อมูลนั้นจะถูกส่งต่อไปยัง Observer ซึ่งเราจะพูดถึง Operator ในบทความต่อไป ทั้งนี้แผนภาพ Marble นี้จะถูกใช้ในการอธิบายการทำงานของ Observable และ function ต่างๆที่มีให้ใช้ใน ReactiveX เยอะมาก ดังนั้นจึงอยากให้ทำความคุ้นเคยกับมันไว้ครับ
## Observable พื้นฐานที่ควรรู้จัก
เราพอจะรู้จักการทำงานของ Observable กันไปบ้างแล้ว ทีนี้เรามาลองรู้จักประเภทของ Observable ใน RxJava 2 กันบ้างว่ามี Observable ประเภทไหนบ้างที่เราควรรู้ และแต่ละประเภทมีความแตกต่างกันอย่างไร
### Observable
Observable คือสิ่งที่เราจะเจอบ่อยที่สุดเมื่อเราใช้ ReactiveX ซึ่ง Observable เป็นตัวที่สามารถส่งข้อมูลออกมาได้ตั้งแต่ 0 item จนถึง N items โดยจะปล่อยข้อมูลให้กับผู้รับผ่านทาง onNext(), แจ้ง error ที่เกิดขึ้นผ่านทาง onError() และจบการทำงานด้วยการเรียก onComplete() โดยหลังจากที่เรียก onError() หรือ onComplete() แล้วนั้น Observable ก็จะหยุดการทำงานลงทันที ซึ่ง function ทั้งหมดที่นี้ถูกอธิบายไว้แล้วในตอนต้นของบนความนั้นเองครับ
### Single
Single คือ Observable ที่สามารถปล่อยเหตุการณ์หรือข้อมูลออกมาได้เพียงครั้งเดียว ซึ่งเหตุการณ์นั้นจะเป็นได้แค่ onSuccess() หรือ onError() เท่านั้น (ไม่มี onComplete()) ซึ่งเมื่อเหตุการณ์ใดเหตุการณ์หนึ่งถูกเรียกแล้ว Single ก็จะหยุดการทำงานทันที ลองดู interface ของฝั่งรับ (observer) ประกอบเพื่อความเข้าใจที่มากขึ้นครับ
```java
interface SingleObserver<T> {
    void onSubscribe(Disposable d);
    void onSuccess(T value);
    void onError(Throwable error);
}
```
ดังนั้น Single จึงเหมาะกับงานที่ต้องการได้รับข้อมูลแค่ครั้งเดียวแล้วจบการทำงานเลย เช่น การเรียก API เป็นต้น ซึ่งหากใครที่ใช้ Retrofit อยู่แล้ว ก็จะมี retrofit adapter-rxjava ที่สามารถคืนค่ากลับมาเป็น Single หรือ Observable ได้ ตัวอย่างการเรียกใช้งาน Single สามารถดูได้จากโค้ดด้านล่างครับ
```kotlin
// full version
val singleObservable = RetrofitApi.getConfiguration()
singleObservable.subscribe(object : SingleObserver<Configuration> {
    override fun onSubscribe(disposable: Disposable) {
        // keep Disposable object for disposal 
    }
    override fun onSuccess(configs: Configuration) {
        // do some stuffs with data
    }
    override fun onError(error: Throwable) {
        // handle error
    }
})

// short version 
val singleObservable = RetrofitApi.getConfiguration()
singleObservable.subscribe({configs ->
    // do some stuffs with data
}, {error ->
    // handle error
})
```
### Completable
Completable คือ observable ที่คล้ายกับ Single เพียงแต่เหตุการณ์ที่ Completable สามารถปล่อยออกมาได้นั้น จะมีแค่ onComplete() และ onError() เท่านั้น (ไม่มี onNext()) ลองดู interface ของฝั่งรับ (observer) ประกอบครับ
```java
interface CompletableObserver<T> {
    void onSubscribe(Disposable d);
    void onComplete();
    void onError(Throwable error);
}
```
ดังนั้น Completable จึงเหมาะกับงานที่ไม่ต้องการการคืนค่าของข้อมูล หรือเหมาะกับงานที่ต้องการทราบแค่เพียงสถานะว่า complete หรือ error เท่านั้น ตัวอย่างงานประเภทนี้ได้แก่ การ save ข้อมูลลงไฟล์ หรือ การ insert ข้อมูลลง database เป็นต้น ลองดูตัวอย่างการใช้งานครับ
```kotlin
// Full version
val completable: Completable = repository.updateUser(user)
completable.subscribe(object : CompletableObserver{      
    override fun onSubscribe(d: Disposable) {
        // do something when subscribe
    }
    override fun onComplete() {
        // do some stuff when update success
    }
    override fun onError(e: Throwable) {
        // do some stuff when update error
    }
})

// Short version
val completable: Completable = repository.updateUser(user)
completable.subscribe({
    // do some stuff when update success     
}, { e: Throwable ->
    // do some stuff when update error     
})
```
### Maybe
Maybe คือ Observable ที่รวมความสามารถของ Single และ Completable เข้าไว้ด้วยกัน นั้นคือ Maybe สามารถส่งข้อมูลออกมาผ่านทาง onSuccess() ได้ 1 ข้อมูลเหมือน Single (ซึ่งจะไม่เรียก onComplete()) หรือไม่ส่งข้อมูลออกมาเลย แล้วจบการทำงานด้วยการเรียก onComplete() เหมือน Completable ลองดู interface ของฝั่งรับ (observer) ประกอบเพื่อความเข้าใจมากยิ่งขึ้นครับ
```java
interface MaybeObserver<T> {
    void onSubscribe(Disposable d);
    void onSuccess(T value);
    void onComplete();
    void onError(Throwable error);
}
```
ดังนั้น Maybe จึงเหมาะกับงานที่เราไม่แน่ใจว่าจะมีการคืนค่ากลับมาหรือไม่ เช่น การ query database เป็นต้น ลองดูตัวอย่างการใช้งานข้างล่างครับ
```kotlin
// Full version
val maybeObservable: Maybe<User> = repository.getUserById(101)
maybeObservable.subscribe(object : MaybeObserver<User>{
    override fun onSubscribe(d: Disposable) {
        // do something on subscribe
    }
    override fun onSuccess(user: User) {
        // this method will call when It has data
    }
    override fun onComplete() {
        // this method will call when it has no data 
    }
    override fun onError(e: Throwable) {
    	// handle error 
    }
})

// Short version
val maybeObservable: Maybe<User> = repository.getUserById(101)
maybeObservable.subscribe({ user: User ->
    // this method will call when It has data
}, { e: Throwable ->
    // handle error 
}, {
    // this method will call when it has no data 
})
```
### Flowable
Flowable ทำงานเหมือน Observable ทุกอย่าง ตั้งแต่ความสามารถในการส่งข้อมูลออกมาทาง onNext() จนถึงจบการทำงานด้วยการเรียก onComplete() หรือ onError() แต่เหตุที่ต้องมี Flowable เนื่องมาจากมีสิ่งที่เรียกว่า backpressure เข้ามาเกี่ยวข้อง ซึ่งผมจะไม่ขอลงลึกเกี่ยวกับ backpressure ว่าคืออะไร แต่จะขออธิบายสั่นๆว่ามันคือกระบวนการที่ไว้ชะลอการปล่อยข้อมูลออกมาจากฝั่งส่ง (Observable) ในกรณีที่ผู้ผลิต (Producer) ผลิตข้อมูลออกมาเร็วเกินไป จนฝั่งรับ (Consumer) ซึ่งในที่นี้คือ Observer ไม่สามารถประมวณผลข้อมูลเหล่านั้นได้ทันหรือได้เร็วพอ <br/>
ตัวอย่างของเหตุการณ์นี้ได้แก่ กรณีที่เราต้องการจัดการกับอะไรบางอย่างเมื่อมีเหตุการณ์การกดปุ่มเกิดขึ้น แน่นอนว่าเราไม่สามารถบอกผู้ใช้ให้ช่วยกดปุ่มให้ช้าลงหน่อยเพื่อรอให้แอปฯของเราประมวณผลเสร็จก่อน หรืออาจจะมีบางคนแย้งว่างั้นก็ disable ปุ่มซะเลยสิจะได้ไม่ต้องมีเหตุการณ์การกดปุ่มรัวๆ ซึ่งก็อาจจะทำแบบนั้นได้ครับ แต่อยากให้ลองนึกถึงเหตุการณ์อื่นที่เราไม่สามารถควบคุมการปล่อยข้อมูลออกมาจากฝั่งผู้ผลิต (Producer) ได้เช่น กรณี location update เป็นต้น ซึ่งจริงๆแล้วสิ่งที่จะบอกคือ เราไม่สามารถควบคุมคนที่ผลิตข้อมูลได้ (data source) แต่เราสามารถควบคุมคนกลางที่คอยปล่อยข้อมูลออกมาได้นั้นคือ Observable ของเรานั้นเอง ซึ่งหากเราไม่มีการะบวนการในการชะลอการปล่อยข้อมูลออกมาในฝั่ง Observable แล้วละก้อ สิ่งที่เราจะเจอก็คือ MissingBackpressureException นั้นเอง
```java
interface Subscriber<T> {
    void onNext(T t);
    void onComplete();
    void onError(Throwable t);
    void onSubscribe(Subscription s);
}

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
อีกสิ่งหนึ่งที่แตกต่างกันระหว่าง Flowable และ Observable นั้นก็คือ Interface ของฝั่งรับ โดยที่ฝั่งรับของ Flowable จะเป็น Subscriber ส่วนฝั่งรับของ Observable จะเป็น Observer ซึ่งถึงแม้ว่าทั้งสองจะมีหน้าตาของ method ที่คล้ายกัน แต่จะแตกต่างกันตรงที่หลักการและ Design pattern ที่นำมาใช้ โดยที่ Flowable จะมีการใช้หลักการของ Producer-Consumer Pattern ดังที่ได้กล่าวไว้แล้วใน [บทความ](https://github.com/kingland/RxJavaThai/blob/master/README.md) เข้ามาช่วยด้วย ลองมาดูตัวอย่างการใช้งานของ Flowable ด้านล่างกันครับ
```kotlin
// create flowable
val flowable = Flowable.create(FlowableOnSubscribe { emitter: FlowableEmitter<MotionEvent> ->
    // do sum stuff 
    ...
    emitter.onNext(motionEvent)
}, BackpressureStrategy.BUFFER)

// Subscribe in full version
flowable.subscribe(object : Subscriber<MotionEvent> {
    override fun onSubscribe(s: Subscription) {
        // do some stuff with subscription
    }
    override fun onNext(event: MotionEvent) {
        // do some stuff with data
    }
    override fun onComplete() {
        // do some stuff
    }
    override fun onError(t: Throwable) {
        // handle error
    }
})

// Subscribe in short version
flowable.subscribe({ event: MotionEvent -> 
    // do some stuff with data
}, { t: Throwable ->
    // handle error
}, {
    // do some action when complete
}, {
    // do some stuff with subscription
})
```
### Update
ใน RxJava 2.x ได้มีการเพิ่มความสามารถให้กับ Operator Observable.observeOn() ทำให้ไม่มีปัญหาเรื่อง MissingBackpressureException เกิดขึ้นเมื่อใช้ Observable ลองดู link ด้านล่างประกอบครับ
<br/>
[Observable.observeOn](https://github.com/ReactiveX/RxJava/issues/4726)
<br/>
[Observable backpressure](https://stackoverflow.com/questions/44674174/rxjava2-observable-backpressure)
<br/>
[Observable backpressure](https://github.com/ReactiveX/RxJava/wiki/Backpressure-%282.0%29)
## Conclusion
จากบนความข้างต้น เราได้เรียนรู้ว่า Observable และ Observer คืออะไร และมี Obsevable ประเภทไหนบางให้เราเรียกใช้งานและแต่ละประเภทแตกต่างกันอย่างไร โดยจะขอสรุปเป็นข้อๆ ได้ดังนี้
* **`Observable`** สามารถส่งข้อมูลออกมาได้ตั้งแต่ 0 item จนถึง N items โดยส่งข้อมูลผ่านทาง onNext() และจบการทำงานทันที เมื่อเรียก onError() หรือ onComplete()
* **`Single`** ปล่อยเหตุการณ์หรือข้อมูลออกมาได้เพียงครั้งเดียว ซึ่งเป็นได้แค่ onSuccess() หรือ onError() เท่านั้น
* **`Completable`** สามารถส่ง event ออกมาได้เพียง onComplete() และ onError() เท่านั้น
* **`Maybe`**: คือ Observable ที่รวมความสามารถของ Single และ Completable เข้าไว้ด้วยกัน
* **`Flowable`** ทำงานเหมือน Observable ทุกอย่าง เพียงแต่มีเรื่อง backpressure เข้ามาเกี่ยวข้อง
* **`backpressure`** คือกระบวนการที่ไว้ชะลอการปล่อยข้อมูลออกมาจากฝั่งส่ง