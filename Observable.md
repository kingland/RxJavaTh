## Create Observable
### just()
```kotlin
Flowable.just("Hello");

Flowable.just("Hello", "World");

Observable.just("Hello");

Observable.just("Hello", "World");

Maybe.just("Hello"); // can be only one data

Single.just("Hello"); // can be only one data
```
เริ่มต้มการสร้าง Observable ง่ายๆ ด้วยการใช้ `just()` โดย `just()` จะเหมาะกับกรณีที่เรามีข้อมูลพร้อมอยู่แล้ว และต้องการสร้าง observable ขึ้นมาจากข้อมูลเหล่านั้น ซึ่งจากตัวอย่างด้านบนจะเห็นว่า แทบจะทุกประเภทของ Observable มีฟังก์ชั่น `just()` ให้เราเรียกใช้ แต่อาจจะมีเงื่อนไขการใช้งานที่แตกต่างกันใน Observable แต่ละประเภท ซึ่งได้กล่าวไปแล้วในบทความที่สอง
<br/>
จากตัวอย่างอาจทำให้เราเข้าใจผิดได้ว่า `observable.just()` รับข้อมูลในรูปแบบของ Array Argument (คือใส่ข้อมูลในรูปแบบของ argument เท่าไรก็ได้ โดยข้อมูลนั้นจะถูกมองเป็น array) แต่จริงๆแล้วไม่ใช่ ซึ่งเมื่อเราเข้าไปดูการทำงานจริงๆของ `observable.just()` แล้วจะเห็นว่า มี method overloading ที่สามารถรับข้อมูลได้สูงสุดเพียง 10 ตัวเท่านั้น และด้วยการที่ `observable.just()` นั้นประกาศรับข้อมูลเป็นแบบ Generic Type ทำให้เมื่อเราใส่ข้อมูลชนิดใดลงไป `observable.just()` ก็จะปล่อยข้อมูลชนิดนั้นออกมา นั้นหมายความว่า หากเราใส่เป็น String ก็จะปล่อยข้อมูลออกมาเป็น String แต่หากเราใส่ข้อมูลเป็น Array<String> ก็จะปล่อยข้อมูลออกมาเป็น Array<String> ไม่ใช่ String แต่ละตัวที่อยู่ภายใน array นั้น
<br/>
ข้อควรระวังที่สำคัญของการใช้ `Observable.just()` อีกอย่างหนึ่งก็คือ ข้อมูลที่ใส่เข้าไปจะต้องมีพร้อมอยู่แล้ว หรือไม่ควรใช้เวลานานในการได้มาซึ่งข้อมูล นั้นหมายความว่าเราไม่สามารถใช้ `Observable.just()` กับเหตุการณ์ด้านล่างนี้ได้
```kotlin
val observable = Observable.just(getDataFromApi()).subscribeOn(Schedulers.io())
```
จากโค้ดด้านบน `equal`getDataFromApi() ถูกสั่งให้ทำงานทันทีตอนที่เรากำลังสร้าง observable ขึ้นมาด้วย คำสั่ง `just()` โดย `getDataFromApi()` จะทำงานที่ thread เดียวกับ caller ถึงแม้ว่าเราจะสั่งให้ observerble นั้นทำงานที่ IO thread ด้วยคำสั่ง `subscribeOn()` แล้วก็ตาม (คำสั่ง `subscribeOn()` จะกล่าวถึงในบทความถัดไป) ซึ่งหากเราลองเอาโค้ดด้านบนมาเขียนใหม่ เราสามารถเขียนได้ดังนี้
```kotlin
val data = getDataFromApi()
val observable = Observable.just(data).subscribeOn(Schedulers.io())
```
ซึ่งจริงๆแล้วจะเห็นว่า มันคือการไปเรียก getDataFromApi() เพื่อเตรียมข้อมูลให้พร้อมก่อน ก่อนที่จะใส่ข้อมูลเหล่านั้นให้กับ Observable.just() นั้นเอง
### fromCallable()
```kotlin
Flowable.fromCallable{ "Hello" }

Observable.fromCallable{ "Hello" }

Maybe.fromCallable{ "Hello" }

Single.fromCallable{ "Hello" }

Completable.fromCallable{ "Ignored!" } // ignore result
```
`fromCallable()` คือวิธีที่หนึ่งที่ไว้ใช้สร้าง Observable แบบง่ายๆ ด้วยการเอา `fromCallable()` ไปครอบโค้ดที่เราต้องการ `fromCallable()` นั้นเหมาะกับงานในลักษณะที่เป็น Synchronous ที่คืนค่าเพียงค่าเดียว โดยจะปล่อยข้อมูลออกมาผ่านการเรียก retrun ซึ่งจะถูกแปลงเป็นการเรียก `onNext()` และต่อด้วย `onComplete()` ให้เองอัตโนมัติ ตัวอย่างของงานประเภทนี้ได้แก่ การอ่าน/เขียนไฟล์ หรือการคำนวนที่ต้องใช้เวลานานๆ เป็นต้น โดยเมื่อครอบงานเหล่านั้นด้วย `fromCallable()` แล้ว งานเหล่านั้นจะยังไม่ถูกทำจนกว่าจะมี Observer มา subscribe และเพื่อป้องกันการ Block UI Thread ที่อาจเกิดขึ้นได้จากการประมวณผลที่ใช้เวลานาน เราสามารถกำหนด thread ในการทำงานให้กับงานเหล่านั้นผ่านทาง Observable ที่สร้างขึ้นได้ นอกจากนี้ `fromCallable()`ยังอนุญาติให้เรา throw exception จากภายใน callable เมื่อเกิดข้อผิดพลาดขึ้น ซึ่งจะเห็นว่าทั้งรูปแบบของการ retrun และการ throw exception นั้นจะคล้ายกับที่เราทำใน Java นั้นเองครับ ลองดูตัวอย่างของการใช้ `fromCallable()` ด้านล่างประกอบครับ
```kotlin
// create Observable
val fromCallableObservable = Observable.fromCallable<String> {
    val bitmap = //...
    val imagePath = //...
    var path: String? = writeBitmapToFile(bitmap, imagePath)
    return@Callable path ?: ""
}

...
// when subscribe 
fromCallableObservable.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe { path -> 
            //do some stuffs
}
```
### fromAction() & fromRunnable()
```kotlin
Maybe.fromAction(() -> {...});

Maybe.fromRunnable(() -> {...});

Completable.fromAction(() -> {...});

Completable.fromRunnable(() -> {...});
```
เมื่อพูดถึง `fromCallable()` ไปแล้วถ้าจะไม่พูดถึง `fromAction()` และ `fromRunnable()` ก็คงจะไม่ได้ สองอย่างนี้ทำหน้าที่คล้ายกับ `fromCallable()` แต่จะแตกต่างจาก `equal`fromCallable() ตรงที่ `fromAction()` และ `fromRunnable()` จะไม่คืนค่าใดๆ หรือปล่อยค่าใดๆออกมาให้กับ Observer และมีให้ใช้เฉพาะใน Maybe และ Completable เท่านั้น ซึ่งเราจะมักจะใช้ในกรณีที่ต้องการทำงานอะไรสักอย่างก่อนที่ onComplete จะถูกเรียก หรือใช้ในงานที่ต้องการทราบแค่เพียงว่า Success (`onComplete()`) หรือ Fail (`onError()`) เช่น การบันทึกข้อมูลลง Database เป็นต้น ลองดูตัวอย่างการใช้งานของ `Completable.fromAction()` ครับ
```kotlin
// create observable
val addressList = //...
val completable = Completable.fromAction {
    RealmHelper.saveAddressList(addressList)
}

// when subscribe 
completable.subscribe({
    // this method will be called when action is success
}, { error -> 
    // this method will be called when action has error
})
```
### Create()
```kotlin
Flowable.create<String>({emitter -> { … } }, BUFFER)

Observable.create<String>{emitter -> { … } }

Maybe.create<String>{emitter -> { … } }

Single.create<String>{emitter -> { … } }

Completable.create{emitter -> { … } }
```
การสร้าง Observable ด้วย `create()` เป็นการสร้าง Observable ที่ powerful มากที่สุด เราสามารถกำหนดจังหวะในการปล่อยเหตุการณ์ต่างๆออกมาเองได้ ผ่านสิ่งที่เรียกว่า emitter โดยเราสามารถส่งข้อมูลออกมาได้ผ่านทาง `emitter.onNext()`, ส่ง error ได้ผ่านทาง `emitter.onError()` และจบการทำงานของ Observable ด้วยการเรียก `emitter.onComplete()` ซึ่งข้อดีของการสร้าง observable ด้วยวิธีนี้คือ เราสามารถเรียก `onNext()` กี่ครั้งก็ได้ จากนั้นจึงค่อยสั่ง `onComplete()` เพื่อจบการทำงาน ซึ่งต่างจาก `fromCallable()` ตรงที่จะยิงข้อมูลออกมาแค่ครั้งเดียว แล้วเรียก `onComplete()` ให้ทันที
```kotlin
// Create observable
val observable = Observable.create<String> { emitter ->
    emitter.onNext("Hello")
    emitter.onNext("World")
    emitter.onComplete()
}

// when subscribe 
observable.subscribe({ str ->
    // deal with data
}, { error ->
    // handle error 
}, {
    // do something when complete 
})
```
ข้อดีอีกอย่างหนึ่งคือ เราสามารถใช้ `create()` เพื่อสร้าง Observable ที่ใช้จัดการกับการทำงานแบบ Asynchronous ได้ง่ายขึ้น ยกตัวอย่างเช่น หากเราต้องการดึงข้อมูลด้วย HTTP เราสามารถเอา `observable.create()` ไปครอบกระบวนการเรียก API นั้นไว้ และเมื่อเราได้รับข้อมูลกลับมา ก็สามารถเรียก onNext ได้จากภายใน callback ของ HTTP เพื่อส่งข้อมูลที่ได้ออกไปให้กับ Observer ลองดูตัวอย่างประกอบครับ
```kotlin
OkHttpClient client = // …
Request request = // …

Observable.create<String> { emitter ->
    val call = client.newCall(request)
    emitter.setCancellable { call.cancel() }
    call.enqueue(object : Callback() {
        
        @Throws(IOException::class)
        fun onResponse(r: Response) {
            emitter.onNext(r.body().string())
            emitter.onComplete()
        }

        fun onFailure(e: IOException) {
            emitter.onError(e)
        }
    })
}
```
ในบางครั้ง ด้วยการทำงานแบบ Asynchronous ทำให้เราไม่สามารถคาดเดาได้ว่างานนั้นจะทำเสร็จเมื่อไร อีกทั้งยังมีโอกาสสูงที่แอปฯจะหยุดการทำงานก่อนที่งานนั้นจะทำเสร็จ การใช้ `observable.create()` จะช่วยให้เราสามารถหยุดการทำงานของ Asynchronous เหล่านั้นได้เมื่อ Observable หยุดการทำงานลง โดยเราสามารถจัดการได้ผ่านทางการเรียกใช้ `emitter.setCancelable()` ลองดูตัวอย่างด้านบนประกอบครับ
<br/>
แต่อย่างไรก็ตาม การที่มัน powerful มากเกินไปก็อาจกลายเป็นข้อเสียของมันได้เช่นกัน เนื่องจากเราต้องกลายเป็นคนที่เรียก `onNext()` และ `onComplete()` เอง (รวมถึง `onError()` ในบางครั้งด้วย) ซึ่งต่างจาก `fromCallable()` ตรงที่การคืนค่า (retrun) คือการเรียก `onNext()` และ `onComplete()` ให้เองอัตโนมัติ ทำให้ในบางครั้งเราอาจจะลืมเรียกบางขั้นตอน หรือเรียก `onComplete()` ก่อน `onNext()` ซึ่งก็จะทำให้เกิดข้อผิดพลาดของการทำงานได้ ดังนั้นจึงต้องระมัดระวังขณะใช้งานด้วย
## Subscription & Disposable
หลังจากที่เราเรียนรู้วิธีการสร้าง Observable รวมถึงวิธีการใช้กันไปแล้ว ทีนี้เรามาเรียนรู้วิธีการหยุดการทำงานของมันบ้าง ในบทความที่แล้วเราได้พูดถึง Interface ของฝั่งรับของทั้ง Flowable และ Observable กันไปแล้ว ซึ่งจะขอยก Interface นั้นมาให้ดูกันอีกสักรอบครับ
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
เราจะสังเกตุเห็นว่ามี Interface ที่ชื่อว่า Disposable และ Subscription ซึ่งเจ้าสองสิ่งนี้คือตัวที่เอาไว้หยุดการทำงานของ Observable และ Flowable ตามลำดับ โดยมี method dispose และ cancel ให้เราเรียกใช้เพื่อหยุดการทำงาน ซึ่งหากเทียบกับเหตุการณ์ที่เรา observe การกดปุ่มที่เคยยกตัวอย่างไปแล้วในบทความที่สอง การเรียก dispose หรือ cancel คือการที่เราบอกกับฝั่งส่ง (Observable/Flowable) ว่า “พอแล้ว ฉันไม่ต้องรับ touch event อีกแล้ว” นั้นเอง
<br/>
โดยปกติแล้วเมื่อเรา subscribe Observable หรือ Flowable แล้วเราจะได้รับ Object ที่เป็น Disposable หรือ Subscription กลับมาด้วย ซึ่งเราควรจะเก็บ Object เหล่านี้ไว้ เพื่อเอาไว้สั่งหยุดการทำงานของ Observable หรือ Flowable เมื่อจบการทำงาน
<br/>
สำหรับ Observable นั้น เรามักจะใช้ CompositeDisposable เข้ามาช่วยในกระบวนการสั่งหยุดการทำงานของ Observable ด้วย โดยเราสามารถสั่ง `compositeDisposable.add()` เพื่อใส่เจ้า Disposable Object ที่ได้จากการ subscribe เข้าไปเตรียมเอาไว้ก่อน โดยจะใส่เข้าไปกี่ตัวก็ได้ จากนั้นจึงค่อยสั่ง `compositeDisposable.clear()` เพื่อหยุดการทำงานของ Observable เหล่านั้น ซึ่งวิธีนี้ทำให้เราสามารถจัดการกับหลายๆ Observable ได้ในคราวเดียว ลองดูตัวอย่างด้านล่างประกอบครับ
```kotlin
val disposable1 = Observable.create<String> { emitter -> 
    emitter.onNext("Hello")
    emitter.onComplete()
}.subscribe { str -> 
    // do something with data
}

val disposable2 = Observable.create<Int> { emitter ->
    emitter.onNext(1)
    emitter.onComplete()
}.subscribe { int ->
    // do something with data
}

val disposeBag = CompositeDisposable()
disposeBag.add(disposable1)
disposeBag.add(disposable2)

...

//clear when you want to terminate all observable 
disposeBag.clear()
```
การ dispose observable ทุกครั้งถือเป็น Best practice ที่ควรทำ ถึงแม้ว่า Observable นั้นจะหยุดการทำงานไปแล้วด้วยการเรียก onComplete หรือ onError แล้วก็ตาม แต่เราก็ควรจะทำการเรียก dispose observable เหล่านั้นให้ติดเป็นนิสัย มิฉะนั้นอาจมีโอกาศที่จะเกิด memory leak ได้
## Conclusion
จากบทความข้างต้น เราได้เรียนรู้วิธีการสร้าง Observable ด้วยวิธีต่างๆไปแล้ว รวมถึงเรียนรู้วิธีการสั่งให้ Observable หยุดการทำงาน โดยจะข้อสรุปออกมาเป็นข้อๆให้อ่านดังนี้
* `Just()` คือวิธีการสร้าง Observable ที่เหมาะกับกรณีที่เรามีข้อมูลพร้อมอยู่แล้ว และต้องการสร้าง Observable ขึ้นมาจากข้อมูลเหล่านั้น
* `fromCallable()` เหมาะกับงานในลักษณะที่เป็น Synchronous ที่คืนค่าเพียงค่าเดียว
* งานที่ `fromCallable()` ครอบอยู่จะยังไม่ถูกทำจนกว่าจะมี Observer มา subscribe
* `fromAction()` & `fromRunable()` ทำงานคล้ายกับ fromCallable แต่จะไม่มีการคืนค่าหรือปล่อยค่าใดๆออกมา
* `create()` คือ วิธีการสร้าง Observable ที่มีความยืดหยุ่นสูงมาก
* `Subscription` & `Disposable` คือ Interface ที่มีไว้เพื่อหยุดการทำงานของ Observable
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 