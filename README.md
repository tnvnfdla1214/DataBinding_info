# DataBinding

이전 포스터를 봤으면 MVVM에 DataBinding이 얼마나 필요한 것인지 알 것 입니다.

그래서 처음부터 끝까지 DataBinding에 대해 알아봅시다.

## 데이터 바인딩이란?
<img src="https://user-images.githubusercontent.com/48902047/146203615-e5a10f43-31af-4eb6-af59-778067b901a3.png"></img>

XML(Screen) <- Java/Kotlin(Data/Logic) 인 부분을 Binding.java가 연결합니다.

이부분은 런타임시가 아닌 xml빌드시 생성됩니다.

## 데이터 바인딩 vs 뷰 바인딩

<img src="https://user-images.githubusercontent.com/48902047/142985022-78607cc8-f8ef-4efc-a57a-18c1cf385d81.png"></img>
그림에서 알 수 있듯이, 데이터 바인딩은 뷰 바인딩의 역할도 할 수 있을뿐더러

추가로 동적 UI 콘텐츠 선언, 양방향 데이터 결합도 지원합니다.

<img src="https://user-images.githubusercontent.com/48902047/142985113-01f61b8c-84e9-4c77-9c11-c32dd74a2bd0.png"></img>

땡! 데이터 바인딩이 기능은 다양하지만

뷰 바인딩이 상대적으로 간단하며 퍼포먼스 효율이 좋고 용량이 절약된다는 장점이 있습니다.

실제로 구글 공식문서에서는 단순히 findViewById를 대체하기 위한 거라면, 뷰 바인딩을 사용할 것을 권장하고 있습니다.

그러니 상황에 맞게 바인딩을 골라 사용하면 됩니다.

본격적으로 데이터 바인딩을 구성해봅시다.

## 기본 
### gradle 추가

```kotlin
// 안드로이드 스튜디오 4.0 이상
android {
    ...
    buildFeatures {
        dataBinding = true
    }
}
```

### 레이아웃
데이터 바인딩을 사용하기 위해서 우리가 기존에 알던 레이아웃 구조를 좀 바꾸어야 합니다.

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

### 액티비티
  
액티비티 세팅을 하기전에 Build->Rebuild Project를 해주어야 합니다.(데이터바인딩의 거의 유일한 단점이지만 매우 귀찮습니다.) 그 이유는 위에 설명한것 처럼 컴파일러가 자동으로 생성해주는 바인딩 클래스가 컴파일할때 생성되기 때문입니다.  

자동으로 생성된 Binding 클래스는 다음과 같은 규칙을 따릅니다.
1. Binding 클래스는 레이아웃 파일의 이름을 기준으로 생성되어 파일 이름을 파스칼 표기법으로 변환하고 그 뒤에 “Binding”을 접미사로 붙입니다.
2. 컴포넌트아이디는 “_”를 기준으로 카멜 표기법으로 변환됩니다.  

xml이름이 activity_main인 경우 ActivityMainBinding으로 자동 생성됩니다. 컴포넌트 id값이 tv_sample일 경우 tvSample로 자동 생성됩니다.

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        //setContentView(R.layout.activity_main)

        var binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.tvSample.text = "임민규입니다."//id: tv_sample
    }
}
```

액티비티 세팅은 매우 간단합니다.
DataBindingUtil클래스를 통하여 레이아웃을 Binding 하고, binding 인스턴스를 통해 값을 세팅하면 됩니다.
그러나 결국 DataBinding의 규칙을 통해서 SetText를 하게 됩니다. 이제 어떠한 규칙에 의해서 DataBinding이 되는지 알아봅시다.
<img src="https://user-images.githubusercontent.com/48902047/146318564-9810fb6a-35fb-4b4f-b70d-cd5a223ea31f.png"></img>

## DataBinding, 어떤 기준으로?
데이터 바인딩이 이루어지는 규칙은 3개지로 구성된다고 합니다. 아래의 순위는 실행우선 순위 순입니다.
<img src="https://user-images.githubusercontent.com/48902047/146328769-d0ce11ca-fc9b-4010-b339-2e3f27572a50.png"></img>

1. BindingAdapter (사용자 지정 Setter)
2. BindingMethod (이름이 바뀐 Setter)
3. Set Method (자동 Setter)

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
좀더 들여다 보면

<img src="https://user-images.githubusercontent.com/48902047/146329670-649cb7ad-d1c0-45ab-b6cc-49eab7c7262b.png"></img>

title이란 아이디 값으로 레퍼런스가 생성되는데 거기서 setEnable과 boolean 타입을 보고 setEnabled를 생성하게 됩니다.

이것은 커스텀뷰의 추가된 메소드 등을 아래와 같이 사용 가능하게 됩니다.
<img src="https://user-images.githubusercontent.com/48902047/146330979-08b8aef3-3056-48d3-8e18-e3972be590c5.png"></img>

### BindingAdapter

다음은 Data binding의 꽃이라 불리는 BindingAdapter설명입니다.

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
2. "naver"라는 xml Atrribute를 정의하는데
3. Boolean Type의 값을 받을 수 있다.

실제 적용된 xml에 대해

<img src="https://user-images.githubusercontent.com/48902047/146332587-bf713b15-7e22-44ba-bfb5-190e59c2afa3.png"></img>

실제에도 아래와 같이 위 xml 빌드시 생기는 \*\*\*Binding.java 입니다.
```kotlin
SampleBindingAdapter.naverBindingAdater(this.title, true);
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
예시로는 View에 대해 visiblity 관리를 할떄 생김,사라짐 두가지만 하지 않지 않습니다. 최소 fade In, fade out 과같이 다양한 작동을 넣고 있습니다.

<img src="https://user-images.githubusercontent.com/48902047/146334328-8b6d5c4e-1156-484f-a498-f4881a17ec3d.png"></img>

이점에서 중요한점은 아래와 같이 **xml과 파라미터의 개수와 위치가 같아야합니다.**

<img src="https://user-images.githubusercontent.com/48902047/146334960-f0ff67cd-02b2-4134-ad89-af2a11d2aa61.png"></img>

만약 모든것을 전부 넘기지 않게 아래 예시와 같이 발표자께서는 requireAll=false 을 사용하였었으나, 다수 개의 Attribute에서 어떤 값만 옵셔널하게 처리가 안되어 아래와 같이 오버로딩하면서 쓰고 있습니다. 만약 안넘겨온 것이 있다면 숫자는 0, object는 null이 넘겨 옵니다.
```kotlin
@BindingAdapter(value={"android:visibility", "animType", "animDuration"}) //원래 쓰던것
public static void newAnimationBindingMethod(View view, boolean visibility, @AnimType int animType, int animDuration) { ... }

@BindingAdapter(value={"android:visibility", "animType"}) //발표자님이 바꾸신것
public static void newAnimationBindingMethod(View view, boolean visibility, @AnimType int animType) { ... }
```


