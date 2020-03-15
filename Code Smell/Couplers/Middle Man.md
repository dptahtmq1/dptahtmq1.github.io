# Middle Man

Description: 어떤 클래스의 절반 이상의 인터페이스가 기능을 다른 클래스에 위임하고 있는 경우
Payoff: 코드 사이즈를 줄일 수 있다.
무시할 수 있는 경우
- middle man이 클래스간 의존관계를 끊고 있는 경우
- Proxy나 Decorator와 같이 의도적으로 생성된 경우
Treatment: Remove Method
Inline Method
Replace Delegation with Inheritance