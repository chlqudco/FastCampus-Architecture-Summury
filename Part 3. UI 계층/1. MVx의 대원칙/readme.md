- UI계층의 구현에는 많은 합의가 있었다. 이를 UI 계층의 대 원칙이라 부르는데 어쨋든 MVx임
	- 어떤 경우던 Model은 분리하라
		- MVx에서의 맨 앞의 M의 모델을 의미
		- 모델은 데이터를 다루는 전반적인 로직을 의미함
		- 엔티티나 데이터클래스의 집합을 말하는 것이 아님. 방금 말한 데이터를 다루는 모든 것을 의미
		- 넓은 범위로는 UI의 외적인 모든걸 M이라고 부르기도 함
	- 뷰의 역할을 할 수 있는 한 분리시켜야 함
		- MVC라고 해도 뷰는 context를 이용한 디바이스 의존적인 일만 처리하도록 하자
		- 이것에 벗어나는 일은 다른쪽으로 넘기도록 해보자
	- 안드로이드에서 발생할 수 있는 특수한 상황들을 잘 처리할 수 있는 체계가 필요함
		- context를 사용해야 하는 처리, 생명주기 이벤트 처리 등등 
		- 또한 다크모드를 킨다던가, 폰트 사이즈를 바꾸는 config 변화에도 유연하게 처리해야 함
		- 될 수 있는 한 많은 부분이 테스트 가능하도록 만들어 져야 함
		- 디바이스가 굉장히 다양하기 때문에 최대한 많이 자동화를 해야 함

---

- MVC 패턴
	- 모델 : 저장, 서버 요청, 데이터 응답 등등의 처리
	- 뷰 : 유저에게 보여주는 화면 담당. 유저 이벤트 처리 담당
	- 컨트롤러 : 모델과 뷰를 마치 레고블럭 마냥 조합해주는 애
		- 유저가 입력을 했을 때 미리 받아서 처리할 수도 있음.
		- 문자 그대로 컨트롤 타워가 됨
		- 컨트롤러는 결정권을 갖고 있다고 말함
	- 모델과 뷰는 직접 연결할 필요가 전혀 없음
		- 처음부터 이러진 않았고 논의 하에 합의 함
	- 에러의 처리도 컨트롤러가 담당함
	- 플랫폼을 막론하고 유용하게 적용되는 패턴임
		- 특히 웹앱에서 잘 작동함
		- 각자 역할 분할이 된 클래스를 제공하고 있고 컨트롤러가 이를 잘 조직화하기 때문
		- 특히 비즈니스 로직도 분화가 잘 될 수 있음
---

- 안드로이드의 MVC, 좋은 얘기를 들어본적이 있나?
- 왜 잘 동작하지 않을까?
	
- 모바일 환경의 문제
	- 비동기 처리가 굉장히 복잡함, 유저의 이벤트, 시스템 이벤트 등등을 컨트롤러가 모두 처리하기 힘들다.
		- 컨트롤 타워의 역할로만으로는 부족해진다. 할일이 막대해진다.
	- 라이프 사이클 처리가 까다로움
		- 얘도 마찬가지로 컨트롤러가 힘들어짐
	- UI 로직의 분리가 어려움
		- 웹의 html에서는 뷰가 컨트롤러와 완전히 독립된 형태로 UI 로직을 구현 가능하지만, 모바일은 불가능
		- 물론 안드로이드도 xml이 있지만 그 안의 복잡한 로직(UI처리 로직)은 결국 컨트롤러가 손봐야함
		- 예를들어 리사이클러뷰가 있음. 결국 xml만으로는 동작하지 못함.

- 근본적인 안드로이드의 문제
	- 액티비티은 작은 앱과 같다.
		- 그 안에 모든 역할을 갖고 있다. 심지어는 컨트롤러 역할도 갖고 있다.
		- 뷰안에서 일어나는 모든 역할을 다 해버리기 때문에 뷰와 컨트롤러의 역할을 이미 다 갖고 있다.
		- 이 문제 때문에 Fat Activity가 되어버림 -> 유지보수, 테스트, 확장 모두 어려움
	- 유닛 테스트를 만들기가 매우 까다로움
		- 액티비티 자체가 context이기 때문에 떼어내기 힘들다

---

- 어떻게 해결할까?
	- 어느정도 해결은 가능하다!
	- 다른 패턴을 도입하는 해결법도 물론 있다. 그러나 기존의 패턴의 문제점을 모르는 상태에서 맹목적으로 다른 패턴을 쓰는건 근본적인 문제를 해결할 수 없다
	
- 해결책 1. 뷰의 분화
	- 안드로이드의 좋은 점은 include 태그를 통해 xml 정의를 분할 할 수 있다는 것이다		 
	- 나눈 뷰 들은 액티비티가 아닌 별도의 컨트롤러를 통해 제어하자
	- 그러나 config 변화나 프로세스의 변경에 대해 대비해야 한다는 문제점이 있다.
	- 힐트를 이용하면 생명주기나 config 처리에서 살아남을 수 있긴 함

- 여전히 남는 문제
	- 사용자 이벤트와 외부 이벤트 등의 효과적인 처리가 여전히 어려움
	- 그놈의 context 때문이다!
		- 뷰컨트롤러의 상당수 동작을 위해선 결국 context가 필요함
		- Fragment에 연결된 뷰컨트롤러는 설정 변경(config change)에서 살아남기 매우 힘듬
		- 또한 테스트도 어려움
	- 해결법 : MVC를 쓰지 말자
		- 뷰와 컨트롤러는 완전히 분리해야 한다
		- 특히 컨트롤러는 context 없이도 동작을 수행할 수 있어야 함	

- non-MVC의 설계 전략을 살펴보자
	- non-MVC에서 액티비티는 뭐하는 애냐?
		- 컨트롤러지, 그러나 생성만 담당하는 아주 제한적인 친구가 되어야 함
		- 뷰의 역할을 갖는건 바람직하지 못하..다
	- MVP의 궁극적 접근법
		- 액티비티에서 뷰와 컨트롤러의 역할을 최대한 빼앗아서 뷰와 프레젠터로 넘긴다
		- 액티비티는 단지 객체 생성 및 순수한 흐름 관리 역할 위주
	- MVVM/MVI 등의 접근점
		- 단방향 데이터 흐름 : 컨트롤러에서 뷰의 방향으로 데이터 흐름을 이벤트 수신의 형태로 구현 
		- 뷰와 모델의 의존성을 극도로 떼어내기 위함
		- 뷰 로직은 최대한 data binding으로 구현
