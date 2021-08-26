# [DAY03] ViewBinding, Local DB

## 학습내용

- ViewBinding에 대해 학습한 후, 직접 앱에 적용시키기
- 안드로이드 Room을 이용하여 Local DB 활용하기
- ListAdapter + DiffUtil 사용하여 RecyclerView 구성하기



## ViewBinding

ViewBinding(뷰 바인딩)은 뷰와 상호작용하는 코드를 보다 쉽게 작성할 수 있는 기능이다.

뷰바인딩을 사용함으로써, 기존에 사용하던 findViewById를 대체할 수 있다.



뷰 바인딩을 이용하기 위해서는 build.gradle에 다음과 같이 명시하여야 한다.

```kotlin
viewBinding {
    enabled = true
}
```

그럼 이제 실제로 뷰 바인딩을 이용해보자.

먼저 나는 메인 액티비티에서 사용을 하려고 한다.



기존에는 MainActivity와 activity_main_xml을 연결하기 위해

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
```

이렇게 하여 activity_main 레이아웃 파일과 액티비티를 연결하였고

```kotlin
button = findViewById(R.id.button)
```

이러한 형식으로 뷰를 연결했을 것이다.



하지만 뷰 바인딩을 이용하면 다음과 같이 코드가 바뀐다.

```kotlin
private lateinit var binding: ActivityMainBinding
....
 override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
     	setContentView(binding.root)
```

binding이라는 변수에 Activity 뷰 전체가 연결되어 들어가는 형식이다.

그래서 binding.root를 하게 되면 activity_main 뷰 전체가 해당이 되고

binding.button을 하게 되면 activity_main 뷰 안의 버튼을 가져올 수 있게 된다.



ActivityMainBinding은 임의로 이름을 지은것이 아니라,

레이아웃 파일의 이름에 맞게 탄생한다.

activity_main.xml 이면 ActivityMainBinding이 되는 것고

item_book.xml 이면 ItemBookBinding이 되는 것이다.



처음 사용해보는 거라 아직 익숙하진 않지만,

가독성 측면이나 사용성 측면에서 훨씬 편리하다. 앞으로도 뷰 바인딩을 이용해 작성해보려고 한다.



## LocalDB

안드로이드의 **Room**을 이용해 LocalDB를 구성하였다.

Room에는 다음과 같은 3가지 개념이 존재한다.

- Database (데이터베이스)
- Entity ( 데이터베이스 내의 테이블 )
- DAO ( 데이터베이스에 접근하는 함수. insert, update, delete 등등 )



이 개념을 가지고 Room을 이용해 보자.

먼저, Room을 이용하기 위해선, build.gradle에 다음과 같이 추가하자.

```kotlin
implementation 'androidx.room:room-runtime:2.2.6'
kapt 'androidx.room:room-compiler:2.2.6'
```



그리고 Entity를 구성해보자.

다음과 같이 History라는 데이터클래스 파일을 만들고 class위에 Entity 어노테이션을 추가하여

Entity로 사용할 것을 명시하였다.

```
@Entity
data class History (
    @PrimaryKey val uid: Int?,
    @ColumnInfo(name = "keyword") val keyword: String?
)
```



그리고 DAO를 만들어 데이터베이스 테이블에 쿼리로 접근할 수 있도록 인터페이스를 만들자.

다음과 같이 @Dao 어노테이션을 추가하여 Dao로 사용할 것을 명시한 후

HistoryDao 인터페이스를 만들어 주었다.

```kotlin
@Dao
interface HistoryDao {

    @Query("SELECT * FROM history")
    fun getAll() : List<History>

    @Insert
    fun insertHistory(history: History)

    @Query("DELETE FROM history WHERE keyword == :keyword")
    fun delete(keyword: String)
}
```



마지막으로 AppDatabase를 만들어주자.

AppDatabase는 실질적으로 Room을 구현하는 부분이며 RoomDatabase를 상속받는다.



코드의 @Database(...) 부분의 version은 테이블이 추가되거나 변경되면 바꿔주어야 한다.

이 때, 마이그레이션 과정이 필요하다.

그 마이그레이션 함수가 밑에 위치한 Migration(1,2) 이다. version1 -> version2



AppDatabase에 추상 클래스로 historyDao와 reviewDao를 명시함으로써

각각의 Dao에 접근할 수 있도록 했다.



마지막으로 Builder를 통해 실질적으로 RoomDatabase를 만들어 주었다.

```
@Database(entities = [History::class, Review::class], version = 2)
abstract class AppDatabase: RoomDatabase() {
    abstract fun historyDao(): HistoryDao
    abstract fun reviewDao() : ReviewDao
}

val migration_1_2 = object : Migration(1,2) {
        override fun migrate(database: SupportSQLiteDatabase) {
            database.execSQL("CREATE TABLE 'REVIEW' ('id' INTEGER, 'review' TEXT," + "PRIMARY KEY('id'))")
        }
    }

fun getAppDatabase(context: Context): AppDatabase {
	return Room.databaseBuilder(
        context, AppDatabase::class.java, "BookSearchDB"
    )
        .addMigrations(migration_1_2)
        .build()
}
```



이렇게 만든 Local DB를 액티비티에서 실제로 사용해보자.

```kotlin
private lateinit var db: AppDatabase
db = getAppDatabase(this)
db.historyDao().getAll()
```

이렇게 함으로써 위에서 만들어놓은 AppDatabase와

각각의 Dao, Entity에 접근하여 원하는 DB를 얻을 수 있게 된다.



안드로이드를 이용해 Local DB를 이용해본 것은 이번이 처음이었다.

하지만, 서버를 이용하지 않고 자체적으로 DB를 생성하기 때문에

서버의존도와, 서버의 용량을 줄일 수 있게 되기 때문에

실무에서도 효율적으로 사용할 수 있을 것 같다.



능숙하게 사용할 수 있도록 계속 연습해 나가야겠다.