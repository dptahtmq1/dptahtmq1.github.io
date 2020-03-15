# Message Chains

Description: 한 객체에 제 2의 객체를 요청하면, 제 2의 객체가 제 3의 객체를 요청하고, 이런 식으로 연쇄적 요청이 발생하는 경우
ex. a.getB().getC().getD().getData()
Payoff: 연결된 클래스 사이의 의존도를 줄인다
비대한 코드 양을 줄일 수 있다
→ 과도한 Hide Delegate는 실제 동작이 어디에서 이루어지는지 찾기 힘들게 한다. 또한 Middle Man Smell을 생성할 수 있다.
Treatment: Hide Delegate
Extract Method
Move Method