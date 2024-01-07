---
published: true
title:  "GoF Design Pattern - Adapter Pattern"
categories:
  - architecture
---

## 요약
서로 인터페이스가 다른 경우 어댑터 패턴을 사용해 해결할 수 있습니다. 구현 방법은 총 두가지가 있으며, 자신이 처한 상황에서 가장 실용적인 방법을 선택해 구현하시면 됩니다.


## 서론
김영한님의 Spring MVC 강의에서 어탭터 패턴을 구현하는 단계가 있었습니다. 백기선님의 GoF 디자인 패턴 강의 중 어댑터 패턴이 있다는게 생각이 나서 먼저 수강하고 정리해 봤습니다. 어탭터 패턴이 무엇이고 2가지 구현 방법에 대해서 알아보고 글을 마치겠습니다.


## 어댑터 패턴이란
어댑터 패턴이란 서로 다른 인터페이스를 구현한 클래스끼리 협업할 수 있도록 만들어주는 디자인 패턴입니다. 많이 사용하는 예시로 한국에서는 220V를 사용하지만 외국은 110V를 많이 사용합니다. 이렇게 서로 다른 인터페이스(220V, 110V)끼리 협력하려면 중간에 어댑터를 끼우면 됩니다.

![Adapter1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/디자인패턴/어댑터패턴/adapter1.png?raw=true)


## 구현 방법 1 - 어댑터 클래스 생성

아래의 예시 코드에서 Client는 LoginHandler이고 Adaptee는 Account와 AccountService입니다. 두 코드는 인터페이스가 달라 코드를 직접 수정하지 않는 이상 서로 협력해 사용할 수 없습니다.

### Before

~~~java
public class LoginHandler { // Client
    private final UserDetailsService userDetailsService;

    public LoginHandler(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    public String login(String username, String password) {
        UserDetails userDetails = userDetailsService.loadUser(username); // accountService.findAccountByUsername으로 직접 수정해야 함. 수정해도 Account를 반환하기 때문에 서로 협력이 안됨.
        if (userDetails.getPassword().equals(password)) {
            return userDetails.getUsername();
        } else {
            throw new IllegalArgumentException();
        }
    }
}

public interface UserDetailsService { 
    UserDetails loadUser(String username);
}

public interface UserDetails {
    String getUsername();
    String getPassword();
}
~~~

~~~java
@Getter @Setter
public class Account {
    private String name;
    private String password;
    private String email;
}

public class AccountService {
    public Account findAccountByUsername(String username) {
        Account account = new Account();
        account.setName(username);
        account.setPassword(username);
        account.setEmail(username);
        return account;
    }

    public void createNewAccount(Account account) {
    }

    public void updateAccount(Account account) {
    }
}
~~~

위의 문제를 해결하려면 어댑터 클래스를 만들면 됩니다. 어댑터 클래스를 만드는 방법은 Adaptee 클래스를 이용해 Client의 인터페이스를 구현하면 됩니다. 아래처럼 코드를 작성하면 생성자에 AccountUserDetailsService를 넣어 Client의 코드와 Adaptee의 코드 변경 없이 재사용할 수 있습니다.

### After
~~~java
public class AccountUserDetails implements UserDetails{
    private final Account account;

    public AccountUserDetails(Account account) {
        this.account = account;
    }

    @Override
    public String getUsername() {
        return account.getName();
    }

    @Override
    public String getPassword() {
        return account.getPassword();
    }
}

public class AccountUserDetailsService implements UserDetailsService {
    private final AccountService accountService;

    public AccountUserDetailsService(AccountService accountService) {
        this.accountService = accountService;
    }

    @Override
    public UserDetails loadUser(String username) {
        Account account = accountService.findAccountByUsername(username);
        return new AccountUserDetails(account);
    }
}
~~~


## 구현 방법 2 - Adaptee 클래스가 Client의 인터페이스를 구현하도록 수정

이 방법은 Adaptee의 클래스가 Client의 인터페이스를 직접 구현하는 방법입니다. 이 방법의 장점은 별도의 어댑터 클래스를 만들지 않아도 되며, 복잡도와 코드의 수가 줄어듭니다. 하지만 단일 책임 원칙 입장에서 본다면 어댑터 클래스를 만드는게 더 좋습니다.

이전에 비해 기존의 코드를 Client의 인터페이스를 구현하도록 코드가 바뀐다는 점인데 장점은 별도의 어댑터 클래스를 만들지 않아도 됨. 복잡도가 내려가고 코드의 수가 줄어듬. 하지만 단일 책임 원칙 입장에서는 어댑터 클래스를 만드는게 더 좋음. 하지만 원칙에 목 매달지 말고 현재 하고 있는 프로젝트에서 이 방법이 실용성이 더 높다고 생각할 경우 유연하게 바꿔라.

~~~java
@Getter @Setter
public class Account implements UserDetails {
    private String name;
    private String password;
    private String email;

    @Override
    public String getUsername() {
        return this.name;
    }

    @Override
    public String getPassword() { // 직접 코드를 추가
        return this.password;
    }
}

public class AccountService implements UserDetailsService {

    public Account findAccountByUsername(String username) {
        Account account = new Account();
        account.setName(username);
        account.setPassword(username);
        account.setEmail(username);
        return account;
    }

    public void createNewAccount(Account account) {
    }

    public void updateAccount(Account account) {
    }

    @Override
    public UserDetails loadUser(String username) { // 코드를 추가
        return findAccountByUsername(username);
    }
}
~~~

## 결론
두 클래스가 구현하고 있는 인터페이스가 다를 경우 어댑터 패턴을 사용하여 해결할 수 있습니다. Client와 Adaptee의 코드를 수정할 수 없는 상황이면 어댑터 클래스를 생성하고 구현해 해결할 수 있습니다. 만약 Adaptee의 코드를 수정할 수 있다면 직접 Client의 인터페이스를 구현하도록 수정해 해결할 수 있습니다. 두 가지 방법 모두 장점과 단점이 있기 때문에 자신이 처한 상황에서 가장 실용적인 방법을 선택하시기를 권장합니다.