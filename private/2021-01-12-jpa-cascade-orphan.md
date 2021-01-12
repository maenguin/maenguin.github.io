# 영속성 전이와 고아 객체
## 영속성 전이 : CASCADE
특정 엔티티의 영속 상태를 변경할때 연관된 다른 엔티티의 영속 상태도 같이 변경하고 싶을때 사용한다.  
```java
@Entity
public class Child {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Parent parent;
}
```
```java
@Entity
public class Paent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> childList = new ArrayList<>();
    
    private void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }
}
```
```java
Child child1 = new Child();
Child child2 = new Child();
Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

//PERSIST가 전이되어 parent만 persist해도 child1과 child2도 영속됨
em.persist(parent);
```
## 고아 객체 제거
부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제  
```java
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> childList = new ArrayList<>();
```
```java
Parent parent = em.find(Parent.class, id);
parent.getChildList().remove(0); //컬렉션에서 제거해도 delete 쿼리가 나가게됨
em.remove(parent); //모든 child가 제거되고 parent도 제거됨 (CascadeType.REMOVE와 동일)
```

## 주의사항
* 영속성 전이와 고아 객체 제거 모두 **특정 엔티티가 개인 소유** 일때만 사용하는것이 좋다.  
  단일 소유자, 즉 Parent와 Child의 관계 처럼 Child가 Parent에서만 참조되어질때만 사용해야한다.  
* @OneToXX 에서만 가능
## 생명주기
CascadeType.ALL + orphanRemoval = true 로 사용하게 되면 부모 엔티티를 통해서 자식의 생명 주기를 관리 할 수 있음  
> 도메인 주도 설계의 Aggregate Root 개념을 구현할 때 유용  
부모만 리포지토리를 만들고 자식은 만들지 않음  









