# [DAY02] Retrofit(With Kotlin)

## 학습내용

- Kotlin을 이용한 Retrofit 사용법 숙지
- Retrofit을 이용해 인터파크 도서 api 데이터 가져오기
- 받아온 api 데이터를 바탕으로 안드로이드 뷰 그려보기



## 코틀린에서 Retrofit 사용

먼저 사용하기 전에, 자바와 동일하게 gradle에 코드를 추가하여 셋팅을 하자.

```kotlin
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```



그럼 본격적으로, Retrofit을 이용해 인터파크 도서 api의 데이터를 받아오자.

먼저, 이번 학습에서는 데이터를 받아오기만 할 것이기 때문에 `GET`만을 이용하였다.

GET이외에도 `POST`, `PUT`, `PATCH`, `DELETE`등이 존재하는데

- POST는 자원을 생성(Create)할 때
- PUT은 자원을 수정할 때 ( 전체 수정 )
- PATCH은 자원을 수정할 때 ( 일부 수정)
- DELETE은 자원을 삭제할 때

사용하게 된다.



먼저, 나는 베스트셀러 목록을 받아올 함수와 

키워드 검색 결과로 책 목록을 받아올 함수를 만들었다.



@GET( " 여기에 baseurl을 제외한 api 주소를 기입하면 된다 ")

getBestSellerBooks 함수는 베스트셀러 책 목록을 가져올 목적으로 만든 함수이며

받아올 형식(output)은 json으로 고정하고, 카테고리(categoryId)는 국내 도서만 받아올 것이기 때문에 100으로 고정시켰다.



getBooksByName 함수는 받아올 형식은 json으로 고정이지만, 검색 키워드는 사용자가 직접 입력하기 때문에 항상 바뀌므로, keyword 값에 따라 바뀌도록 @Query("query")로 전달하도록 하였다.



이렇게 인터페이스에 함수를 만들어 놓으면

실제 api를 호출할 액티비티에서 편리하게 이 함수를 호출하여 결과값을 받아올 수 있다.

```kotlin
interface BookService {

    @GET("/api/bestSeller.api?output=json&categoryId=100")
    fun getBestSellerBooks(
            @Query("key") apiKey: String
    ): Call<BestSellerDto>

	@GET("/api/search.api?output=json")
    fun getBooksByName(
            @Query("key") apiKey: String,
            @Query("query") keyword: String
    ): Call<SearchBookDto>
}
```



위 코드를 보면, 실제 받아올 객체가 BestSellerDto와 SearchBookDto임을 알 수 있는데

이 두 Dto는 data class로 만든 객체이다.

작성한 BestSellerDto의 데이터 클래스는 다음과 같다.

```kotlin
data class BestSellerDto(
    @SerializedName("title") val title: String,
    @SerializedName("item") val books: List<Book>
)
```

api의 수많은 데이터 중, 내가 받아오고 싶은 것들만 추려서 데이터 클래스로 만들었다.

@SerializedName을 통해 실제 api에서 전달해주는 파라미터 명을 일치시킨 후

뒤의 변수명은 내가 짓고싶은 것으로 지으면 되는데, 어느정도 연관성 있게 짓는것이 좋다.



그럼 이제, 액티비티에서 위의 자료들을 가지고 실제 api를 호출하여 데이터를 얻어와 보자.

먼저 다음과 같이 Retrofit을 빌드하여 retrofit이라는 변수에 저장을 해주자.

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://book.interpark.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

그리고, 이 빌드한 retrofit을 create() 하여 생성을 하자.

```kotlin
bookService = retrofit.create(BookService::class.java)
```

그럼 이제 아까 BookService 인터페이스 안에 정의해두었던 함수들을 호출하여 사용할 수 있다.

```kotlin
bookService.getBestSellerBooks(getString(R.string.interParkAPIKey))
        .enqueue(object: Callback<BestSellerDto> {
            override fun onResponse(call: Call<BestSellerDto>, response: Response<BestSellerDto>) {
                if (response.isSuccessful.not()) {
                    return
                }
                response.body()?.let {
                    Log.d(TAG, it.toString())
                    it.books.forEach { book ->
                        Log.d(TAG, book.toString())
                    }
                    adapter.submitList(it.books)
                }
            }

            override fun onFailure(call: Call<BestSellerDto>, t: Throwable) {
                // TODO 실패처리
            }

        })
```



Retrofit을 사용하는 부분에 있어서는 자바와 크게 다른점은 없는 것 같았다.

기본적인 자바와 코틀린의 함수 정의 방법이나 호출, 매개변수, 널처리 등등에 있어서 차이가 났던 부분만 신경 써주면 금방 적용할 수 있을 것 같다.
