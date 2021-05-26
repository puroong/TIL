# Declaration vs Definition

* Declaration(선언)
    * 메모리에 할당하지 않고 이름만 알려주는 경우

* Definition(정의)
    * 메모리에 할당된 경우

## Example

```cpp
extern int a; // 전역변수 선언
int add(int a, int b); // 함수의 본문이 없으므로 선언


int add(int a, int b) { // 함수 정의
    return a+b;
}

int a = 1, b = 10; // 변수 정의
```

