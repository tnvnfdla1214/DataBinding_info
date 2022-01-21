# DataBinding

이번 포스터에서는 Databinding에 대해 재대로 학습하도록 합시다.

아래의 목차는 이래 내용을 다 읽고 다시 보시길 추천합니다.

+ [DataBinding을 왜 사용하는가](https://github.com/tnvnfdla1214/Databinding_1)
+ [DataBinding 실제 사용 프로젝트](https://github.com/tnvnfdla1214/DataBinding_ex)

## DataBinding이란?
<img src="https://user-images.githubusercontent.com/48902047/146203615-e5a10f43-31af-4eb6-af59-778067b901a3.png"></img>

XML(Screen) <- Java/Kotlin(Data/Logic) 인 부분을 ***Binding.java가 연결합니다.

이 부분은 런타임 시가 아닌 xml Build 시 생성됩니다.

## DataBinding vs ViewBinding

<img src="https://user-images.githubusercontent.com/48902047/142985022-78607cc8-f8ef-4efc-a57a-18c1cf385d81.png"></img>

그림에서 알 수 있듯이, DataBinding은 ViewBinding의 역할도 할 뿐만 아니라

추가로 동적 UI 콘텐츠 선언, 양방향 데이터 결합도 지원합니다.

<img src="https://user-images.githubusercontent.com/48902047/142985113-01f61b8c-84e9-4c77-9c11-c32dd74a2bd0.png"></img>

땡! DataBinding이 기능은 다양하지만

ViewBinding이 상대적으로 간단하며 퍼포먼스 효율이 좋고 용량이 절약된다는 장점이 있습니다.

실제로 구글 공식문서에서는 단순히 findViewById를 대체하기 위한 거라면, ViewBinding을 사용할 것을 권장하고 있습니다.

그러니 상황에 맞게 바인딩을 골라 사용하면 됩니다.

본격적으로 데이터 바인딩을 구성해봅시다.

## 기본 
### gradle(module 단위) 추가

```kotlin
// 안드로이드 스튜디오 4.0 이상
android {
    ...
    buildFeatures {
        dataBinding = true
    }
}
```

### findViewByID 제거 (DataBinding, ViewBinding)
  
액티비티 세팅을 하기전에 Build->Rebuild Project를 해주어야 합니다.(DataBinding의 거의 유일한 단점이지만 매우 귀찮습니다.) 

그 이유는 위에 설명한것 처럼 컴파일러가 자동으로 생성해주는 바인딩 클래스가 컴파일할때 생성되기 때문입니다.  

자동으로 생성된 Binding 클래스는 다음과 같은 규칙을 따릅니다.
1. Binding 클래스는 레이아웃 파일의 이름을 기준으로 생성되어 파일 이름을 파스칼 표기법으로 변환하고 그 뒤에 “Binding”을 접미사로 붙입니다.
 (activity_main.xml > ActivityMain.xml > ActivityMainBinding.java)
2. 컴포넌트아이디는 “_”를 기준으로 카멜 표기법으로 변환됩니다.  

xml이름이 activity_main인 경우 ActivityMainBinding으로 자동 생성됩니다. 컴포넌트 id값이 tv_sample일 경우 tvSample로 자동 생성됩니다.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        /*
        * 기존
        setContentView(R.layout.activity_main)
        val tvsample = findViewById(R.id.tv_sample)
        tvSample.text = "임민규"
        */
        
        /*뷰바인딩*/
        var binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.tvSample.text = "임민규"//id: tv_sample
    }
}
```

액티비티 세팅은 매우 간단합니다. 
DataBindingUtil클래스를 통하여 레이아웃을 Binding 하고, binding 인스턴스를 통해 값을 세팅하면 됩니다.

위의 예시는 단순히 findViewByID 정도를 제거하는 방법이었고, 이는 DataBinding이 포함하는 ViewBinding에서 제공하는 기능입니다.

아래에서는 DataBinding으로 점차 view의 참조를 제거하는 방법을 살펴보겠습니다. 

### View의 참조 제거 (DataBinding)
이를 위해서 우리가 기존에 알던 레이아웃 구조를 좀 바꾸어야 합니다.

\<layout> 태그가 가장 바깥쪽에 위치해있으며 그 안에 <data> 태그와 우리가 사용하던 레이아웃이 들어가는 형태입니다.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
   
   <data>
        <variable
            name="user"
            type="com.example.selfstudy_kotlin.User" />
    </data>
    
   <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        <TextView
            android:id="@+id/tv_sample"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}"
            android:layout_gravity="center"/>
    </FrameLayout>
</layout>
```
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        /*
        * 기존
        setContentView(R.layout.activity_main)
        val tvsample = findViewById(R.id.tv_sample)
        tvSample.text = "임민규"
        */
        
        /*
        *뷰바인딩
        var binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.tvSample.text = "임민규" //id: tv_sample
        */
        
        /*데이터 바인딩*/
        var binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        var user = User("임민규", "27", "개발자")
        binding.setUser(user)
        
    }
}
```
뷰바인딩과 데이터 바인딩으로 명시한 예시가 데이터 하나일 경우는 차이가 없어보이지만, 아래 사진과 같이 데이터가 여러개일 경우 view의 참조를 줄여줍니다.

<img src="https://user-images.githubusercontent.com/48902047/146318564-9810fb6a-35fb-4b4f-b70d-cd5a223ea31f.png"></img>

android : text = “@{datamodel명.변수명}”으로 원하는 텍스트를 넣을 수 있습니다. 

이런 모양의 코드는 build 시 “이 코드는 DataBinding의 코드이다. 이런 로직도 필요하니 만들어라” 라는 의미로 build됩니다. 

하지만 이때, setText를 java단에서 사용하지 않는다고 setText를 하지 않는 것이 아닙니다. 

DataBinding의 규칙에 따라 Binding클래스 안에 주입 가능한 메서드가 생성되어 우리가 해야할 것들을 대신 해준 것입니다.

이제 어떠한 규칙에 의해서 DataBinding이 되는지 알아봅시다.


## DataBinding, 어떤 기준으로?
데이터 바인딩이 이루어지는 규칙은 3개지로 구성된다고 합니다. 아래의 순위는 실행우선 순위 순입니다.
<img src="https://user-images.githubusercontent.com/48902047/146328769-d0ce11ca-fc9b-4010-b339-2e3f27572a50.png"></img>

1. BindingAdapter (사용자 지정 Setter)
2. BindingMethod (이름이 바뀐 Setter)
3. Set Method (자동 Setter)
4. 1,2,3이 안되면 빌딩 에러

### Set Method
가장 쉬운것부터 집고 넘어 가겠습니다.
```kotlin
//xml
<TextView
          android:id="@+id/title"
          ...
          android:enabled="@{true}" />
