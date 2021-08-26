# [DAY04] ViewModel, ListAdapter+DiffUtil

## 학습내용

- MVVM패턴의 핵심이라고 할 수 있는 ViewModel에 대해 학습하기

  - ViewModel의 개념

  - ViewModel과 생명주기

  - ViewModel의 사용목적

  - 직접 ViewModel 사용해보기

    

- RecyclerView에 ListAdapter+DiffUtil 활용하기

  

- MVVM에 대한 전반적인 개념 학습하기



## ViewModel

- UI관련 데이터 저장 및 UI로직을 처리 및 관리하기 위해 만들어짐

- LifeCycle 패키지에 포함된 것에서 알 수 있듯이 생명주기를 고려해서 동작하도록 구현됨

  

### ViewModel과 생명주기

- View의 생명주기는 안드로이드 FrameWork에 의해 관리됨

- 화면 회전이나 글씨 크기 변경 등 구성 변경 발생시 View는 Destroy되고 다시 재생성

- ViewModel은 View의 Lifecycle에 scoping되어 View가 완전히 종료될 때 까지

  객체가 유지됨

- View -> ViewModel -> Repository의 구조로 의존성이 단방향으로만 생성

- UI데이터 저장과 로직 처리를 View에서 분리함



### ViewModel의 사용 목적

- View는 최대한 모르게. 멍청하게(?)
- Android의 까다로운 View 생명주기 때문에 실수하기 쉬운 코드를 예방할 수 있음
- 테스트 코드 작성이 간편해짐
- 동일한 Activity에 attach된 Fragment간 데이터 공유가 편리함



### ViewModel 사용해보기

```kotlin
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.3.1' (최신버전)
```

- androidx.lifecycle.ViewModel 상속
- 상속한 클래스를 View에서 사용

- ```kotlin
  private val viewModel: MainViewModel by lazy{
  	// onCreate 이후에 호출되어야 함
  	ViewModelProvider(this)[MainViewModel::class.java]
  }
  ```

- ViewModel은 객체는 항상 ViewModelProvider를 통해 받아야 함

  

### ViewModel 사용 전과 후의 코드 비교

<전>

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private val count = 0L

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)

        setContentView(binding.root)
        initViews()
    }

    private fun initViews() {
        binding.increaseButton.setOnClickListener {
            count++
            binding.countText.text = count.toString()
        }
    }
}

```



<후>

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    private val viewModel: MainViewModel by lazy{
        ViewModelProvider(this)[MainViewModel::class.java]
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)

        setContentView(binding.root)
        initViews()
    }

    private fun initViews() {
        binding.increaseButton.setOnClickListener {
            viewModel.increaseCount()
        }
        viewModel.count.observe(this) {
            binding.countText.text = it.toString()
        }
    }
}

class MainViewModel: ViewModel() {
    private val _count = MutableLiveData<Long>().apply {
        value = 0
    }
    
    val count: LiveData<Long>
        get() = _count

    fun increaseCount() {
        _count.value = (count.value ?: 0) + 1
    }
}
```

<전>과 <후>의 코드를 보면 알 수 있듯이

ViewModel을 이용하면 View가 직접 데이터 처리를 하지 않는다.

데이터의 저장과 로직 처리는 모두 ViewModel에서 이루어지고 있고,

View는 뷰모델을 observe하다가 변화가 일어나면 단지 view를 업데이트 하는 역할을 한다.



즉, 그동안 View에서 UI 데이터의 저장과 로직처리를 모두 했다면

이 역할을 ViewModel로 분리시킨 것이다.







## RecyclerView+ListAdapter+DiffUtil

기존에는 RecyclerView의 Adapter는 RecyclerView.Adapter를 이용하여 구성하였다.



이는 RecyclerView에서 데이터가 변경되었을 경우 notifyDatasetChanged()를 사용하게 되는데

이 경우, 리스트 내의 데이터를 모두 바꾸게 된다.

따라서, 데이터가 매우 많을 경우 시간 지연이 발생하게 된다.



이러한 불편한 점을 해소하기 위해 ListAdapter + DiffUtil을 활용하게 된다.

DiffUtil은 현재 데이터 리스트와 교체할 데이터 리스트를 비교하여

변경이 필요한 부분만을 뽑아내어 변경을 하기 때문에

notifyDatasetChanged()보다 훨씬 빠른 시간내에 리스트를 변경할 수 있게 되기 때문이다.



그럼 RecyclerViewAdapter와 코드를 비교해보며 사용법을 알아보자.

```kotlin
// RecyclerView.Adapter 클래스 선언 부분
class QuotesPagerAdapter(private val quotes: List<Quote>, private val isNameRevealed: Boolean
): RecyclerView.Adapter<QuotesPagerAdapter.QuoteViewHolder>() {...

// ListView.Adapter + DiffUtil 클래스 선언 부분
class HouseListAdapter: ListAdapter<HouseModel,HouseListAdapter.ItemViewHolder>(diffUtil) { inner class ItemViewHolder(private val view: View): RecyclerView.ViewHolder(view) { ...

```

ListAdapter는 클래스 선언시 따로 리스트를 인자로 넘겨주지 않아도 된다.

다만, 리스트의 데이터를 교체해야 할 경우에는 adapter를 사용하는 클래스에서

```kotlin
adapter.submitList(it.books)
```

이렇게 리스트를 넘겨주어 교체하게 된다.



DiffUtil은 리스트 변경 전 후의 데이터를 비교하여 변경이 필요한 부분만 변경한다고 했는데,

그 기능을 하는 코드가 다음 코드이다.

```kotlin
companion object {
    val diffUtil = object: DiffUtil.ItemCallback<HouseModel>() {
        override fun areItemsTheSame(oldItem: HouseModel, newItem: HouseModel): Boolean {
            return oldItem.id == newItem.id
        }

        override fun areContentsTheSame(oldItem: HouseModel, newItem: HouseModel): Boolean {
            return oldItem == newItem
        }

    }
}
```

areItemsTheSame()은 두 아이템이 동일한 아이템인지 체크한다.

위에서는 id를 기준으로 두 아이템의 id가 같다면 동일한 아이템으로 체크하는것을 알 수 있다.



areContentsTheSame()은 두 아이템이 동일한 컨텐츠를 가지고 있는지 체크한다.

이 함수는 위의 areItemsTheSame()이 같은 경우에만 호출하게 된다.



이렇게 두 함수를 통해 변경이 필요한 부분에 대해서만 List변경을 하게 되는 것이다.



따라서 기존의 RecyclerViewAdapter보다 효율적이라고 볼 수 있기 때문에

앞으로는 RecyclerView를 구성할 때 ListAdapter+DiffUtil을 활용할 예정이다.