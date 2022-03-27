# Redis Simple

생성일: 2022년 3월 6일 오후 11:12
태그: 익명

[[10분 테코톡] 🤔디디의 Redis](https://www.youtube.com/watch?v=Gimv7hroM8A)

---

# Redis란

Redis란? Remote dictionary server 의 약자이며

**Key Value** 형태로 이루어진 외부 서버 이다.

- Remote Dictionary Server
- Database, Cache, Message Broker
- In-memory Data Structure Store
- Supports rich data structure

---

# Redis를 어디다 쓰는가?

Redis 는 In-memory DB (Cache DB) 이다 즉

SSD 나 HDD 에 접근하는것보다 보다 빠르게 저장된 데이터에 접근이 가능하다

**일반 RDB 보다 더 빠르게 더 자주접근해야하고 덜 자주 바뀌는 데이터를 적재해야 하는 곳에 사용한다.**

**여러 서버에서 같은 데이터를 공유할 때**

---

# Redis Collection

## 1. Strings

Java 의 Map.Entry 처럼 Key Value 로 이루어진 자료구조

Key - Value

## 2. List

Java 의 LinkedList 처럼 이루어진 자료구조

Key - List

## 3. Set

Java 의 HashSet 처럼 이루어진 자료구조

Key - Set 

## 4. Sorted Set

Java의 TreeSet 처럼 이루어진 자료구조

Key - Set 

## 5. Hash

Java의 HashMap 또는 Object로 이루어진 자료구조 

---

# Redis 사용 시 주의사항

Single Thread 서버 이므로 시간 복잡도를 고려하고 사용해야한다.

Single Thread 이므로 하나의 요청이 끝나기 전 까지 다른 요청들은 대기된다.

**즉 O(N) 의 명렁어는 꼭 주의해서 사용해야한다**

> **Keys, Flush, GetAll 연산 등이 있다.**
>