```
위와 같은 xml 빌드시 Binding.java가 아래처럼 생성됩니다.

```kotlin
//ActivityMainBindingImpl.java
this.title.setEnabled(true);
```
좀 더 들여다 보면

<img src="https://user-images.githubusercontent.com/48902047/146329670-649cb7ad-d1c0-45ab-b6cc-49eab7c7262b.png"></img>

[[setEnabled라는 메소드를 쓰는 지, 그 반환값이 boolean값인지]]를 가지고 TextView에 일치사항이 있는지 확인하고 있으면 그것을 사용합니다. 

<img src="https://user-images.githubusercontent.com/54509842/146667807-bd279a16-73e5-4260-b413-b5ce7ba689df.PNG"></img>

다음과 같이 커스텀뷰의 추가된 메소드가 사용 가능한 것으로 확인하고 활용 가능합니다.

### BindingAdapter

다음은 Databinding의 꽃이라 불리는 BindingAdapter설명입니다.

바인딩 어댑터를 사용시 애노테이션 사용하면 public static 필수입니다.
```kotlin
@BindingAdapter(value={"naver"})
public static void naverBindingAdapter(TextView, boolean isNaver) {
    ...
}
```
<img src="https://user-images.githubusercontent.com/48902047/146331954-fcc12d5a-2589-49e9-b8e0-ca450e75e6bc.png"></img>

먼저 정의되어 있는 방법은
1. TextView에 대해
2. "naver"라는 xml Atrribute를 정의하는데 (app:naver라는 속성이 있는지)
3. Boolean Type의 값을 받을 수 있다.

실제 적용된 xml에 대해

<img src="https://user-images.githubusercontent.com/48902047/146332587-bf713b15-7e22-44ba-bfb5-190e59c2afa3.png"></img>

실제에도 아래와 같이 위 xml 빌드시 생기는 \*\*\*Binding.java 입니다.
```kotlin
(naverBindingAdater를 가지고 있는 클래스명).naverBindingAdater(this.title, true);
```
BindingAdapter 애노테이션 선언부 입니다.
```kotlin
@Target(ElementType.METHOD)
public @interface BindingAdapter {
    String[] value(); //value부분
    boolean requireAll() default true; //requireAll부분
}
```
+ value부분 : 이 BindingAdapter가 담당하는 XML Attribute Name List, 다수 개일 경우 메소드 파라미터 순서와 매칭이 되어야 합니다.
+ requireAll : value()의 Attribute가 모두 존재할 때만 처리할지 여부. false일 경우 int/double/float/... 등의 타입의 Attribute가 없을 경우는 값은 0, Object 타입의 Attribute가 없을 경우는 null로 처리됩니다.

예시로는 View에 대해 visibility 관리를 할떄 생김,사라짐 두가지만 하지 않지 않습니다. 최소 fade In, fade out 과같이 다양한 작동을 넣고 있습니다.

<img src="https://user-images.githubusercontent.com/48902047/146334328-8b6d5c4e-1156-484f-a498-f4881a17ec3d.png"></img>

이점에서 중요한 점은 아래와 같이 **xml과 파라미터의 개수와 위치가 같아야합니다.**

<img src="https://user-images.githubusercontent.com/48902047/146334960-f0ff67cd-02b2-4134-ad89-af2a11d2aa61.png"></img>

모든 것을 전부 넘기지 않게 아래 예시와 같이 발표자님은 requireAll=false 을 사용하였으나, 이 경우 안 넘어온 것이 있다면 숫자는 0, object는 null이 넘어옵니다.
```kotlin
@BindingAdapter(value={"android:visibility", "animType", "animDuration"}, requireAll = false) //원래 쓰던 것
public static void newAnimationBindingMethod(View view, boolean visibility, @AnimType int animType, int animDuration) { ... }
```
이러한 경우 사용하지 않는 파라미터에 기본값이 들어와 특정 Attribute만 옵셔널하게 처리 할 수 없기 때문에 아래와 같이 오버로딩해서 씁니다.
```kotlin
@BindingAdapter(value={"android:visibility", "animType", "animDuration"}) //원래 쓰던 것
public static void newAnimationBindingMethod(View view, boolean visibility, @AnimType int animType, int animDuration) { ... }
@BindingAdapter(value={"android:visibility", "animType"}) //발표자님이 바꾸신것
public static void newAnimationBindingMethod(View view, boolean visibility, @AnimType int animType) { ... }
```

#### Observable
<img src="https://user-images.githubusercontent.com/48902047/146635440-3dcfd194-9e82-4c90-bc7f-f8cc5ae34f22.png"></img>

Java/Kotlin 등에서 변경 시 XML에 알려주기 위해 Observable 사용합니다. (DataBinding이 아니라면 LiveData해도 됨.)

<img src="https://user-images.githubusercontent.com/48902047/146635496-40566418-ab37-4194-b063-5fab82f05380.png"></img>

위에는 Observable 인터페이스 입니다. 그러나 만드는 것은 까다롭기에 구글에서 사용하기 편하게 BaseObservable을 제공합니다. BaseObservable은 다음과 같습니다.

<img src="https://user-images.githubusercontent.com/48902047/146635558-4f0e683c-4bb3-415e-823f-f6c4108c7acc.png"></img>

다음은 Observable 사용 예시입니다.


```kotlin
public static class SampleModel { //변경 전
    private String title;
    public SampleMode(String title) {
        this.title = title;
    }
}
public static class SampleModel extends BaseObservable { //변경 후
    private String title;
    public SampleMode(String title) {
        this.title = title;
    }
    public void setTitle(String title) {
        this.title = title;
        notifyPropertyChanged(BR.title);
    }
    @Bindable
    public String getTitlt() {
        return title;
    }
}
```
+ BaseObservable 추가
+ Getter/Setter 추가
+ @Bindable 추가

위에서 코드를 빌드시 BR(Binding Resource)이 생성이 됩니다.

```kotlin
public class BR {
   public static final int _all = 0;
   public static final int titile = 1;
   public static final int model = 2;
}
```
<img src="https://user-images.githubusercontent.com/48902047/146635850-806cc34d-e2ac-4d21-82ca-bbe078462f9e.png"></img>

위 SampleModel 객체 생성 후 setTitle 메소드 사용시 notifyPropertyChanged()를 통해 Binding.java에 변경이 전달 됩니다.

Binding.java에서는 getTitle()을 통해 바뀐 값을 전달 받습니다.

그래도 이렇게 매번 정리하기 불편하므로 구글에서는 아래와 같은 것들을 제공합니다.

+ ObservableBoolean
+ ObservableByte
+ ObservableChar
...
+ ObservableArrayList
+ ObservableArrayMap
+ ObservableParcelabe
+ ObservableField <- 커스텀하는 제공 소스
    + ObservableField<String>
    + ObservableField<CustomMode>
    
SampleModel을 바꿔봅시다.


```kotlin
public static class SampleModel extends BaseObservalbe {
    //private String title; 변경 전
    public final ObservableField<String> title; //변경 후
    public SampleMode(String title) {
        this.title = new ObservableField<>(title);
    }
}
```

이렇게 Observable 처리를 해주면, BindingClass의 setMethod가 "아 observable한 것이 들어와서 변경을 알아야하는구나" 라고 인지하여 notifyChange()메서드를 오버라이드 합니다. 

#### Listener를 Binding 하는 법

<img src="https://user-images.githubusercontent.com/48902047/146636110-914cdd92-f629-4574-b325-9180f06b611c.png"></img>
    
이제 ClickListner들을 어떻게 Binding하는 법을 알아봅시다. 총 세가지가 있습니다. 그 중 3가지를 Object를 직접 넘겨주는 방식과 메서드를 View들과 연결하여 넘겨주는 두가지로 나누는 방법이 있습니다.
##### Object 전달
    
먼저 Object 방식입니다. 

<img src="https://user-images.githubusercontent.com/48902047/146636207-d3efc553-1a4f-4fb3-a132-93c3b3b7e9c2.png"></img>

```kotlin
<Button
        ...
        android:onClick="@{model.clickListener}" />
