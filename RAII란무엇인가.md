# RAII란 무엇인가
RAII는 “자원을 객체의 수명에 묶어서”, 생성자에서 획득하고 소멸자에서 자동으로 해제하는 방식이다.

예를 들면:
new 했으면 delete 해야 함 (메모리)
fopen 했으면 fclose 해야 함 (파일)

## 왜 필요한가
함수 중간에 return 하거나 예외가 터지면, 수동 해제 코드를 놓쳐 누수/미해제가 발생하기 쉽다.

특히 OS 자원을 지속적으로 다루는 엔드포인트 에이전트에서는 자원 해제를 수동으로 관리하면, 예외/조기반환/실패 분기에서 정리 누락이 발생해 누수·교착·성능 저하로 이어질 수 있다.
RAII는 자원 획득/해제를 객체 수명에 묶어 스코프 종료 시 자동 정리되게 하므로, 에이전트의 안정성과 예외 안전성을 높인다.

## 핵심 규칙 3개
- 생성자에서 자원을 얻고, 소멸자에서 반드시 해제한다.
- 스코프를 벗어나면 소멸자가 호출되어 정리가 자동으로 된다.
- 자원을 가진 객체는 복사하면 사고가 나기 쉬우니(중복 해제) 설계를 조심한다.

## 짧은 예시
#include <iostream>

using namespace std;

struct IntArrayGuard {
    int* p;

    explicit IntArrayGuard(std::size_t n) : p(new int[n]) {
        cout << "할당 완료"<<endl;
    }

    ~IntArrayGuard() {
        delete[] p;
        cout << "자동 해제" << endl;
    }

    // 복사 금지(중복 해제 방지)
    IntArrayGuard(const IntArrayGuard&) = delete;
    IntArrayGuard& operator=(const IntArrayGuard&) = delete;
};

int main() {
    IntArrayGuard arr(5);
    arr.p[0] = 42;

    cout << "중간에 return 해도..." << endl;
    return 0; // 여기서 arr 소멸자 호출 → 자동 해제
}
