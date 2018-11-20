## Subject
Subject เป็นเหมือนสะพานที่เชื่อมระหว่าง Observable แล้ว Observer (ต่อไปจะขอเรียกว่า subscriber) คือ Subject จะทำหน้าที่เป็นทั้ง Observable และ Subscriber ในตัวเดียวกัน นั้นหมายความว่าเราสามารถใช้ Subject แทน Subscriber ในการ subscribe Observable ได้ และยังสามารถใช้มันในการส่งต่อ Item ที่ได้รับมาไปยัง Subscriber ตัวอื่นที่ subscribe มันอยู่ หรือแม้กระทั้งสร้าง Item ใหม่ขึ้นมาได้ เพราะเนื่องจากมันเป็น Observable ในตัวมันเองอยู่แล้ว

Subject เป็น Hot observable (ที่กล่าวถึงในบทความที่ 4) นั้นหมายความว่า มันยังคงทำงานต่อไปได้ถึงแม้ไม่มีใคร subscribe มันอยู่ ให้เราลองนึกถึงสถานีวิทยุที่ยังคงเปิดเพลงต่อไป ถึงแม้ว่าเราจะปิดวิทยุของเราไปแล้วก็ตาม และเนื่องจากมันเป็น Hot observable ดังนั้นมันจะไม่ execute operation ใหม่ทุกครั้งที่เกิดการ subscribe นั้นหมายความว่า Subscriber ทุกตัวที่ subscribe มันอยู่จะได้รับข้อมูลชุดเดียวกัน โดยไม่มีการรันโค้ดภายใน Subject ใหม่ ซึ่งผิดกับ Cold Observable ตรงที่จะรันโค้ดภายใน Observable ใหม่ทุกครั้งที่เกิดการ subscribe ขึ้น

