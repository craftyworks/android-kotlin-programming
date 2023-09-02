# View Binding

## About

ViewB inding 은 Android Studio 3.6 에서 새롭게 추가된 기능(Feature)

모듈의 View Binding 설정이 enabled 되면 모듈내의 XML layout 파일에 대한 바인딩 클래스를 자동 생성한다.

바인딩 클래스는 ID 를 가진 모든 View 에 대한 참조를 포함한다.

## Setup

```
android {
    buildFeatures {
	    viewBinding=true
    }
}
```

## 바인딩 클래스

View Binding 이 활성화되면 각 XML layout file 별로 바인딩 클래스가 생성된다.

바인딩 클래스는 레이아웃의 Root View 와 ID attribute 를 가진 모든 View 에 대한 참조를 갖는다.

바인딩 클래스명은 XML 파일명을 Pascal Case 로 변환하여 Binding 단어를 맨끝에 추가하여 명명된다.


예를 들어 레이아웃 XML 파일 명이 result_profile.xml 이라면 자동 생성되는 바인딩 클래스명은 ResultProfileBinding 이다.

## Activity 에서 사용하기

```
private lateinit var binding: ResultProfileBinding 

override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	binding = ResultProfileBinding.inflate(layouteInflater)
	setContentView(binding.root)
}
```

## Fragment 에서 사용하기 

```
private var _binding: ResultProfileBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = ResultProfileBinding.inflate(inflater, container, false)
    return binding.root
}

override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}
```

Fragment 는 View 보다 오래 살아남는다. onDestroyView 함수를 override 하여 바인딩 클래스에 대한 참조를 지워주자.
번거롭다.

## See Also

- [Simple one-liner ViewBinding in Fragments and Activities with Kotlin](https://zhuinden.medium.com/simple-one-liner-viewbinding-in-fragments-and-activities-with-kotlin-961430c6c07c)
- [](https://developer.android.com/topic/libraries/view-binding?hl=ko)
- https://yoon-dailylife.tistory.com/57