---
layout: post
title: "@SneakyThrows가 뭐지??"
tags:
  - Springboot
  - Java
categories:
  - 스프링부트
published: true
---

어느 한날 코딩을 하고 있었습니다.  
명시적으로 예외가 발생하는 코드에서 IDE가 다음과 같은 코드 수정을 제안해주었습니다.  
![소스코드](/assets/2023-09-12/code.png)
1. 'catch' 절을 추가합니다.
2. 'IncompatibleClassChangeError'가 있는 catch에 'IOException'을(를) 추가
3. 예외를 메서드 시그니처에 추가
4. try/catch로 둘러싸기
5. 메서드 상단에 `@SneakyThrows` 어노테이션 추가
  
여기서 5번 항목을 선택해보면, 
![](/assets/2023-09-12/code2.png)
이런식으로 메서드 상단에 `@SneakyThrows`가 추가되며, IDE에서 밑줄이 사라졌습니다.  


## @**SneakyThrows** 어노테이션
### JavaDoc
일단 어노테이션의 JavaDoc을 확인 해 보겠습니다.
![](/assets/2023-09-12/sneakythrows-docs.png)
    
```java
Example:
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
      return new String(bytes, "UTF-8");
  }
  
Becomes:
  public String utf8ToString(byte[] bytes) {
      try {
          return new String(bytes, "UTF-8");
      } catch (UnsupportedEncodingException $uniqueName) {
          throw useMagicTrickeryToHideThisFromTheCompiler($uniqueName);
          // This trickery involves a bytecode transformer run automatically during the final stages of compilation;
          // there is no runtime dependency on lombok.
      }
```
이런식으로 컴파일 마지막 단계에서 자동으로 수정되는 듯합니다.
주석에서는 이런 방식을 *trickery*(속임수)라고 표현을 해두었네요

### 예시코드

예시로 다음과 같은 코드를 가져왔습니다.
```java
public void throwsCheckedException() {  
  throw new IOException("IO exception thrown");  
}
```
해당 코드를 실행해보면, 
```java
error: unreported exception IOException; must be caught or declared to be thrown
    > throw new IOException("IO exception thrown");
```
이런식으로 예외처리를 명확히 하라는 메세지와 함께 컴파일 되지 않았습니다. 이번에는 `@SneakyThrows`를 사용 해 보겠습니다. 

```java
@SneakyThrows // 추가된 어노테이션
public void throwsCheckedException() {  
  throw new IOException("IO exception thrown");  
}
```

해당 코드를 실행해보면,  
![](/assets/2023-09-12/err.png)
컴파일이 정상적으로 진행되며, Checked 예외가 잘 발생한것을 확인할 수 있습니다.

이로써 `@SneakyThrows`는 Checked Exception(예외처리가 필요한 예외들) 처리를 무시하고 실행하게 만드는 것으로 이해했습니다.