```
    
위와 같이 사용시 BindingMethod(이름이 바뀐 Setter) 사용이 필요합니다.
    
```kotlin
@BindingMethod(type=View.class, attribute="android:onClick", method="setOnClickListener")
```
    
##### Method 직접 연결
    
```kotlin
<Button
        ...
        android:onClick="@{model:clickListener}" /> //첫번째 방식
<Button
        ...
        android:onClick="@{(view)->model.clickListener(view)}" /> //두번째 방식
```
    
위와 같이 사용시 아래와 같이 사용합니다.
    
```kotlin
public class MainActivity extend AppCompatActivity {

    /*일반식 */
    public void onClickButton(View button) { //둘다 사용가능
        
    }
   
}
```

일반식과 람다식이 있는데, 발표자 분은 람다식을 지향하셨습니다. 왜냐하면 

"어떤 버튼이 어떤 상황에서 온 것이다" 가 식별이 되면, 

1번은 해당 리스너의 메서드 성분들을 전부 충족시켰을 때만 사용이 가능하므로 사용할 수 없습니다. 

하지만 () -> model.onClickButton()과 같이 람다방식을 쓰는 것은 해당 리스너의 성분을 받는 메서드를 그때 그때 만들 필요 없이 하나로 다음과 같이 사용하기 때문입니다.

```kotlin
<Button
        ...
        android:onClick="@{()->model.clickListener()}" /> //두번째 방식
