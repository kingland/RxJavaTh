- [ReactiveX คือ](https://github.com/kingland/RxJavaThai/blob/master/README.md)
- [Observer & Observable](https://github.com/kingland/RxJavaThai/blob/master/Observer%26Observable.md)
- [สร้าง Observable](https://github.com/kingland/RxJavaThai/blob/master/Observable.md)
- [Hot Observable Vs Cold Observable](https://github.com/kingland/RxJavaThai/blob/master/Hot%26ColdObservable.md)
- [Observer Vs Subscriber Vs Consumer](https://github.com/kingland/RxJavaThai/blob/master/ObserverSubscriberConsumer.md)
- [observeOn Vs subscribeOn](https://github.com/kingland/RxJavaThai/blob/master/SubscribeOn%26ObserveOn.md)
- [Operator ใน ReactiveX](https://github.com/kingland/RxJavaThai/blob/master/Operators.md)
- [Subject คือ](https://github.com/kingland/RxJavaThai/blob/master/Subject.md)
## Observable & Producer
ก่อนที่จะไปอธิบายถึงความหมายของ Cold Observable และ Hot Observable ก็อยากจะมาทบทวนความหมายของ Observable และ Producer กันซะก่อน โดยอย่างที่ได้บอกไปแล้วในบทความที่สอง ว่าจริงๆแล้ว Observable เป็นแค่ตัวกลางหนึ่งที่ทำหน้าที่เชื่อมระหว่างผู้ผลิต (Producer) กับผู้รับ (Consumer)

![ObservableProducer image](https://cdn-images-1.medium.com/max/800/1*hMFkGWfq6oIJe5zN2N8zbQ.jpeg)

นั้นหมายความว่า Observable ไม่ได้เป็นคนสร้างข้อมูลเหล่านั้นขึ้นมาเอง แต่ข้อมูลเหล่านั้นถูกสร้างโดย Producer โดยมี Observable ทำหน้าที่ลำเลียงข้อมูลเหล่านั้นส่งออกไปให้กับผู้รับ โดยให้มองว่า observable ทำหน้าที่เป็นฟังก์ชั่นและการ Subscribe คือการเรียกให้ฟังก์ชั่นนั้นทำงาน และคืนค่าออกมานั่นเอง ลองดูภาพด้านบนประกอบครับ
## Cold Observable
![ColdObservable image](https://cdn-images-1.medium.com/max/800/1*mKI4mNHesE1bfXk6YJ_rEA.png)

Cold Observable หรือที่บางคนเรียกว่า Passive Sequence คือ Observable ที่จะเริ่มปล่อยข้อมูลออกมาก็ต่อเมื่อมี Observer (ซึ่งต่อไปจะขอเรียกว่า Consumer) มา Subscribe ซึ่งข้อมูลที่ปล่อยออกมาให้แต่ละ Consumer นั้นจะเป็นข้อมูลคนละชุดกัน (ถึงแม้หน้าตาของข้อมูลที่ได้จะเป็นแบบเดียวกัน) เสมือนหนึ่งว่าเป็นข้อมูลของใครของมันไม่เกี่ยวข้องกันในแต่ละ Consumer (ดูแผนภาพ Marble ด้านบนประกอบครับ) สาเหตุที่เป็นอย่างนี้เพราะว่า โดยทั่วไปแล้วเมื่อเราต้องการสร้าง Cold Observable ขึ้นมา เราจะสร้างโดยการ create Producer ให้อยู่ภายใน Observable นั้น ทำให้ทุกครั้งที่มีเหตุการณ์ Subscribe เกิดขึ้น Observable จะสร้าง Producer ขึ้นมาใหม่ ส่งผลให้ Consumer แต่ละตัวที่มา subscribe จะมี Data Source หรือ Producer เป็นของตัวเองนั่นเอง ลองดูโค้ดตัวอย่างครับ

```kotlin
val locationObservable = Observable.create<Location> { emitter -> 
    // create data source
    val myLocationService = MyLocationService(this)
    val listener =  { location: Location ->
        emitter.onNext(location)
    }
    myLocationService.addListener(listener)
        
    // stop tracking when un-subscribe
    emitter.setCancellable {
        myLocationService.removeListener(listener)
        myLocationService.stop()
    }
            
    //start tracking 
    myLocationService.start()
}
```
จากโค้ดด้านบน เมื่อ locationObservable ถูก subscribe ด้วย Consumer ใดๆก็ตาม Consumer นั้นจะมี Data source หรือ Producer ซึ่งในที่นี้คือ MyLocationService เป็นของตัวเอง และเมื่อ Observable นั้นหยุดการทำงานลง Producer ก็จะถูกหยุดการทำงานไปด้วย ซึ่งจะเห็นได้ว่าลักษณะการทำงานจะเป็นแบบ 1 Producer ต่อ 1 Consumer

## Hot Observable
![HotObservable image](https://cdn-images-1.medium.com/max/800/1*gSwdDY0PoF0hUR0yGcNpZA.png)

Hot Observable หรือที่บางคนเรียกว่า Active Sequence (ซึ่งน่าจะสื่อความหมายมากกว่า) คือ Observable ที่มีลักษณะพฤติกรรมการปล่อยข้อมูลออกมาแบบไม่สนใจว่าจะมี Consumer มา subscribe มันตอนไหน ซึ่งข้อมูลที่ปล่อยออกมาให้แต่ละ Consumer นั้นจะเป็นข้อมูลชุดเดียวกัน หรือพูดง่ายๆว่าเป็นการแชร์ Data source หรือ Producer ตัวเดียวกันนั่นเอง นั่นหมายความว่า จะไม่มีการสร้าง Producer ขึ้นมาใหม่เมื่อมี Consumer มา subscribe Hot Observable นั้นในภายหลัง ลองดูตัวอย่าง แผนภาพ Marble ด้านบนประกอบครับ
<br/>
การสร้าง Hot Observable โดยทั่วไปจะเป็นในลักษณะที่เราสร้าง Porducer ไว้ด้านนอก Observable แล้วจึงค่อยนำ Producer นั้นไปใช้ภายใน Observable ทำให้ Producer จะไม่ถูกสร้างใหม่ทุกครั้งเมื่อมี Consumer มา subscribe ลองดูโค้ดด้านล่างประกอบครับ
```kotlin
// create location data source
val myLocationService = MyLocationService(this)

val locationObservable = Observable.create<Location> { emitter ->

    val listener =  { location: Location ->
        emitter.onNext(location)
    }
    myLocationService.addListener(listener)

    // stop tracking when un-subscribe
    emitter.setCancellable {
        myLocationService.removeListener(listener)
    }
}

//start tracking
myLocationService.start()
```
ซึ่งจากโค้ดด้านบนจะเห็นว่าเมื่อ locationObservable ถูก subscribe ด้วย Consumer ตัว Producer ที่ locationObservable ใช้จะเป็นตัวเดิมเสมอ ทำให้ข้อมูลที่ถูกส่งไปให้แต่ละ Consumer ที่มา subscribe จะเป็นข้อมูลชุดเดียวกัน
<br/>
สิ่งหนึ่งของ Hot Observable ที่อยากให้สังเกตุคือ ช่วงเวลาในการ subscribe ของแต่ละ Consumer นั้น มีผลต่อข้อมูลที่ Consumer จะได้รับ ซึ่งเมื่อเราย้อนไปดูแผนภาพ Marble ด้านบนจะเห็นว่า Consumer ตัวที่สองจะได้รับข้อมูลแค่ สีน้ำเงิน เพียงสีเดียวเท่านั้น เนื่องจาก subscribe หลังจากที่ Producer ได้ผลิตและ Observable ได้ปล่อยข้อมูลสองตัวแรกออกไปแล้ว
## Why do we need Hot Observable?
สาเหตุที่เราต้องใช้ Hot Observable ในบางเหตุการณ์นั้น เนื่องมาจากในบางครั้งการสร้าง Producer ขึ้นมาเพื่อให้ Observable แต่ละตัวใช้นั้น อาจทำให้กินทรัพยากร (Resource) ของเครื่องโดยไม่จำเป็น ซึ่งจากตัวอย่างโค้ดด้านบน หากเรามี Consumer หลายตัวที่ต้องการรอฟังเหตุการณ์ Location Change เหมือนกัน หากเราใช้ Cold Observable ในการทำงาน MyLocationService ก็จะถูกสร้างทุกครั้งเมื่อเกิดเหตุการณ์ subscribe ขึ้น ซึ่งคงไม่ดีแน่ อีกเหตุผลหนึ่งของการเลือกใช้ Hot Observable คือ Consumer อาจจะไม่ได้อยากได้ข้อมูลเก่าทั้งหมดที่เกิดขึ้นก่อนหน้าที่จะ subscribe แต่ต้องการเฉพาะข้อมูล ณ ปัจจุบันเท่านั้น ดังนั้นการใช้ Hot Observable จึงมีความเหมาะสมมากกว่า ในขณะที่การใช้ Cold Observable จะเหมาะกับกรณีที่เราต้องการให้ Producer ผลิตข้อมูลใหม่ให้เราทุกครั้งที่มีการ subscribe เช่น การดึงข้อมูลจากบาง API เป็นต้น
## Conclusion
เราได้เรียนรู้กันไปแล้วว่า Hot Observable และ Cold Observable นั้นแตกต่างกันอย่างไร และเหมาะกับการทำงานประเภทใด ซึ่งสาเหตุที่เราต้องรู้การทำงานและลักษณะของ Observable ทั้งสองประเภทนี้เนื่องจาก มันจะไปเกี่ยวข้องกับเรื่องของ Subject ที่เราจะพูดถึงในบทความต่อไป อีกทั้งการเรียนรู้พฤติกรรมการทำงานของ Observable เหล่านี้ช่วยให้เรารู้และเลือกไปใช้งานได้อย่างถูกต้องครับ
## Credit
โดยคุณ [nutron](https://medium.com/@nutron) 
