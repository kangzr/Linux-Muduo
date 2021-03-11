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

Singleton* Singleton::m_instance_ptr = nullptr;  // 这里要定义一下，不然会报错

int main() {
	Singleton* instance = Singleton::get_instance();
	Singleton* instance_2 = Singleton::get_instance();
	system("pause");
	return 0;
}

```

```cpp
#include <iostream>
#include <memory>
#include <mutex>
// version 2:
// with problems below fixed:
// 1. thread is safe now：lock+双检锁
// 2. memory doesn't leak: shared_ptr管理
// 缺点：有些平台双检锁可能会失败；代码量增多；要求用户也得使用智能指针


class Singleton
{
public:
	typedef std::shared_ptr<Singleton> Ptr;
	~Singleton() {
		std::cout << "destructor" << std::endl;
	}
	static Ptr get_instance()
	{
		if (m_pInstance == NULL)
		{
			std::lock_guard<std::mutex> lk(m_mutex);
			if(m_pInstance == NULL)
				m_pInstance = std::shared_ptr<Singleton>(new Singleton);
		}
		return m_pInstance;
	}

private:
	Singleton()
	{
		std::cout << "constructor" << std::endl;
	}
	Singleton(Singleton&) = delete;
	Singleton& operator=(Singleton&) = delete;
	static Ptr m_pInstance;
	static std::mutex m_mutex;
};

Singleton::Ptr Singleton::m_pInstance = NULL;
std::mutex Singleton::m_mutex;

int main() {
	Singleton::Ptr s1 = Singleton::get_instance();
	Singleton::Ptr s2 = Singleton::get_instance();
	system("pause");
	return 0;
}

```

```cpp
#include <iostream>
#include <memory>
#include <mutex>
// version 3:
// magit static
// If control enters the declaration concurrently while the variable is being 
// initialized, the concurrent execution shall wait for completion of the initialization.
// 如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。
// 1. thread is safe now


class Singleton
{
public:
	~Singleton() {
		std::cout << "destructor" << std::endl;
	}
    // 返回引用？防止用户delete指针实例，或者将指针设置为null，而引用是不可以的
    // 引用返回的实例生存期由非用户代码管理，而通过指针返回实例，
	static Singleton& get_instance()
	{
		static Singleton m_pInstance;
        // static 实现单例，且只有用到时才创建，提高程序的启动速度。
		return m_pInstance;
	}

    Singleton(Singleton&) = delete;
	Singleton& operator=(Singleton&) = delete;
    // Note: Scott Meyers mentions in his Effective Modern
    //       C++ book, that deleted functions should generally
    //       be public as it results in better error messages
    //       due to the compilers behavior to check accessibility
    //       before deleted status
private:
	Singleton()
	{
		std::cout << "constructor" << std::endl;
	}
    // 由于Singleton限制其类型实例有且只能有一个，private保护构造函数，防止用户创建，
    // 通过m_pInstance实现单例

};

int main() {
	Singleton& s1 = Singleton::get_instance();
	Singleton& s2 = Singleton::get_instance();
	system("pause");
	return 0;
}

```

#### boost::noncopyable实现

```cpp
class noncopyable
{
protected:
	// 默认构造函数和析构函数都是protected
	// 不允许创建noncopyable实例，但允许子类创建
	// (即允许派生类构造和析构)
	noncopyable() = default;
	~noncopyable() = default;
private:
	// 使用delete关键字禁止编译器自动产生copy构造函数和赋值操作符
	noncopyable(noncopyable&) = delete;
	noncopyable& operator=(const noncopyable&) = delete;
};
```