```
```kotlin
public class MainActivity extend AppCompatActivity {

    /*2번 */
    public void onClickButton() { //두번째 방식만 사용가능
    }
}
```

그렇기 때문에 발표자 분은 **람다 방식으로 사용하는게 정신 건강에 좋다**고 설명합니다.

##### CustomView + Custom Listener
<img src="https://user-images.githubusercontent.com/48902047/146636383-87840506-eec1-4219-a787-0062ba94fc1f.png"></img>
    
```kotlin
public class CustomTextView extends AppCompatTextView {
    public interface OnCustomEventListener {
        void onEvent();
    }
}
```
```kotlin
@BindingAdapter("onCustomEvent")
public void setOnCustomEventListener(CustomTextView view, CustomTextView.OnCustomEventListener listener) {
    view.setOnCustomEventListener(listener);
}
//또는 
@BindingMethods({
    @BindingMethod(type=CustomTextView.class, attribute="onCustomEvent", method="setOnCustomEventListener")
})
```
```kotlin
<com.blabla.CustomTextView
                           ...
                           app:onCustomEvent="@{()->activity.onCustomEvent()}"
```

##### 리스너 바인딩 사용시 주의점

<img src="https://user-images.githubusercontent.com/48902047/146636444-5eb0526a-fa9a-4453-a914-c75987db62b4.png"></img>

대부분 리스너들은 return이 void 형식이나 onLongClick 일경우 return이 boolean형식이라 잘못 쓸 경우 이상한 오류가 발생합니다. 아직 데이터 바인딩에 대한 오류 지원은 미흡함으로 주의해야 합니다. 그러니 아래와 같이 의심을 해야 합니다.

<img src="https://user-images.githubusercontent.com/48902047/146636490-bed719ae-282d-4241-8cdb-186d8d943dfb.png"></img>

### Two-way Binding이란
위에까지는 DataBinding의 기본적인 내용이였고 지금부터는 조금 어려울 수 있는 내용입니다.

<img src="https://user-images.githubusercontent.com/48902047/146636593-fdcec3e5-e902-4898-b037-69338ddda04a.png"></img>

먼저 지금까지 예시 부터 보겠습니다.

<img src="https://user-images.githubusercontent.com/48902047/146636697-a236a676-ae28-485c-a4c2-09efbef9080a.png"></img>

지금까지는 Model을 UI에 주입한다고 설명했습니다. 하지만 UI의 data를 받아서 처리해야하는 경우가 있습니다.(ex. editText, CheckBox, Seekbar) 그럴때 우린 기왕 UI의 변화를 Model에 주입하여 받기를 원합니다. 이제 우리는 방향에 따라 이름을 붙입니다.

<img src="https://user-images.githubusercontent.com/48902047/146636722-2953fcad-5252-4884-b0c0-ca60844728db.png"></img>

+ Binding : UI <- Model
+ InverseBinding : UI -> Model
+ Two-way Binding : 둘다

#### Two-way Binding 예시

<img src="https://user-images.githubusercontent.com/48902047/146636790-2bedb0a7-488b-41f5-b1bf-8895b7c9a5c7.png"></img>

위에 예시는 EditText에 값을 입력시 TextView에 값이 자동으로 입력되는 예시입니다. 또한 위의 버튼을 누를 시 EditText와 TextView를 초기화 해주는 예시입니다.
```kotlin
public class TwowayBindingModel {
    public final ObservableField<String> text = new ObservalbeField<>("");
}
```
```kotlin
public class MainActivity {
    public void onClickDone() {
        binding.getModel().text.set("이렇게 초기화가 됩니다.");
    }
}
```
```kotlin
<layout>
    <data>
        <variable
                  name="activity"
                  type="com.test.blabla.MainActivity" />
        <variable
                  name="model" //모델 만든곳
                  type="com.test.blabla.TwowayBindingModel" />
    </data>
    <LinearLayout
                  ... >
        <Button android:onClick="@{()->activity.onClickDone()}" />
        <EditText
                  android:text="@={model.text}" /> //Two-way Binding 
        <TextView
                  android:text="@{model.text}" /> //Binding(정방향)
    </LinearLayout>
