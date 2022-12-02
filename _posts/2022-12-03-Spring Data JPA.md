---
categories: Project
tags: [JPA, study, project, Java, summary]
---

# Spring Data JPA

## Repository.findById()
```java
List<Long> allByIdx = registryRepository.findAllByIdx();
List<Long> temp = new LinkedList<>();
temp.addAll(allByIdx);
        
List<Optional<Registry>> result = new ArrayList<>();

if (temp.isEmpty()) {
    Registry registry = new Registry("admin", "admin","admin");
    result.add(Optional.ofNullable(registry));
}

if (temp.toArray().length > 10) {
    for (int i = 0; i < 10; i++) {
        result.add(registryRepository.findById(temp.get(i)));
    }
} else {
    for (int i = 0; i < temp.toArray().length; i++) {
        result.add(registryRepository.findById(temp.get(i)));
    }
}
```
프로젝트 기능 수정을 하기 위해 작성했던 코드 중 일부 생략된 코드이다.

내가 의문이 들었던 것은 `registryRepository.findById`를 작성할 때 **Optional**로 줘야 하는 부분이다.

그냥 `List<Registry> result`로 하면 안되나? 싶었는데 Optional을 생략하면 에러가 떴고 IDE에서 Optional로 바꿔주었다.

그래서 그 이유를 찾아보게 되었다.  

<br><br><br>

### JPA
해당 Repository가 상속받고 있는 *JpaRepository* interface에 들어가봤다.

![image](https://user-images.githubusercontent.com/74857364/205247136-44aa662f-e8aa-4791-aa61-0f68620e7917.png)

*JpaRepository* 인터페이스에서 find와 관련된 메서드는 *CrudRepository*를 참고하라고 한다.

<br><br>

*CrudRepository* interface에 **findById** 메소드를 확인해보니 return값이 **Optional** 타입으로 고정되어있다.

![image](https://user-images.githubusercontent.com/74857364/205245676-7f3a6cf2-a344-44c7-8649-9f4591eccb04.png)

<br>

원인은 바로 *CrudRepository* interface의 return값이 Optional로 주어져서였다.

<br><br><br>

## JPA 구조
![image](https://user-images.githubusercontent.com/74857364/205320029-9efc8ac5-4e41-4fe8-8730-37f4c01ec16f.png)

JpaRepository는 PaingAndSortingRepository, QueryByExampleExecutor를 상속하고 있다.

<br>

***CrudRepository***가 조부모, **PagingAndSortingRepository**가 부모, *JpaRepository*가 자식의 관계이다.

우리가 만든 Repository는 JpaRepository를 상속받는다.  

<br>


![image](https://user-images.githubusercontent.com/74857364/205324571-295c24f8-4a6e-43f3-9781-eff22941e984.png){: width="50%"}

모든 인터페이스가 `<T, ID>` 두 개의 제네릭 타입을 사용하는 것을 볼 수 있는데, 

**T**는 Entity의 타입 클래스를, **ID**는 식별자(PK)의 타입을 의미한다. 

<br><br>

최상위 부모인 ***Repository***를 **CrudRepository**가 상속받아 확장시키고, 

이를 *PagingAndSortingRepository*가, 그리고 이를 또 JpaRepository가 상속받아 확장시킨 것을 볼 수 있다.

<br>

실제 주로 사용하는 것은 CRUD(Create, Read, Update, Delete) 작업을 위주로 하는 *CrudRepository* 인터페이스나

페이징 처리, 검색 처리 등을 할 수 있는 *PagingAndSortingRepository* 인터페이스다.

<br><br><br>

## CrudRepository

조부모 인터페이스인 CrudRepository에는 다음과 같은 메소드들을 제공한다.

![image](https://user-images.githubusercontent.com/74857364/205328108-beef9b8c-2b2c-4413-b521-7aa0940f7448.png){: width="60%"}


<br><br><br>
  
*reference*                                             
[[JPA] 리턴타입이 옵셔널(Optional)인 이유는?](https://sowon-dev.github.io/2022/07/10/220711JPA-optional/)           
[[Spring Framework] Spring Data JPA Repository](https://memories95.tistory.com/136)             
[[JPA] CrudRepository vs JpaRepository](https://codify.tistory.com/103)                   
[[Spring Data JPA] Repository 생성](https://sky-h-kim.tistory.com/22)               
[[JPA] JpaRepository vs CrudRepository(1) - Paging과 Sorting / QueryExampleExecutor](https://devonce.tistory.com/53)                      
