# Feature Envy

Description: 어떤 메서드가 자신이 속하지 않은 클래스에 더 많이 접근하는 경우
데이터와 그것을 사용하는 function은 대부분 변경이 함께 이루어진다. 동시에 변경이 이루어지는 것들은 같은 장소에 위치해야 한다.
Payoff: 코드 조직력 개선 및 중복 제거
Strategy / Visitor Pattern과 같이 overriding을 통해 기능을 데이터와 분리하는 경우에는 허용
Treatment: Move Method
Extract Method