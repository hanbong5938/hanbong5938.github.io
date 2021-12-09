StringBuilder는 jdk 1.5에서 추가된 클래스이다.

StringBuffer와 동작하는 방식은 같으나 syncronized의 차이를 가진다.

thread safe한 작업은 StringBuffer를 사용하고 그 외의 경우에선 StringBuilder를 사용하는 것이 좋다.

String은 jdk 1.5 이후 + 연산이 되면 StringBuilder가 컴파일러에 의해 StringBuilder로 변환되며 append() 가 사용되도록 적용되었다.

반복문의 경우 new StringBuilder()로 객체를 계속 생성하므로 효율적이지 않다.

반복문이 아닌 경우에는 String의 + 연산자를 사용하여도 컴파일러가 변환시켜준다.  

String의 길이가 긴경우 + 를 사용하는것 또한 new StringBuilder()를 생성하는데 성능에 영향을 미칠 수 있다.