</layout>
```

양방향과 단반향의 차이는 아래와 같습니다.

양방향 바인딩 : @={model.title}

단방향 바인딩 : @{model.title}

양방향을 사용했다고 해서 Activity의 코드가 바뀌지 않습니다.

<img src="https://user-images.githubusercontent.com/48902047/146637532-786b2921-7319-4911-8910-5502bc10c462.png"></img>

위에 그림은 단방향 바인딩 순서 입니다. 아래 그림은 양방향 그림입니다.

<img src="https://user-images.githubusercontent.com/48902047/146637595-a03be281-b088-46c7-931d-1a6abefafb37.png"></img>

EditText는 양방향 바인딩이라 텍스트 수정시 InverseBinding이 이루어지며, 이 후 변경된 값으로 Binding이 이루어집니다. 알아야 할 점은 실제 코드에서 EditText쪽으로 전달해 주지 않습니다.
아래는 Binding과 InverseBinding의 정의된 코드입니다.
+ Binding
```kotlin
@BindingAdapter("android:text")
public static void setText(TextView textView, String text) {
    tetView.setText(text)
}
```
+ InverseBinding
```kotlin
@InverseBindingAdapter(attribute={"android:text"})
public static String getText(TextView textView) {
    return tetView.getText().toString();
}
```

<img src="https://user-images.githubusercontent.com/48902047/146637752-be1d6e31-3faa-44f3-a450-c2cb28cbdbdc.png"></img>

위의 코드를 볼 시 위 그림과 같이 크게 두 가지의 차이점이 있습니다.
이유는 binding 같은 경우 "값을 받아서 UI 에 값을 넣는다","UI 작업을 업데이트 시킨고 UI작업을 한다" 라고 생각하고 있는데
InverseBinding 같은 경우 "UI의 변화, UI 이벤트를 감지해서 값을 돌려준다"라고 반대 방향입니다. 그렇기에 파라미터 값을 받을 필요없고 반환값이 필요합니다.

아래를 보면 

+ Binding 선언부
```kotlin
@Target(ElementType.METHOD)
public @interface BindingAdapter {
    String[] value(); //이것이
    boolean requireAll() default true;
}
```
+ InverseBinding 선언부
```kotlin
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
public @interface InverseBindingAdapter {
    String attribute(); //이렇게 바뀜
    String event() default ""; //중요
}
```
이처럼 Binding은 복합적인 이벤트를 가지고 와서 view에 적용할 수 있어서 String[]인 것에 반해, InverseBinding는 하나의 attributes는 하나의 이벤트로 처리해야 하기 때문에 String 입니다.

다음은 중요라 표시한 event의 대한 설명이다.

먼저 InverseBinding은 "UI의 변화를 Model의 주입"이 목표이다. 그렇다면 event는 언제,혹은 어떤 것을 받느냐에 관련된 내용이다. 아래 사진은 마음대로 event를 바꿀 수 있다.

<img src="https://user-images.githubusercontent.com/48902047/146638201-301143f5-1063-439f-805b-e08449f535af.png"></img>

event는 따로 BindingAdapter를 만들어 정의해 주어애 합니다.

<img src="https://user-images.githubusercontent.com/48902047/146638349-850fd678-4678-4269-bce0-835b66c22abd.png"></img>
<img src="https://user-images.githubusercontent.com/48902047/146638408-a42d3b57-3b6b-428f-a1c0-94df946a8a3e.png"></img>

event사용시 특이하게 InverseBindingListener라는 파라미터를 받아야합니다. 그리고 onChange를 발생시키는데 그것은 InverseBinding이 동작하는 시점입니다.

+ InverseBinding에서 event 사용시
```kotlin
@InverseBindingAdapter(attribute={"android:text", event="textEvent"})
public static String getText(TextView textView) {
    return textView.getText().toString();
}
@BindingAdapter("textEvent")
public static void setTextEvent(TextView textView, final InverseBindingListener listener) {
    textView.addTextChangedListener(new TextWatcher() {
        @Overrride
        public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
            listener.onChange();
        }
    });
}
```
위 구현의 동작 흐름입니다.

<img src="https://user-images.githubusercontent.com/48902047/146638552-a84ffdd6-01b6-4339-b197-35422b7e815b.png"></img>

1. EditText에 텍스트 변경
2. @BindingAdapter("textEvent")에서 텍스트 변경으로 InverseBindingListener의 onChange() 호출
3. @InverseBindingAdapter(attribute={"android:text", event="textEvent"}) 에서 변경된 텍스트 반환
4. TwowayBindingModel의 ObservableField인 text에서 값이 변경됨을 알림.
5. @BindingAdapter("android:text") 으로 EditText와 TextView에 변경된 값을 설정함.

하지만 **위와 같이 처리시 5번에서 양방향으로 들어오면 BindingAdapter에서 이를 변경이라 생각하여 다시 1번이 발생하게 되어 계속 반복하게 무한 루프하게 됩니다.**
구글에서도 이에 대한 직접 로직을 추가하여 막아야 한다고 그런 매직은 없다고 합니다. (TextViewBindingAdpater 참조)

TextViewBindingAdpater 에서는 이전과 변경되는 텍스트 값을 비교하여 예외처리를 추가합니다.
<img src="https://user-images.githubusercontent.com/48902047/146638687-677f23e4-2a5e-43d5-ba8e-d9396b9c4d1e.png"></img>


### \<include>와 \<ViewStub>의 DataBinding
마지막 챕터입니다. 아래 예시는 네이버 프리즘라이브 사진인데 프리즘 앱이 라이브를 하는 앱이기에  Activity가 항상 돌아가는 pause 되면 안되는 상태여야 합니다. 거기에 액티비티는 화면이 갱신되면서 카메라 뷰를 비추면서 다양한 기능을 보여주어야 합니다.

**하나의 Activity**
+ 수많은 View들
+ 개발자들 사이의 분할(분담)
+ 기능들의 확장성
+ 변경의 용이성

**"결국은 어떻게 나눠서 개발할 것인가"** 가 목표였습니다.

이번 챕터는 architecture나 패턴에 대한 부분은 제외하고 **DataBinding 관점** 과 **View 관점**에서만 논합니다.

<img src="https://user-images.githubusercontent.com/48902047/146638787-223dffb0-e68c-4df2-bc3a-6abaf7352b51.png"></img>

결국은 이것을 해결하기 위해 이번 챕터의 제목처럼  \<include>와 \<ViewStub> 를 잘 사용하면 됩니다.

#### include
재사용 하고 싶은 컴포넌트를 레이아웃에 넣을 때 include 태그를 사용합니다. (예 상단바와 같이 재사용하는 컴포넌트)
```kotlin
   <include
        layout="@layout/include_top_bar"/>
