- [ReactiveX คือ](https://github.com/kingland/RxJavaThai/blob/master/README.md)
- [Observer & Observable](https://github.com/kingland/RxJavaThai/blob/master/Observer%26Observable.md)
- [สร้าง Observable](https://github.com/kingland/RxJavaThai/blob/master/Observable.md)
- [Hot Observable Vs Cold Observable](https://github.com/kingland/RxJavaThai/blob/master/Hot%26ColdObservable.md)
- [Observer Vs Subscriber Vs Consumer](https://github.com/kingland/RxJavaThai/blob/master/ObserverSubscriberConsumer.md)
- [observeOn Vs subscribeOn](https://github.com/kingland/RxJavaThai/blob/master/SubscribeOn%26ObserveOn.md)
- [Operator ใน ReactiveX](https://github.com/kingland/RxJavaThai/blob/master/Operators.md)
- [Subject คือ](https://github.com/kingland/RxJavaThai/blob/master/Subject.md)

หลังจากที่เราได้เรียนรู้ Operators พื้นฐานบางตัวทีเราควรรู้จักกันไปแล้ว ยังมีอีกสอง Operators ที่อยากจะให้ผู้อ่านได้รู้จัก นั้นคือ subscribeOn() และ observeOn() เนื่องจากมันถูกใช้บ่อยมาก และต้องอาศัยความเข้าใจในระดับหนึ่งในการเรียกใช้ จึงได้แยกออกมาเป็นเป็นอีกบทความนึง เพื่อให้ผู้อ่านได้ทำความเข้าใจกับมันได้มากขึ้นครับ

จริงๆแล้ว บทความเกี่ยวกับ subscribeOn() และ observeOn() นั้นผมได้เคยเขียนไว้แล้วในบทความนี้ แต่เนื่องจากต้องการรวบรวมบทความที่เกี่ยวข้องกับ Rxjava2 ใหม่ จึงขอนำบทความเหล่านั้นมารวบรวมไว้ในที่เดียว ดังนั้นโค้ดตัวอย่างในบทความนี้จึงเขียนด้วย Java ครับ

ในบทความที่ผ่านมา เราได้เห็นโค้ดตัวอย่างที่มีการใช้ subscribeOn() และ observeOn() กันไปบ้างแล้ว มาในบทความนี้เราจะมาลงลึกกันสักนิดว่าเจ้า SubscribeOn() กับ ObserveOn() ใน RxJava ที่เราใช้กันอยู่มันต่างกันยังไง และจะใช้มันตอนไหน

## Thread
ก่อนจะพูดถึง SubscribeOn() และ ObserveOn() ก็ขอเกริ่นนำเรื่อง thread กันสักเล็กน้อย หลายๆแอปฯที่เราทำกันทุกวันนี้ ล้วนมีการเชื่อมต่อกับอินเทอร์เน็ต ซึ่งเรารู้ดีว่าใน Android หากเราเชื่อมต่ออินเทอร์เน็ตภายใต้ UI thread (main thread) เราจะเจอกับปัญหา Network on mainthread exeption ที่ตัว Android พ่นออกมา ทั้งนี้เนื่องจากการโหลดข้อมูลจากอินเทอร์เน็ตนั้น ผู้ใช้งานอาจจะต้องใช้เวลาดาวน์โหลดนานเพราะปัญหาอินเตอร์เน็ตช้าหรือข้อมูลมีขนาดใหญ่ อย่างเช่นรูปภาพ ซึ่งการที่เราเชื่อมต่ออินเทอร์เน็ตภายใต้ UI thread อาจทำให้เกิดเหตุการณ์หน้าจอค้างขึ้น หรือที่เราเรียกกันว่า ANR (Android Not Responding) ซึ่งไม่ส่งผลดีต่อผู้ใช้เป็นแน่ ไม่เพียงแต่การดาวน์โหลดข้อมูลผ่านอินเทอร์เน็ตเท่านั้นที่ทำให้เกิด ANR การที่เราประมวลผลข้อมูลบางอย่างบน UI thread นานๆก็มีผลทำให้เกิด ANR เช่นกัน วิธีแก้คือให้แยกการทำงานออกไปเป็นอีก thread และเมื่อได้ผลลัพธ์ก็ส่งกลับคืนมาแสดงผลที่ UI thread ซึ่งใน Android เองก็มีสิ่งที่เรียกว่า AsynTask ให้เราได้ใช้กัน แต่ถ้าหากพูดถึง RxJava แล้ว SubscribOn() และ ObserveOn() คือสิ่งที่เข้ามาช่วยแก้ปัญหาตรงจุดนี้

โดยปกติแล้วการทำงานของ observable และ subscriber ใน RxJava จะทำงานใน thread เดียวกันกับ thread ของผู้เรียก ซึ่งในบทความนี้จะขอเรียกว่า caller thread นั่นหมายความว่า หากเรา subscribe ใน main thread ตัว observable และตัว subscriber ก็จะทำงานใน main thread แต่หากเราต้องการเปลี่ยน thread การทำงานให้กับ observable และ subscriber เราสามารถกำหนดได้ผ่านทาง subscribeOn() และ ObserveOn()

## SubscribeOn()
เป็นคำสั่งที่ไว้กำหนดว่าจะให้ observable ปล่อยข้อมูลออกจาก thread ไหน (To instructs the source Observable which thread to emit items on) หรือจะพูดให้เข้าใจง่ายขึ้นอีกนิดคือ เป็นการกำหนดการทำงานของ observable ว่าจะให้ทำงานที่ thread ไหนนั้นเอง ซึ่งหากเราไม่กำหนด ตัว observable ก็จะทำงานที่ thread เดียวกันกับ caller thread ลองดูตัวอย่างการใช้งานด้านล่างครับ
```kotlin
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        Log.d("RxJava", "emit from thread: " + Thread.currentThread().getName());
        emitter.onNext("result");
        emitter.onComplete();
    }
});

observable.subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
    }
});
```
```text
emit from thread: main
received on thread: main
```
จาก code ด้านบนจะเห็นว่าผมไม่ได้ใส่ subscribeOn() ให้กับ Observable เพื่อต้องการชี้ให้เห็นว่า หากไม่กำหนด threadให้กับ observable มันจะทำงานที่ maintread ซึ่งเป็น thread เดียวกับ caller นั้นเอง

ต่อไปเราลองใส่ subscribeOn() ให้กับ Observable เพื่อให้ไปทำงานที่ I/O thread แทน
```kotlin
observable.subscribeOn(Schedulers.io())
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
            }
        });
```
ผลลัพธ์ที่ได้ออกมาเป็นดังนี้
```text
emit from thread: RxCachedThreadScheduler-1
received on thread: RxCachedThreadScheduler-1
```
จะเห็นว่าตัว observable ถูกเปลี่ยนให้ไปทำงานอยู่บน thread อื่นที่ไม่ใช่ main thread และส่งผลให้ตัว subscriber ทำงานอยู่บน thread เดียวกับ observable เช่นกัน (สังเกตุได้จาก received on thread: RxCachedThreadScheduler-1)
## ObserveOn()
จากกรณีข้างบน หากต้องการนำข้อมูลที่ได้จาก observable ไปแสดงผลบน UI เราจำเป็นที่จะต้องเปลี่ยน thread การทำงานของ Subscriber ให้กลับมาทำงานที่ main thread ซึ่งการใช้คำสั่ง observeOn() จะช่วยให้เราสามารถสลับ thread การทำงานของตัว Subscriber ได้ ลองดูตัวอย่างประกอบครับ
```kotlin
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        Log.d("RxJava", "emit from thread: " + Thread.currentThread().getName());
        emitter.onNext("result");
        emitter.onComplete();
    }
});

observable.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
            }
});
```
```text
emit from thread: RxCachedThreadScheduler-1
received on thread: main
```
ซึ่งจะเห็นว่า subscriber ได้เปลี่ยนกลับมาทำงานที่ main thread โดยที่ observable ยังคงทำงานอยู่ใน I/O thread
## Schedulers
จากตัวอย่างด้านบน เราจะเห็นว่าตอนที่สั่ง subscribeOn() หรือ observeOn() เราได้กำหนดค่าบางอย่างเข้าไปด้วย ซึ่งเรียกว่า scheduler โดย scheduler จะเป็นตัวที่ระบุว่าจะให้ทำงานที่ thread ไหน โดย scheduler นั้นมีหลายตัวให้เราเลือกใช้ ซึ่งจะขอยกตัวอย่างบางตัวที่ใช้กันบ่อยๆมาให้ดูครับ
* Schedulers.io(): เหมาะสำหรับงานที่ต้องยุ่งเกี่ยวกับ I/O เช่นการเชื่อมต่ออินเทอร์เน็ต การอ่านหรือเขียนไฟล์
* Schedulers.computation(): เหมาะกับงานที่ต้องประมวลผลเป็นเวลานาน เช่น for-loop ขนาดใหญ่ หรือการจัดการกับข้อมูลจำนวนมาก
* AndroidSchedulers.mainThread(): ใช้เมื่อต้องการเปลี่ยนให้มาทำงานที่ main thread
## ใช้อย่างไร ใช้เมื่อไร
หลายคนอาจสงสัยว่า เมื่อไรควรใช้ subscribeOn()? เมื่อไรควรใช้ observeOn()? ใช้ subscribeOn() อย่างเดียวได้มั้ย? หรือใช้ observeOn() อย่างเดียวได้มั้ย? ตำแหน่งการใช้มีผลรึป่าว? จากตัวอย่างข้างต้น ผมได้แสดงตัวอย่างการใช้ subscribeOn() และ observeOn() ให้เห็นกันไปบ้างแล้ว แต่ขอเพิ่มเติมอีกสักนิดว่าเราควรใช้มันอย่างไรและเมื่อไร
### subscribeOn()
subscribeOn() ถูกกำหนดได้เพียงครั้งเดียวเท่านั้นโดยจะยึดการกำหนดครั้งแรกเป็นหลัก นั้นหมายความว่าต่อให้กำหนด subscribeOn() หลายครั้งก็จะยึดคำสั่งแรกเป็นหลัก ลองดูตัวอย่างครับ
```kotlin
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        Log.d("RxJava", "emit from thread: " + Thread.currentThread().getName());
        emitter.onNext("result");
        emitter.onComplete();
    }
});

observable.subscribeOn(Schedulers.io())
        .subscribeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
            }
        });
```
```text
emit from thread: RxCachedThreadScheduler-2
received on thread: RxCachedThreadScheduler-2
```
จาก code ด้านบนจะเห็นว่า Observable ได้สั่ง subscribeOn() ถึงสองครั้ง โดยครั้งแรกสั่งให้ทำงานที่ I/O thread และในครั้งที่สองสั่งให้ทำงานที่ main thread แต่จากผลลัพท์จะเห็นได้ว่า Observable ไม่ได้เปลี่ยนการทำงานไปที่ main thead โดยมันยังคงทำงานอยู่ที่ I/O thread อยู่เหมือนเดิม ซึ่งแสดงให้เห็นว่าsubscribeOn()จะยึดคำสั่งแรกเพียงคำสั่งเดียวในการกำหนดการทำงานให้กับ observable เท่านั้น

เราใช้ subscribeOn() เพื่อกำหนด thread ในการทำงานให้กับ observable ซึ่งโดยส่วนใหญ่แล้วเรามักจะกำหนดให้ทำงานบน I/O thread หรือ computation thread ก็ต่อเมื่อเราเห็นว่างานที่ต้องทำใน observable นั้นใช้เวลานานและเราไม่ต้องการให้ทำงานใน main thread ตัวอย่างของงานเหล่านั้นได้แก่ การเชื่อมต่ออินเทอร์เน็ต และการเขียนข้อมูลลงไฟล์
### observeOn()
observeOn() สามารถกำหนดได้หลายครั้ง โดยที่การกำหนดในแต่ละครั้งจะส่งผลให้เปลี่ยนการทำงานของ thread ตามที่กำหนด ลองดูตัวอย่างด้านล่างครับ
```kotlin
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        Log.d("RxJava", "emit from thread: " + Thread.currentThread().getName());
        emitter.onNext("result");
        emitter.onComplete();
    }
});

observable.observeOn(Schedulers.io())
        .doOnNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "execute doOnNext on thread: " + Thread.currentThread().getName());
            }
        })
        .observeOn(Schedulers.computation())
        .doAfterNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {

                Log.d("RxJava", "execute doAfterNext on thread: " + Thread.currentThread().getName());
            }
        })
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
            }
});
```
```text
emit from thread: main
execute doOnNext on thread: RxCachedThreadScheduler-1
execute doAfterNext on thread: RxComputationThreadPool-1
received on thread: main
```
จากผลลัพธ์ข้างบนจะเห็นว่า เราสามารถสับเปลี่ยน thread ในการทำงานของตัว subscriber โดยที่ลำดับของคำสั่ง มีผลต่อการสลับการทำงานของ thread ไปมา

เรามักใช้ observeOn() เมื่อต้องการเปลี่ยน thread การทำงานจาก thread หนึ่งไปเป็นอีก thread หนึ่ง เช่นการเปลี่ยนให้ไปทำงานที่ main thread เมื่อต้องการอัพเดท UI หรือเปลี่ยนไปเป็น computation thread เมื่อต้องการประมวลผลข้อมูลที่ต้องใช้เวลานาน
## แต่ละคำสั่งควรจะกำหนดไว้ตรงไหนดี
หลายคนอาจสงสัยว่า จำเป็นต้องกำหนด subscripeOn() ไว้เป็นคำสั่งแรกไหม คำตอบคือ ไม่จำเป็นครับ subscripeOn() สามารถกำหนดไว้ตรงไหนก็ได้โดยที่ลำดับไม่มีผล ลองดูตัวอย่างครับ
```kotlin
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
        Log.d("RxJava", "emit from thread: " + Thread.currentThread().getName());
        emitter.onNext("result");
        emitter.onComplete();
    }
});

observable.observeOn(Schedulers.io())
        .doOnNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "execute doOnNext on thread: " + Thread.currentThread().getName());
            }
        })
        .observeOn(Schedulers.computation())
        .doAfterNext(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {

                Log.d("RxJava", "execute doAfterNext on thread: " + Thread.currentThread().getName());
            }
        })
        .observeOn(AndroidSchedulers.mainThread())
        .subscribeOn(Schedulers.io())
        .subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d("RxJava", "received on thread: " + Thread.currentThread().getName());
            }
});
```
```text
emit from thread: RxCachedThreadScheduler-1
execute doOnNext on thread: RxCachedThreadScheduler-2
execute doAfterNext on thread: RxComputationThreadPool-1
received on thread: main
```
จาก code จะเห็นว่าเราไม่ได้กำหนด subscribeOn(Schedulers.io()) ไว้เป็นคำสั่งแรก แต่การทำงานของ observable ก็ยังคงทำงานใน I/O thread ได้อย่างถูกต้องตามที่ระบุไว้ ซึ่งต่างจาก observeOn() ตรงที่ว่า หากเราสลับบรรทัด การทำงานของ thread ก็ถูกสลับไปด้วย
## ใช้ตัวใดตัวหนึ่งเพียงอย่างเดียวได้หรือไม่
คำตอบคือ อาจจะได้ หรือ ไม่ได้ ขึ้นอยู่กับว่าต้องการทำอะไร หากเราใช้แค่ subscribeOn() นั้นหมายความว่างานที่ทำในฝั่ง subscriber ก็จะทำงานใน thread เดียวกับ observable ด้วย แต่หากเราใช้แค่ observeOn() งานที่ทำใน observable ก็จะทำงานใน thread เดียวกับ caller thread ซึ่งจะเห็นได้ว่า มันขึ้นอยู่กับจุดประสงค์ของงานที่จะทำ
## conclusion
การเข้าใจการทำงานของ subscribeOn() และ observeOn() อย่างถูกต้องจะช่วยให้เราสามารถใช้ RxJava ได้อย่างมีประสิทธิภาพมากขึ้น ซึ่งจากบทความข้างต้นเราพอจะสรุปการทำงานของ subscribeOn() และ observeOn() ออกมาได้ดังนี้
* subscribeOn() คือการกำหนด thread การทำงานให้กับ observable และมีผลต่อ thread การทำงานของ subscriber (หากใช้ subscribeOn() เพียงอย่างเดียว)
* subscribeOn() กำหนดได้เพียงครั้งเดียวโดยยึดคำสั่งแรกที่กำหนด
* observeOn() ใช้เพื่อสับเปลี่ยน thread ในการทำงานของฝั่ง subscriber
* ลำดับของคำสั่ง observeOn() มีผลในการเปลี่ยน thread แต่ลำดับของคำสั่ง subscribeOn() จะไม่มีผล
* observable และ subscriber จะทำงานใน thread เดียวกันกับ caller thread หากไม่มีการกำหนด subscribeOn() หรือ observeOn()
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 
