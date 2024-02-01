Optional이란 java 8 에서 처음 나온 기능으로 Null 값을 가질 가능성이 있는 객체를 감싸는 Wrapper 클래스 이다.

Null을 직접 다룰 경우에는 위험성이 커 Optional 객체를 사용하면 더욱 효과적인 처리가 가능하며, 코드가 Null-Safe 해지고 가독성이 좋아지며, 코드가 안정적이 된다.



먼저 Null이 될 수 있는 객체를 직접 다룰 시에 발생할 수 있는 문제점이다.

public String priceFormCar(Car car) {
return car.getOwner().getPhoneNumber();
}
위와 같이 자동차(Car) 객체의 주인(owner)의 휴대폰 번호 (phoneNumber)를 가져오는 메소드가 있다고 할 때,

만약 Car car가 null값으로 들어올 경우 owner 또한 null , phoneNumber 또한 null의 값을 가지면서

이 메소드의 호출부에 NullPointerException을 발생시킬 수 있는 위험성이 존재한다.







하지만 Optional은 null을 반환하면 오류가 발생할 가능성이 매우 높은 경우에 '결과 없음'을 명확하게 드러내기 위해 메소드의 반환 타입으로 사용되도록 매우 제한적인 경우로 설계되어 있기 때문에 이 Optional을 잘못 사용하게 되면 또다른 부작용들이 발생하게 된다.



Optional을 잘못 사용하였을 경우 나타날 수 있는 부작용에는

NullPointerException 대신 NoSuchElementException이 발생
시간적, 공간적 비용이 증가함
직렬화의 경우 문제점 발생
코드의 가독성을 떨어뜨림




앞서 Optional을 사용하면 코드의 가독성이 좋아진다고 말했는데 Optional을 사용할 경우 가독성이 떨어질 수 있다는것에 의문점이 들 수 있다.



이 이유는 Optional을 잘못 사용할 경우 NoSuchElementException와 NullPointerException을 둘 다 검사하고 처리해주어야 할 상황이 발생할 수 있기 때문에 단순히 null을 사용하였을때 보다 코드가 더 복잡해지고 불필요한 코드 글자수까지 늘어나게 된다.









따라서 Optional을 올바르게 사용하기 위해서는 아래의 가이드에 따라 사용해야한다

[올바른 Optional 가이드]
Optional 변수에 Null을 할당하지 말아라
값이 없을 때 Optional.orElseX()로 기본 값을 반환하라
단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
반환 타입으로만 사용하라


Optional의 사용방법과 자세한 메소드들은 다음에 알아보자.