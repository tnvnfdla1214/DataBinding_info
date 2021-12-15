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

*** xml이름이 activity_main인 경우 ActivityMainBinding으로 자동 생성됩니다. 컴포넌트 id값이 tv_sample일 경우 tvSample로 자동 생성됩니다.***

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
