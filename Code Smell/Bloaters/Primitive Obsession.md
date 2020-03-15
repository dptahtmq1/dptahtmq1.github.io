# Primitive Obsession

Description: 객체보다 Primitive를 사용하는 경우
Type Code를 위해 상수를 사용하는 경우
Data Array에서 필드 이름을 상수로 표현하는 경우(ex. data[NAME])
Payoff: Primitive 대신 객체를 사용하면 코드가 유연해지고, 이해하기 쉬워지며 조직적으로 변경된다.
데이터의 동작이 하나의 클래스로 모여지고, 상수와 배열이 왜 사용 되었는지 고민하지 않아도 된다.
Treatment: Replace Data Value with Object
Introduce Parameter Object
Preserve Whole Object
Replace Type Code with Class
Replace Type Code with Subclasses
Replace Type Code with State/Strategy
Replace Array with Object