## Varieties of Subject
Subject ที่มีให้เราใช้ใน RxJava2 มีอยู่ 4 ตัวด้วยกัน โดยแต่ละตัวก็จะมีพฤติกรรมการทำงานที่แตกต่างกัน โดยเราจะมาดูกันว่าแต่ละตัวทำงานแตกต่างกันอย่างไร
### PublishSubject
![PublishSubject image](https://cdn-images-1.medium.com/max/800/0*qG2HcUBXRvYJuoSA.)

PublishSubject คือ Subject ที่ปล่อย Item ออกมาตามช่วงเวลาที่ Source Observable (Observable ที่ผลิตข้อมูล) ปล่อยข้อมูลนั้นๆออกมา โดยที่ Subscriber จะได้รับข้อมูลตามช่วงเวลาที่เข้ามา subscribe นั่นหมายความว่า หาก Subscriber เข้าไป subscribe ตั้งแต่ Subject เริ่มปล่อย Item ออกมา Subscriber นั้นก็จะได้รับ item ที่ปล่อยออกมาตั้งแต่เริ่ม แต่หากไป subscribe ตอนช่วงกลางๆ หลังจากที่ Subject มีการปล่อย Item ออกไปบ้างแล้ว Subscriber ตัวนั้นก็จะได้รับแค่เฉพาะ Item ที่เกิดขึ้นหลังจากการ subscribe เท่านั้น ลองดูแผนภาพ Marble ด้านบนประกอบครับ

หากเปรียบเป็นนักเรียนกับห้องเรียน PublishSubject ก็จะเหมือนกับตอนที่นักเรียนเข้าเรียนตอนช่วงเวลาใดๆ ของ class นักเรียนคนนั้นก็จะได้เรียนแต่สิ่งที่อาจารย์สอนนับตั้งแต่ช่วงเวลาที่เข้าเรียน โดยจะไม่รู้เลยว่าก่อนหน้านั้นอาจารย์สอนอะไรไปบ้าง
```kotlin
val source: PublishSubject<Int> = PublishSubject.create()
// subscribe with FirstObserver
source.subscribe { number -> Log.d("PublishSubject", "FirstObserver onNext: $number") }
source.onNext(1)
source.onNext(2)
source.onNext(3)
// subscribe with SecondObserver
source.subscribe{ number -> Log.d("PublishSubject", "SecondObserver onNext: $number") }
source.onNext(4)
source.onComplete()
```
```text
// OUTPUT:
FirstObserver onNext: 1
FirstObserver onNext: 2
FirstObserver onNext: 3
FirstObserver onNext: 4
SecondObserver onNext: 4
```
### BehaviorSubject
![BehaviorSubject image](https://cdn-images-1.medium.com/max/800/0*0eY370DoXN0FrEQ3.)

BehaviorSubject คือ Subject ที่เริ่มต้นด้วยการปล่อย Item ตัวล่าสุด (most recent item) ที่ถูกปล่อยออกมาจาก Source Observable จากนั้นจึงค่อยปล่อย Item ที่เกิดขึ้นหลังจากการ subscribe ไปแล้วให้กับ Subscriber ตามปกติเหมือน PublishSubject ดังนั้น BehaviorSubject จึงมีความแตกต่างกับ PublishSubject ตรงที่ BehaviorSubject จะมีค่า Default หรือ ค่าล่าสุด ปล่อยออกไปให้กับ Subscriber ก่อน แต่หาก Source Observable ไม่ได้ปล่อย Item ใดๆ ออกมาก่อนหน้าการ subscribe หรือ เราไม่ได้กำหนดค่า Default ใดๆให้กับ BehaviorSubject ตัว Subscriber ก็จะไม่ได้รับ Item ใดๆ ออกมาเช่นกัน

หากเปรียบเป็นนักเรียนกับห้องเรียน BehaviorSubject ก็จะเหมือนกับตอนที่นักเรียนเข้าห้องเรียนสาย แต่โชคดีที่มีเพื่อนคอยบอกให้ว่า อาจารย์พูดเรื่องอะไรไปก่อนหน้านี้ ทำให้นักเรียนคนนั้นพอจะประติดประต่อเรื่องราวได้บ้าง
```kotlin
val source: BehaviorSubject<Int> = BehaviorSubject.create()
// subscribe with FirstObserver
source.subscribe { number -> Log.d("BehaviorSubject", "FirstObserver onNext: $number") }
source.onNext(1)
source.onNext(2)
source.onNext(3)
// subscribe with SecondObserver
source.subscribe{ number -> Log.d("BehaviorSubject", "SecondObserver onNext: $number") }
source.onNext(4)
source.onComplete()
```
```text
// OUTPUT:
FirstObserver onNext: 1
FirstObserver onNext: 2
FirstObserver onNext: 3
SecondObserver onNext: 3
FirstObserver onNext: 4
SecondObserver onNext: 4
```
### ReplaySubject
![ReplaySubject image](https://cdn-images-1.medium.com/max/800/0*mNud58pPJQEoN3NE.)

ReplaySubject คือ Subject ที่จะปล่อย Item ทั้งหมดตั้งแต่ Item แรกให้กับ Subscriber ที่เข้ามา subscribe โดยที่ไม่สนใจว่า Subscriber นั้นจะเข้ามา subscribe ตอนไหน จากนั้นจึงปล่อย Item ที่เกิดขึ้น (จาก Source Observable) หลังจากการ subscribe ตามปกติเหมือน PublishSubject

หากเปรียบเป็นนักเรียนกับห้องเรียน ReplaySubject ก็จะเหมือนกับการที่นักเรียนเรียนออนไลน์ผ่านทาง Youtube channel ซึ่งหากนักเรียนเปิดเข้ามาเรียนไม่ทันตอนเริ่ม class นักเรียนก็สามารถย้อนกลับไปดู replay ที่อาจารย์ได้สอนไปในช่วงต้นชั่วโมงได้นั้นเอง
```kotlin
val source: ReplaySubject<Int> = ReplaySubject.create()
// subscribe with FirstObserver
source.subscribe { number -> Log.d("ReplaySubject", "FirstObserver onNext: $number") }
source.onNext(1)
source.onNext(2)
source.onNext(3)
// subscribe with SecondObserver
source.subscribe{ number -> Log.d("ReplaySubject", "SecondObserver onNext: $number") }
source.onNext(4)
source.onComplete()
```
```text
// OUTPUT:
FirstObserver onNext: 1
FirstObserver onNext: 2
FirstObserver onNext: 3
SecondObserver onNext: 1
SecondObserver onNext: 2
SecondObserver onNext: 3
FirstObserver onNext: 4
SecondObserver onNext: 4
```
### AsyncSubject
![AsyncSubject image](https://cdn-images-1.medium.com/max/800/0*vJzZvi0rmQjKnqlM.)

AsyncSubject คือ Subject ที่จะปล่อย Item ออกมาแค่ค่าสุดท้ายค่าเดียวเท่านั้น และจะปล่อยออกมาเมื่อ Source Observable ทำงานเสร็จแล้วเท่านั้น ดังนั้น หาก Source Observable ไม่ปล่อยค่าใดๆ ออกมา หรือ เกิด Error ขึ้นก่อน จะทำให้ AsyncSubject หยุดการทำงานโดยที่ไม่ปล่อย Item ใดๆ ออกมาได้เช่นกัน (แต่จะเรียก onError() แทนในกรณีที่เกิด Error ขึ้น)

หากเปรียบเป็นนักเรียนกับห้องเรียน ก็คงเปรียบได้กับกรณีที่นักเรียนเข้าห้องเรียนไม่ว่าช่วงเวลาใดก็ตาม แต่รอฟังแค่บทสรุป (หรือสิ่งที่อาจารย์สอนเป็นอย่างสุดท้าย) ของ class เรียนนั้นเท่านั้น
```kotlin
val source: AsyncSubject<Int> = AsyncSubject.create()
// subscribe with FirstObserver
source.subscribe { number -> Log.d("AsyncSubject", "FirstObserver onNext: $number") }
source.onNext(1)
source.onNext(2)
source.onNext(3)
// subscribe with SecondObserver
source.subscribe{ number -> Log.d("AsyncSubject", "SecondObserver onNext: $number") }
source.onNext(4)
source.onComplete()
```
```text
FirstObserver onNext: 4
SecondObserver onNext: 4
```
## แล้วเราจะใช้มันอย่างไร
สำหรับคำถามที่ว่าจะใช้มันอย่างไร หรือใช้มันตอนไหนนั้น อันนี้ก็ขึ้นอยู่กับจุดประสงค์ของการใช้งานแล้วล่ะครับ ในบทความนี้ขอยกตัวอย่างเหตุการณ์ที่ใช้ BehaviorSubject และ PublishSubject สองตัวนี้มาให้ดูกันนะครับ เนื่องจากเป็นตัวที่เราใช้ค่อนข้างบ่อย ซึ่งอีกสองตัวยังไม่เคยเจอเหตุการณ์ที่ต้องใช้มันจริงๆจังๆเหมือนกัน หากใครมีไอเดียก็มาแลกเปลี่ยนกันได้ครับ แต่ผมเชื่อว่าหากใช้สองตัวนี้เป็น อีกสองตัวที่เหลือก็ไม่น่าเป็นห่วงเท่าไหร่ครับ

ตัวอย่างของการเลือกใช้ BehaviorSubject ได้แก่ กรณี Location update แน่นอนว่าเราคงไม่อยากได้ข้อมูล Location ทั้งหมดตั้งแต่เริ่มทำงาน แต่เราอยากได้แค่ Location ล่าสุดเท่านั้น และควรจะได้ Location ทันทีที่ subscribe ด้วย ดังนั้น BehaviorSubject จึงดูเหมาะสมที่สุดครับ ลองดูตัวอย่างการใช้งานครับ
```kotlin
class MyLocationManager(private val context: Context) {

    private var mLocationManager: LocationManager? = null
    private val listener = MyLocationListener()
    val locationUpdateSubject: BehaviorSubject<Location> = BehaviorSubject.create()
    fun startUpdateLocation() {
        mLocationManager = context.getSystemService(Context.LOCATION_SERVICE) as LocationManager?
        mLocationManager?.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0f, listener)
        // Force an update with the last location, if available.
        val lastLocation = mLocationManager?.getLastKnownLocation(LocationManager.GPS_PROVIDER)
        if (lastLocation != null) {
            locationUpdateSubject.onNext(lastLocation)
        }
    }
    ...
    private inner class MyLocationListener : LocationListener {
        override fun onLocationChanged(location: Location) {
            locationUpdateSubject.onNext(location)
        }
        ...
    }
}
class LocationFragment: Fragment() {
    private lateinit var locationManager: MyLocationManager
    private val disposeBag = CompositeDisposable()
    locationManager = MyLocationManager(context)
    ...
    private fun initModule() {
        locationManager.startUpdateLocation()
        locationManager.locationUpdateSubject.subscribe { location ->
            // Do some thing with location on Subscriber1
        }.addTo(disposeBag)
        locationManager.locationUpdateSubject.subscribe { location ->
            // Do some thing with location on Subscriber2
        }.addTo(disposeBag)
    
        // add more subscriber that you want 
    }
    ...
}
```
จากโค้ดจะเห็นว่าเราสร้าง locationUpdateSubject ขึ้นมาเป็น BehaviorSubject โดยทุกครั้งที่ได้รับ Location update ก็จะทำการเรียก onNext(location) เพื่อส่ง update location ไปยัง Subscriber ซึ่งทางฝั่ง Subscriber ที่อยู่ใน Fragment ก็ทำหน้าที่ subscribe เจ้า locationUpdateSubject และเมื่อได้ผลลัพธ์ ก็จะนำ Location ที่ได้ไปใช้งานต่อไป โดยเราจะเห็นว่า เราสามาถ subscribe ได้มากกว่าหนึ่งตัว โดยที่ทุกตัวจะได้ข้อมูลเดียวกัน

ส่วน PublishSubject ที่ผมใช้บ่อยนั้น จะใช้ในกรณีที่เราไม่อาจ subscribe ระหว่าง Source Observable และ Subscriber กันได้โดยตรง ตัวอย่างเช่น หากใครใช้ MVVM อยู่ ก็คงไม่อยากให้ ViewModel รู้จัก View มากสักเท่าไหร่หากไม่จำเป็น ซึ่งถ้าเราต้องการดักฟังเหตุการณ์การกดปุ่มของผู้ใช้ที่เกิดขึ้นในฝั่ง View แล้วส่งต่อเหตุการณ์นั้นให้กับ Subscriber ที่อยู่ในฝั่ง ViewModel แล้วละก้อ หากเราให้ Subscriber เหล่านั้นมา subscribe View โดยตรงก็จะเกิด Dependency ขึ้น ดังนั้นเราจึงนำ PublishSubject มาช่วยในกรณีนี้ได้ ซึ่งสาเหตุที่เลือกใช้ PublishSubject กับกรณีการกดปุ่มนั้นเพราะว่า Subscriber ไม่จำเป็นต้องสนใจว่าก่อนหน้าที่จะ subscribe นั้น ผู้ใช้กดปุ่มไปแล้วกี่ครั้ง แต่จะยึดตามเหตุการณ์ที่เกิดขึ้นหลังจาก subscribe แล้วเป็นสำคัญ และตอบสนองต่อเหตุการณ์ที่เกิดขึ้นเหล่านั้น ดังนั้นการเลือกใช้ PublishSubject จึงเหมาะกับเหตุการณ์นี้ ลองดูตัวอย่างครับ
```kotlin
interface SampleViewModel {

    val input: Input
    val output: Output

    interface Input {
        val refresh: Observer<Unit>
    }

    interface Output {
        val number: Observable<ValueEntity>
    }
}

class SampleViewModelImpl(val getNumberInteractor: GetNumberInteractor) :
        SampleViewModel, SampleViewModel.Input, SampleViewModel.Output {
    
    override val input: SampleViewModel.Input get() = this
    override val output: SampleViewModel.Output get() = this
    override val refresh: PublishSubject<Unit> = PublishSubject.create()
    override val number: Observable<ValueEntity>

    init {
        // View will subscribe 'number' to get the output
        number  = refresh.observeOn(Schedulers.io())
                .flatMap { getNumberInteractor.buildObservable()}
    }
}

class SampleFragment(): Fragment() {

    ...

    private fun initInput() {
        RxView.clicks(randomRefreshBtn)
                .map { Unit }
                .doOnSubscribe { disposeBag.add(it) }
                .subscribeWith(viewModel.input.refresh)
    }

    private fun initOutput() {
        viewModel.output.number.observeOn(AndroidSchedulers.mainThread())
                .subscribe {
                    randomNumberTextView.text = it.value.toString()
                }
                .addTo(disposeBag)
    }
}
```
จากโค้ดจะเห็นว่าเราสร้าง Interface ของ ViewModel ไว้ โดยมีทั้ง Input และ Output อยู่ภายใน ให้เราสังเกตุที่ Input ว่าเราประกาศ refresh เป็น Observer แต่เมื่อ Implement จริงใน SampleViewModelImpl เราประกาศให้เป็น PublishSubject แทน สาเหตุที่ทำแบบนี้เนื่องจาก PublishSubject นั้นเป็นได้ทั้ง Observable และ Observer ดังนั้นเราจึงสามารถประกาศ Interface เป็น Observer ได้ อีกเหตุผลหนึ่งก็คือ เราไม่อยากให้ฝั่ง View สามารถเรียก onNext() ได้เองโดยตรง ดังนั้นการประกาศเป็น Observer จะช่วยป้องกันไม่ให้ฝั่ง View เรียก onNext() โดยพลการได้นั้นเอง ต่อไปให้เราสังเกตที่ฟังก์ชั่น initInput ใน SampleFragment ซึ่งจะเห็นว่าตอนนี้ refresh ทำหน้าที่เป็น Observer ที่รอรับข้อมูลจาก RxView.clicks() และเมื่อเราไปดูฟังก์ชั่น init ใน SampleViewModelImpl เราก็จะเห็นว่า refresh จะทำหน้าที่เป็น Observable ที่นำข้อมูลที่ได้รับมา ไปทำอะไรบางอย่างเพื่อให้ได้ผลลัพธ์ออกมานั้นเอง
## Credit
โดยคุณ [nutron](https://medium.com/@nutron)