```

#### ViewStub
VideStub 는 사이즈가 0인 더미 뷰입니다. 기능은 include 와 동일하지만, setVisibility(int) 또는 inflate 되었을 때 view 를 그리기 시작합니다. lazy include 라고 생각하시면 됩니다.
```kotlin
   <ViewStub
    android:id="@+id/stub"
    android:layout="@layout/include_top_bar"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/actionBarSize"/>
```
```kotlin
ViewStub stub = findViewById(R.id.stub);
View inflated = stub.inflate();
TextView tvIncludeTopBar = inflated.findViewById(R.id.tvIncludeTopBar);
```

아래의 화면은 프리즌 화면인데 \<include>와 \<ViewStub>으로 쪼개면 아래와 같이 됩니다.

<img src="https://user-images.githubusercontent.com/48902047/146639036-73d7543c-76e1-42f3-8beb-e040c1e50bcd.png"></img>

이 그림 대신 아래와 같이 트리 구조로 볼 수 있습니다.

<img src="https://user-images.githubusercontent.com/48902047/146639068-eeb9afcf-3234-4c65-bd77-92ab2deab254.png"></img>

이들은 Binding Class로 구성하면 각각의 Binding Class가 생길 것 입니다. 그런데 Binding을 하기 위해 model을 넘기는데 MainActivity만 model을 받아 MainBinding은 받을 수 있는데 나머지는 어떻게 넘기느냐 라는 고민이 있었습니다.

<img src="https://user-images.githubusercontent.com/48902047/146639130-c6fba6f5-9d12-4d3b-b317-ff439a0e150e.png"></img>

해결법은 MainBinding에서 IncludeBindingModel이라는 child를 만들어 놓습니다. 그 다음 xml에서 include는 main.xml에 있는 것을 가져 옵니다.

main.xml -> include1.xml

-> viewstub1.xml
-> viewstub2.xml

<img src="https://user-images.githubusercontent.com/48902047/146639357-6ed31b50-364e-4cdb-ba5b-a672e40f889d.png"></img>
<img src="https://user-images.githubusercontent.com/48902047/146639369-0cee6d88-d5c8-4855-9b79-b1bd9dee728a.png"></img>
<img src="https://user-images.githubusercontent.com/48902047/146639382-de662629-7b92-4f1d-85d5-9eebd6a77374.png"></img>

메인에서 하위로 데이터를 전달해야 하므로 데이터도 동일하게 depth를 만들어서 사용합니다.

MainBindingModel -> IncludeBindingModel

+ xml에서 하위로 모델 넘길 경우
```kotlin
@InverseBindingAdapter(attribute={"android:text", event="textEvent"})
public static String getText(TextView textView) {
    return tetView.getText().toString();
}
@BindingAdapter("textEvent")
public static void setTextEvent(TextView textView, final InverseBindingListener listener) {
    textView.addTextChangedListener(new TextWatcher() {
        @Overrride
        public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
            listener.onChange();
        }
    });
}
```
```kotlin
<!-- main.xml -->
<layout>
    <data>
        <variable
                  name="model2"
                  type="IncludeBindingModel" />
    </data>
    <TextView
             app:mode2="@{model2.title}" />
