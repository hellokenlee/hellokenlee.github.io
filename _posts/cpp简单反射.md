实现：

```cpp
#include <iostream>
#include <unordered_map>
#include <functional>

#define STR(s) #s

#define REGISTER(cls) \
	Factory::registerClass(STR(cls), []() {return new cls;}); \

using namespace std;

class Object {
public:
	virtual void print() {
		printf("This is Object.\n");
	}
};


class Factory {
public:
	static void registerClass(string className, function<Object*()> func) {
		creators.insert(make_pair(className, func));
	}

	static Object* createObject(const string className) {
		return creators[className]();
	}

private:
	Factory();
	static unordered_map<string, function<Object*()>> creators;
};

unordered_map<string, function<Object*()>> Factory::creators;

```


测试：

```cpp
class A : public Object {
public:
	virtual void print() {
		printf("This is A.\n");
	}
	virtual void printA() {
		printf("This is AA.\n");
	}
};

class B : public Object {
public:
	virtual void print() {
		printf("This is B.\n");
	}
};

class C : public A {
public:
	virtual void print() {
		printf("This is C.\n");
	}
	virtual void printA() {
		printf("This is CC.\n");
	}
};


int main()
{
	REGISTER(Object);
	REGISTER(A);
	REGISTER(B);
	REGISTER(C);
	Object* a = Factory::createObject("A");
	Object* b = Factory::createObject("B");
	A* c = dynamic_cast<A*>(Factory::createObject("C"));
	a->print();
	b->print();
	c->printA();

	return 0;
}
```