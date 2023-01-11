#类

	ctor: 构造函数
	dtor: 析构函数
	
	explict: 常用于构造函数前, 说明该构造函数只能
	显式调用, 让该构造函数不能隐式调用和复制初始化。
	
	public: 只有当前类能访问
	protected: 只有当前类和子类能访问
	public: 所有类都能访问
	
	classname(): 临时对象
	
	类中能加const则要加const
	
	类的构造函数要使用初始化列表
	
	如果类中有指针则必须要带有：
	classname( classname& a): 拷贝构造函数
	classname& operator=( classname& a):重载赋值操作符
	这两个函数和析构函数被称为Big Three
	
	如果类中有指针，默认的拷贝构造函数是浅拷贝，
	会造成内存泄露和指针指向同一位置
	
	classname c1;
	c1.memberFunction();	
		//等价于：classname::memberFunction( &c1);
		//c1相当于this指针, 如果函数为static函数则没有这一步
	
	static成员函数没有this指针, 只能处理static数据
	
	调用static函数可以用object调用, 
	也可以通过classname调用
	
	static数据要在类外做一次定义(给不给初值都可以),
	类内只是声明

#### 1. 拷贝赋值的重载
	需要检测自我赋值, 好处：
		1.速度快, 效率高, 防止不必要的动作
		2.防止a=a的出错, 下例的高亮行如果没有自我赋值检测, 则a=a会出错
```cpp{.line-numbers highlight=6-8}		
//eg:
classname& operator=( classname& a){	//因为可能连续赋值, 所以返回类型不能为void
	if( this == &a)
		return *this;

	/*1*/delete[] this->data;				
	/*2*/this->data = new Type[ a->size()];	
	/*3*/Copy( this->data, a->data);			
	return *this;
}
```
	

#### 2. new / delete
	new时先分配memory, 再调用构造函数
	delete时先调用析构函数, 再释放memory

	如果new了带有指针的类的数组, delete时没有[], 则只会唤起一次dtor
	使用[]后编译器会知道要调用多少次dtor(在cookie后4字节会说明需要调用多少次)
	但如果类中不带指针, 则delete时不使用[]也不会造成内存泄露(为了一致性, 建议也写[])
	
#### 3. 继承(inheritance): A is-a B (A是一种B)
	父类(base)数据可以被子类(derived)完全继承
	函数的继承是继承调用权(子类能调用父类的函数)
	
	可以通过子类对象调用父类函数。
	
	Template Method:
		写好框架, 让子类具体实现函数

	base的析构函数必须是virtual, 否则出现
	undefined behavior
	
	构造由内而外
		derived的构造函数首先调用base的default
		构造函数，然后再执行自己。
	析构由外而内
		derived的析构函数首先执行自己，然后再调用
		base的析构函数

```Cpp{.line-numbers}
//eg:
struct _List_node_base{
	_List_node_base* _M_next;
	_List_node_base* _M_prev;
};
template<typename _Tp>
struct _List_node 
	: public _List_node_base{
		_Tp _M_data;
};
```
	non-virtual function
		不希望derived class重新定义(override)它
	virtual function
		希望derived class重新定义(override)它, 
		且你对它已经有默认定义
	pure virtual function
		希望derived class一定要重新定义(override)
		它, 且你对它没有默认定义
			
			
#### 4. 复合(composition): A has-a B (A拥有B)

```Cpp{.line-numbers}
//eg:
class queue{
	.....
	protected:
		deque<T> c;
	public:
		bool empty() const {
			return c.empty();
		}
		size_type size() const {
			return c.size();
		}
		.....
}
```
	构造由内而外
		container的构造函数先调用component的
		default构造函数，然后在执行自己的。
	析构由外而内
		container的析构函数先执行自己，然后再
		调用component的析构函数。
		
#### 5. 委托(delegation):	composition by reference 
	两者用指针相连, 两者生命不同步
```Cpp{.line-numbers}
//eg:
class String{
	public:
		.....
	private:
		StringRep* rep;	//rep指向的东西可以随着需求的改变而改变
}
```
#### 6. Inheritance+Composition

1.			
		base 
		|
		|
		derived-----composition

		构造函数调用次序：
			先base, 再composition, 最后derived
		析构函数调用次序：
			先derived, 再composition, 最后base
2. 
		base-----composition
		|
		|
		derived

		构造函数调用次序：
			先composition, 后base, 最后derived
		析构函数调用次序：
			先derived, 后base, 最后composition
	
#### 7. Delegation+Inheritance

```Cpp{.line-numbers}
class A{
	vector<B*> b;
}
class B{
	.....
}
//B类可被继承, A中存数据
```
#### 8. pointer-like classes: 智能指针
```cpp{.line-numbers}
template< class T>
class shared_ptr{
public:
	T& operator*() const{
		return *px;
	}
	T* operator->() const{	//因为->操作符可以一直作用下去, 所以调用
				//返回后还是为px->xxx而非没有->
		return px;
	}
	shared_ptr(T* p): px(p) { }
private:
	T* px;
	long* pn;
.....
};
```

