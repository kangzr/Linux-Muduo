#### C++单例实现

> You need to have one and only one object of a type in system

##### 2.1 要点

- 全局唯一特性，申明static，仅本源文件有效, 禁止用户自己声明并定义(将构造函数定义为private)
- 线程安全
- 禁止赋值和拷贝
- 用户通过接口get_instance获取实例



```cpp
#include <iostream>
// version1:
// with problems below:
// 1. thread is not safe 线程AB同时get_instance(), 会出来两个单例
// 2. memory leak 只有new 没有delete

class Singleton {
private:
	Singleton() {
		std::cout << "constructor called!" << std::endl;
	}
	Singleton(Singleton&) = delete;  //禁止赋值
	Singleton& operator=(const Singleton&) = delete;  //禁止拷贝
	static Singleton* m_instance_ptr;
public:
	~Singleton() {
		std::cout << "destructor called!" << std::endl;
	}
	static Singleton* get_instance() {
		if (m_instance_ptr == nullptr) {
			m_instance_ptr = new Singleton;
		}
		return m_instance_ptr;
	}
	void use() const { std::cout << "in use" << std::endl; }
};

Singleton* Singleton::m_instance_ptr = nullptr;

int main() {
	Singleton* instance = Singleton::get_instance();
	Singleton* instance_2 = Singleton::get_instance();
	system("pause");
	return 0;
}

```