</layout>
```

위와 같이 되는 이유는 \<data> 부분을 빌드할 시 SetModel2이 생성된다고 앞서 말했습니다. Main쪽에 있는 model2를 자동 setter해서 SetModel2와 만들어져 연결되지 않나 추측합니다. 그렇기에 이름만 맞춰주면 자동 연결된다고 합니다.

<img src="https://user-images.githubusercontent.com/48902047/146639499-9941df48-8de7-4671-919f-a724336a5093.png"></img>

그래서 결국 Main에 model을 밀어 넣을 시 하위 binding에 념겨 줄 수 있습니다. 이렇게 model을 분리 가능하고 xml을 분리할 수 있으니 자연스럽게 각각에 맞는 로직을 viewmodel에 맞춰 넣을 수 있습니다.

<img src="https://user-images.githubusercontent.com/48902047/146639668-f826883a-4202-42af-821b-e86a599128ff.png"></img>
<img src="https://user-images.githubusercontent.com/48902047/146639674-331befb0-91c1-4712-8b6a-16f79b33db59.png"></img>

프리즘은 각 View별로 ViewModel 1:1 매핑하여 개발했다고 합니다.

ViewStub 바인딩 시 include보다 까다롭습니다.

ViewStub도 View 상속하다보니 일반적인 BindingAdapter를 사용 가능하나, ViewStub 특성을 살리는 부분은 ViewStub를 파라미터로 받는 BindingAdpater로 구현해야 합니다.

하지만 이는 Gradle 3.1.0이상부터 되는 것으로 보여 그 이하의 환경에서는 편법을 써야 합니다.

ViewStub를 DataBinding시에 Binding.java에는 ViewStubProxy가 사용됩니다.

그 ViewStubProxy 안에 ViewStub이 있습니다.

ViewStub을 사용시에는 반드시 android:id 가 있어야 합니다. (없을 경우 에러로그가 딱히 안나옴)

최초에는 inflate 필요합니다. ViewStub의 경우 ViewStubProxy에서 inflat시에 다시 바인딩(rebinding)을 하게 합니다.

위의 이유로 애니메이션 동작 등이 오동작할 수 있습니다.

다시 바인딩되는 범위는 ViewStub을 싸고 있는 부모뷰까지이므로 편법으로 ViewStub을 include로 싸면 되긴 합니다. (대신 아름답지는 않음)

## Q&A

Q. 협업을 위해 바인딩 어답터 오버로딩한다고 하셨는데, 바인딩어답터에서 바인딩어답터를 호출할 수 있습니까?

넵! 이미 BindingAdapter로 넘어오면 그 시점부터는 Java Method라고 보셔도 무방합니다. 즉… 필요한 BindingAdapter의 Method를 호출해주면 됩니다..!

Q. 바인딩 문법 틀렸을때 자바처럼 잘 디버깅해주지않던데 쉽게 문법오류 잡는 팁 있을까요?

안타깝게도… 이건 좋은 방법이 있진 않아요. 좋은 방법이라면… 수없이 많이 맞아보는 것…?ㅎㅎ…

Q. 동적으로 ui위치를 변경하고자할때 데이터바인딩을 사용하고 싶은데 뷰의 속성변경 말고 뷰 객체 자체를 받을 수 있나요?

결론적으로… 저라면 동적으로 UI위치를 변경하고자 한다면 예를들면 이렇게 작성할 것 같아요..

@BindingAdapter](https://github.com/BindingAdapter)( {"positionX", "positionY"} ) public static void moveView(View view, float positionX, positionY) { view.setTranslationX(positionX); view.setTranslationY(positionY); }

Q. recyclerView의 스크롤 이벤트 같은것도 데이터 바인딩으로 받을 수 있을까요? (스크롤 위치에 따라서 이벤트 처리를 하고 싶은 경우에)

해보지는 않았는데요. Listener를 Setting해서 값을 가져오는 방식이라면 가능합니다. InverseBindingAdapter를 이용하면 될것 같은데요.

참고로 저의 경우에는 ViewPager의 Page 이동과 offset 이동도 InverseBinding으로 처리하고 있습니다.

Q. 데이터바인딩을 사용하게 되면서 코드 가독성이 떨어진 점은 없었나요? Textview.setText면 충분하지 않았나 싶은 생각도 들고 그러는데 데이터바인딩의 장점이 궁금합니다.

이건 취향차이기인 해요. 저는 오히려 코드 가독성이 올라가는 부분도 있다라고 생각합니다. View에 대한 로직은 모두 xml에서 볼 수 있다라고 생각할 수 있으니까요. 로직이 복잡해지면 xml도, 코드에서도 복잡하긴 한건 매한가지 이긴 하니까…?

사실. 이 DataBinding은 코드의 가독성을 높인다. 개발 생산성을 높인다.의 이야기와는 맞진 않다고 생각합니다. 정확하게는 Java/코틀린 코드에서 View에 대한 참조를 제거하는 도구이다. 라고 보시는 쪽이 맞긴 해요. 또한… 이런저런 장점들이 있기는 한데… 가장 크게 예를 들면 Image Loading 같은 경우에요. 그냥 코드로 작성하게 되는경우 Image를 Load하는 모든 지점에서 자바/코틀린으로 Loading 로직을 작성해주셔야 하는데 Binding을 이용한다면 이런식으로 작성하면 끝! 할수도 있거든요
