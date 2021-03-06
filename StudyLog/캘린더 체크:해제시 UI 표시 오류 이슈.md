## 화면에 나타낼 캘린더 체크/해제 시 UI 표시 오류

### 오류 현상 :

8월 -> 9월 -> 10월로 이동 후 10월 뷰에서 notifyDataSetChanged를 수행하면 10월달의 이벤트들이 8, 9월 달력에도 표시되는 오류 발생.

</br>

### 원인 및 분석: 

뷰페이저는 현재 페이지 전 후의 페이지를 갖고 있는 특징이 있다.

예를 들어, 8월의 달력으로 이동하게 되면 7월, 9월 페이지도 계속 갖고 있는다.

</br>

이러한 뷰페이저의 특성과 ShareViewModel과 관련된 오류였다.

</br>

오류를 고치기 전에는 MonthFragment(각 월뷰를 나타내는 프레그먼트)는 ShareViewModel과 연결되도록 구현하였는데 이것이 원인이었다.

</br>

이렇게 설계를 한 이유는 ShareViewModel에서는 MonthFragment에서 넘겨주는 날짜를 기준으로 LocalDB에서 이벤트들을 불러오게 되며 

(8월을 넘겨주면 8월 이벤트를, 9월을 넘겨주면 9월 이벤트를)

이 이벤트들을 MonthFragment의 어댑터에 적용을 함으로써 각각의 월에 맞는 이벤트가 각각의 MonthFragment에 표시되도록 하기 위함이었다.

</br>

처음에 ShareViewModel을 사용한 이유는, 하나의 Fragment마다 ViewModel을 각각 갖고 있기 보다는, 어차피 같은 형태의 ViewModel을 갖고있기 때문에 ViewModel을 돌려쓰기 위한 목적이었다.

</br>

하지만 이는, 뷰페이저를 제대로 이해하지 못한 상태에서 진행한 설계였다.

</br>

뷰페이저는 현재 페이지의 전 후의 페이지를 갖고있는 특징이 있기 때문에

8월 페이지로 이동을 하게되면 7월, 9월 페이지도 함께 갖고 있는다.

</br>

그리고, 8월에서 7월이나 9월로 이동하게 되면

7, 9월의 페이지를 새로 생성하는 것이 아니라 미리 갖고있는 7, 9월 뷰를 보여주게 된다.

</br>

처음 초기에 프레그먼트를 생성할 때는 8월은 8월 날짜를 ShareViewModel에 넘겨주어 성공적으로 8월에 맞는 이벤트를 받아오며

9월은 9월 날짜를, 10월은 10월 날짜를 ShareViewModel에 넘겨주어 성공적으로 각각의 달에 맞는 이벤트를 받아올 수 있다.

</br>

하지만 스와이프를 하여 8, 9, 10월 프레그먼트가 이미 생성된 상태에서, 가장 마지막으로 생성된 프레그먼트가 10월일 때, 8, 9, 10 프레그먼트 모두 10월로 날짜 설정이 된 ShareViewModel을 참조하고 있게 된다.

</br>

그렇게 되면, 10월 프레그먼트에서 캘린더에 변화를 주어야 하는 상황이 생겨 notifyDataSetChanged를 하게 되면, 8월 9월도 10월로 설정된 ShareViewModel을 바탕으로 데이터를 다시 셋팅하게 되므로 8월 9월에도 10월의 이벤트들이 나타나는 오류가 발생한 것이다.

</br>

이 오류를 해결하기 위해 Fragment와 Adapter, ViewModel 각각에 로그도 찍어보고 디버깅도 해보면서 원인을 찾아보고자 했다. 로그들을 분석한 결과, 8월 달력이 단독으로 생성되었을 때는 정상적으로 작동을 하다가, 9월, 10월 달력으로 이동을 하게되면 8월에 8월 이벤트를 적용 => 8, 9월에 9월 이벤트를 적용 => 8, 9, 10월에 10월 이벤트를 적용하는 로그가 나타났다.

</br>

그래서 가장 최신에 생성한 날짜를 기준으로 모든 달에 이벤트가 적용된 것임을 깨달았고 이는 ShareViewModel과 연관이 있음을 직감하였다.

</br>

그래서 다음과 같이 해결하였다.

</br>

### 해결 :

각각의 MonthFragment가 ShareViewModel에서 이벤트들을 가져오도록 하지말고, 각각의 ViewModel을 가지게 함으로써 8월달의 Fragment는 8월의 ViewModel객체를 갖게하고 9월은 9월 ViewModel객체를 갖게하였다.

</br>

이렇게 하게되면, 9월의 이벤트에 변화가 생겨 notifyDataSetChanged를 하게되더라도 8월과 10월의 ViewModel과 어댑터에는 영향을 주지 않게 되므로 각각의 이벤트들이 겹치지 않고 정상적으로 UI에 표시될 수 있게 되는 것이다.