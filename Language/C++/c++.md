### C++

----



#### 1. 기본 입출력 함수


* Cout (console out)

  * c언어 출력 함수인 printf() 함수와 흡사합니다.

  ~~~
  #include <iostream>
   
  int main()
  {
      std::cout << "Hello, world!" << std::endl;
   
      return 0;
  }
  ~~~

    * include <iostream> 은 Input/Output Stream의 약자로 c에서 include <stdio.h>와 같은 기능을 합니다.
    * <<는 연산자 오버로딩에 의한 것으로써 c에 시프트 기능이 아니라 데이터의 입출력을 위해 사용됩니다.
    * 마지막 endl 은 c에서의 \n 와 같은 기능을 합니다.



* cin (console input)

  * c언어 입력 함수인 scanf() 함수와 흡사합니다.

    ~~~
    #include <iostream>
     
    int main()
    {
        char name[10];
        int age;
     
        std::cin >> name >> age;
        std::cout << "당신의 이름은 " << name << "이며, 나이는 " << age << "살입니다." << std::endl;
     
        return 0;
    }
    
    
    출력결과:
    
    김철수 14
    당신의 이름은 김철수이며, 나이는 14살입니다.
    ~~~

  * 여기서 cout 함수를 사용할 때랑 다르게 >>을 사용하며 오른쪽에 위치한 변수에 저장합니다.



#### 2. 네임스페이스(namespace)

* 네임스페이스를 나누어 함수이름이 똑같아도 별도의 공간으로 분리되어 있기 때문에 충돌하지 않고 사용할수 있습니다.

* 형태 :

  ~~~
  namespace 이름
  {
     // 변수, 함수 등
  }
  ~~~

* 사용법:

  ~~~
  ..
  이름::함수명();
  이름::식별자 = 10;
  ~~~

* 예제

  ~~~
    #include <iostream>
     
    namespace A {
        void Add() {
            printf("A의 Add() 호출!\n");
        }
    }
     
    namespace B {
        void Add() {
            printf("B의 Add() 호출!\n");
        }
    }
     
    int main()
    {
        A::Add();
        B::Add();
        return 0;
    }
    
    
    출력결과:
    
    A의 Add() 호출!
    B의 Add() 호출!
  ~~~


* using
  : 네임스페이스를 쉽게 사용할 수 있도록 도와줍니다. 

  ~~~
  #include <iostream>
   
  namespace A {
      void Add() {
          printf("A의 Add() 호출!\n");
      }
      void Minus() {
          printf("A의 Minus() 호출!\n");
      }
  }
   
  using A::Add;
   
  int main()
  {
      Add();
      A::Minus();
      return 0;
  }
  ~~~



#### 3. 함수 오버로딩(Function Overloading)

* 함수 오버로딩이란 함수명은 같으며 인자의 자료형이나 수가 다른 함수의 선언을 허용하는 것을 말합니다.

  ~~~
  #include <iostream>
   
  using namespace std;
   
  void func(int a)
  {
      cout << "void func(int a)이 호출됨!" << endl;
  }
   
  void func(int a, int b)
  {
      cout << "void func(int a, int b)이 호출됨!" << endl;
  }
   
  int main()
  {
      func(4);
      func(5, 6);
   
      return 0;
  }
  ~~~

  * 바로 위의 예제에서 **using namespace std;** 를 포함시키면 cout, cin 함수를 편하게 사용할 수 있습니다.

* 함수오버로딩의 특징으로는

  * 함수명이 같아야 합니다.
  * 매개변수의 수가 다르거나, 매개변수 수가 같으면 자료형이 달라야 합니다.
  * 위 두조건을 만족한다면, 반환형의 차이는 함수 오버로딩에 영향을 미치지 않습니다.



#### 4. new, delete 

* C언어에서의 malloc 과 free를 대신하는 함수입니다.

  형식은

  ~~~
  new 자료형          // 단일객체에 동적할당
  new 자료형[길이]     // 객체의 배열을 동적할당
  
  예시)
  // studentScore = (int *) malloc(sizeof(int)*studentNum);
  studentScore = new int[studentNum];
  ~~~

  ~~~
  int* ptr1 = new int(3);
  double* ptr2 = new double[3];
  float* ptr3 = new float[10];
  ~~~




* delete 는 new 연산자로 할당된 메모리를 해제할때 사용합니다.

  형식은

  ~~~
  delete 포인터      //단일 동적할당 해제
  delete []포인터    //배열 동적할당 해제
  ~~~

|                malloc / free                 |             new / delete              |
| :------------------------------------------: | :-----------------------------------: |
|                     함수                     |                연산자                 |
|       초기값을 원하는 값으로 할당가능        |      할당과 초기화 한꺼번에 가능      |
|   필요한 크기는 바이트 단위로 지정해야 함    |     필요한 크기는 컴파일러가 계산     |
| 할당된 메모리는 realloc을 통해 크기변경 가능 |             재할당 불가능             |
|            할당 실패 시 NULL 반환            | 할당실패시 std::bad_alloc 예외를 던짐 |
|  객체 생성/제거 시 생성자/소멸자 호출 불가   | 객체 생성/제거 시 생성자/소멸자 호출  |



#### 5. 구조체

* C언어에서의 구조체

  ~~~
  #include <stdio.h>
   
  struct student {
    int id;
    char *name;
    float percentage;
  }; // 구조체 뒤에 세미콜론이 와야함
    
  int main() {
    struct student s={1, "김철수", 90.5};
    printf("아이디: %d \n", s.id);
    printf("이름: %s \n", s.name);
    printf("백분율: %f \n", s.percentage);
    return 0;
  }
  ~~~

  * 구조체 변수 선언시 struct 키워드를 생략하기 위해선 typedef 선을 따로 추가해야 합니다.

* C++에서의 구조체

  ~~~
  #include <iostream>
   
  using namespace std;
   
  struct student {
    int id;
    char *name;
    float percentage;
   
    void Show() {
        cout << "아이디: " << id << endl;
        cout << "이름: " << name << endl;
        cout << "백분율: " << percentage << endl;
    }
  }; // 구조체 뒤에 세미콜론이 와야함
    
  int main() {
      student s={1, "김철수", 90.5};
   
      s.Show();
    return 0;
  }
  ~~~

  * C++에서의 구조체는 구조체내에 함수 선언도 허용합니다.



#### 6. 접근 제어 지시자

* 함수를 구조체 내에 정의하거나, struct 키워드가 생략되는거 의외에도 멤버의 접근을 제한할 수 있습니다.
  기본적으로 pubilc멤버로 간주되며 종류는 pubilc, private, protected 가 있습니다.

  ~~~
  public: 어디서든 접근이 가능합니다.
  
  private: 외부에서 접근이 불가능합니다.
  
  protected: 외부에서 접근이 불가능하나, 상속된 파생 클래스에서는 접근이 허용된다.
  ~~~

* 예시

  ~~~
  #include <iostream>
   
  using namespace std;
   
  struct student {
  private:
    int id;
    char *name;
    float percentage;
  public:
    void Show();
    void SetInfo(int _id, char * _name, float _percentage);
  }; // 구조체 뒤에 세미콜론이 와야함
    
  void student::Show() {
    cout << "아이디: " << id << endl;
    cout << "이름: " << name << endl;
    cout << "백분율: " << percentage << endl;
  }
   
  void student::SetInfo(int _id, char * _name, float _percentage)
  {
      id = _id;
      name = _name;
      percentage = _percentage;
  }
   
  int main() {
      student s;
      //s.id = 1;
   
      s.SetInfo(1, "김철수", 90.5);
      s.Show();
    return 0;
  }
  ~~~