#### 9. function-like classes: 仿函数
```cpp{.line-numbers}
template < class T>
struct identity {
	const T& operator()(const T& x) const {
		return x;
	}
};
template< class Pair>
struct select1st {
	const typename Pair::first_type& operator()(const Pair& x)const{
		return x.first;
	}
};
template<class Pair>
struct select2nd {
	const typename Pair::second_type& operator() (const Pair& x) const{
		return x.second;
	}
};
```
# 函数

	函数参数要用reference传递	
	函数返回值为函数中创造的本地变量、本地对象，则不能return by reference
	
#### 1.成员函数
	每一个成员函数都默认带一个隐藏的this参数
	(可能是第一个参数也可能是最后一个)
		
#### 2.conversion function: 转换函数
	operator Type() const{	//没有返回类型，编译器默认返回类型为Type
		.....
	}
	一个类可以有多个转换函数, Type类型也不唯一,只要是之前出现过的Type就行
		
# reference(实质上就是指针)

	使用reference传递参数，传递者无需知道接收者是
	以reference形式接受
```Cpp{.class1 .class .line-numbers}
//eg1:
Type& function( Type* t ){
	.....
	return *t;	//返回指针指向的内容，不报错
}
```
```Cpp{.class2 .class .line-numbers}
//eg2:
void function( Type&t ){
	.....
}
Type t;
function( t);	//使用value作为参数传递，不报错
```
	两个例子不报错的原因都是因为: 传递者(使用者)无需关注
	参数是否以reference的形式实现传递
	
#操作符重载
	
	带有=的重载考虑对象被连续赋值, 要reference
	:: 、 . 、 .* 、 ? : 四个不能被重载
	
#### 1. 对于<<等特殊操作符只能重载为非成员函数
```Cpp{ .line-numbers}	
//eg:
ostream& operator << ( ostream& os, const Type& t){
	return os << .....;
}
````
####2. 对于自增自减操作符重载
	成员函数的写法：
```cpp{.line-numbers}
//前置 ++
classname & classname::operator++(){
	n ++;
	return * this;
}
//后置 ++
classname classname::operator++( int k){ 
	classname tmp( *this);//记录修改前的对象
	n++;
	return tmp; //返回修改前的对象
}
//后置的++不能连续++两次, 即(i++)++错误
//前置的++可以连续++两次, 即++(++i)正确
```		
	非成员函数写法：
```cpp{.line-numbers}
//前置--
classname& classname--( classname& d){
	d.n--;
	return d;    
}
//后置--
classname classname--( classname& d,int){
	classname tmp(d);
	d.n --;
	return tmp;
}
```

# template
	class, function, member三大模板
	
	编译器会对function template做实参推倒
	class template要说明使用的template类型
	泛化、specialization(特化)、partial specialization(偏特化)
	
#### 1. partial specialization:
1. 个数的偏特化
```cpp{.line-numbers}
template<typename T, typename Alloc=.....>
class vector{
	.....
};
template<typename Alloc=.....>
class vector<bool, Alloc> {
	.....
};
```
2. 范围的偏特化
```cpp{.line-numbers}
template< typename T>
class C{	//c1
	.....
};
template <typename T>
class C<T*>{	//c2
	.....
};
C<int> obj1;	//使用c1
C<int> obj2;	//使用c2
```
#### 2. template template parameter, 模板模板参数
```cpp{.line-numbers}
template<typename T, template <typename T> class Container>
class XCls{
	private:
		Container<T> c;
	public:
		.....
};

template<typename T>
using Lst = list<T, allocator<T>>;

XCls<string, list> mylst1;	//error
XCls<string, Lst> mylst2;
```
	而下面的就不是template template parameter:
```cpp{.line-numbers}
template <class T, class Sequence = deque<T>>
class stack{
	.....
	protected:
		Sequence c;	//底层容器
};
stack<int>s1;
stack<int, list<int>> s2;
```
#### 3. variadic templates(数量不定的模板参数):
```cpp{.line-numbers}
eg:
void print(){ }
template< typename T, typename... Types>
void print( const T& firstArg, const Types&... args){	
//传入的参数至少是1个, 所以要重载一个参数为0的print
	cout << firstArg << endl;
	print( args...);
}
//可以通过sizeof...(args)可以知道包里面还有多少
```

	
#杂记
	heap，或叫system heap, 是操作系统提供的一块global内存空间，程序可
	动态分配从中获得若干blocks。从heap中获得的空间必须要手动的释放。
	
	stack, 是存在于某作用域的一块内存空间，例如当调用函数, 
	函数本身会形成一个stack用来存放放置它所接受的参数, 以及返回地址。

```cpp{.line-numbers}
Type c4;	//c4为global object，其生命在整个程序结束时才结束, 作用域是整个程序

Type function( Type& c){
	Type c1;	//c1被称为auto object, 其生命在作用域结束时结束。
			//自动调用析构函数。
	Type* c2 = new Type;
		//不能忘记delete, 否则造成内存泄露, 当作用域结束时c2
		//所指向的heap object仍然存在, 但指针c2的生命却结束了, 
		//作用域之外再也看不到c2, 也就无法delete c2
	static Type c3;	
		//c3被称为static object,其生命在作用域结束后仍存在，
		//直到整个程序结束。		
	.....
}
```
```cpp{.line-numbers}
struct A{
	typedef int ii;
};
A a;
sizeof(a); //理论是0, 结果是1 
```
```cpp{.line-numbers}
for( decl : coll){	//ranged-base for, 尽量传引用
	.....
}
```