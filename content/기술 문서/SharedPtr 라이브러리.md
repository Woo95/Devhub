# **SharedPtr 라이브러리**
---
## **개요**
이 라이브러리는 C++ 스마트 포인터의 내부 구조와 동작 방식을 더 자세히 이해할 수 있도록 사용자 정의 **shared pointer**를 구현한 것입니다. `std::shared_ptr`의 **참조 카운트**(reference counting) 방식을 모방하면서도, 명확성과 최소한의 오버헤드에 중점을 두었습니다. 구현은 **헤더 전용**(header-only)으로 되어 있으며, 더 이상 필요하지 않은 리소스를 자동으로 삭제함으로써 안전한 메모리 관리를 제공합니다. 

[**저장소 보기**](https://github.com/Woo95/SharedPtr)

---
## **클래스 목록 및 설명**
### `CRefCounter 클래스` 
- **역할**: 
	- 참조 카운트가 필요한 객체를 위한 기본 클래스입니다.
- **핵심 함수들**:
	- [[SharedPtr Library (Code)#^15063f|void IncrementRef()]]:
		- 참조 카운트를 증가시킵니다.
	- [[SharedPtr Library (Code)#^e68875|void DecrementRef()]]:
		- 참조 카운트를 감소시킵니다.
		- 참조 카운트가 0이 되면 리소스를 삭제합니다.
### `CSharedPtr<T> 클래스`
- **역할**: 
	- 참조 카운트를 사용해 동적으로 할당된 리소스의 수명을 관리하는 스마트 포인터입니다.
- **핵심 함수들**:
	- 이 함수들은 참조 카운트 관리와 포인터 할당을 직접적으로 처리하는 유일한 함수들입니다.
		- [[SharedPtr Library (Code)#^af2f1a|CSharedPtr(T* ptr)]]:
			- 생성자, 원시 포인터를 받아 참조 카운트를 증가시킵니다.
		- [[SharedPtr Library (Code)#^7ed3c8|CSharedPtr(const CSharedPtr\<T\>& other)]]:
			- 얕은 복사 생성자, 포인터를 공유하고 참조 카운트를 증가시킵니다.
		- [[SharedPtr Library (Code)#^19ba77|~CSharedPtr()]]:
			- 소멸자, 참조 카운트를 감소시키고 0이 되면 객체를 삭제합니다.
		- [[SharedPtr Library (Code)#^88b6a9|void operator = (T* ptr)]]:
			- 원시 포인터를 할당하며 참조 카운트를 적절히 갱신합니다.
		- [[SharedPtr Library (Code)#^9a7d66|void operator = (const CSharedPtr\<T\>& other)]]:
			- 다른 스마트 포인터를 할당하며 리소스를 공유하고 카운트를 조정합니다.

---
## **코드 분석**
### `사용 예제`
![[SharedPtr Library (Code)#^1d1de5]]
**참조 카운트**를 사용해 `CObject`의 **메모리 관리**를 수행하는 `CSharedPtr` 사용 예제입니다.