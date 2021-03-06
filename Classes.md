# C++ Classes
## OOP: we can put functions inside a struct
```cpp
struct  Student {
	int assn, mt,final;
	float grade() { ... }
};
struct s = {.....}; // c-style initialization
s.grade(); //calling a function inside a struct 
```
	Class is a struct that can contain functions 
		***C++ has a class keyword
	An instance of a class is called an object
	A function that is inside a class is called a "member function" as "method"
		***you must call an method on an object
	The method has access to the fields of the object on which the method has called
	Difference between functions and methods:
		a method has a hidden parametre called "this"
		"this" : a pointer to the object on which the method was called

## Initializing Objects:
c-style initialization requires compile time constants
c++ allows special methods to construct objects ---- constructors(ctor)
Ex:
```cpp
struct Student {
	int assn, mt,final;
	Student(int assn, int mt, int final) {
		this->assn = assn;
		mt = mt;
		final = final;
	}
};
```
			heap: Student* bobby = new Student(60,80,70);
			stack: Student bobby = Student(70,0,0);
				OR Student bobby(..,..,..);
			*** Warning Student newGuy();
			stack: Student newGuy;
			heap: Studnet* newGuy = new Student;

		Every class comes with a default ctor(0 parametre ctor) which calls the default ctors on fields that are objects
				struct Myst {
					int x; // not intialized
					Student s; // already has ctor
					Student* ps; // not intialized
				};
		The default ctor goes away if you write any ctor
			Vec v; // no longer compiles(no 0 parameter ctor)
			struct Vec{
				int x,y;
				Vec(int x, int y){...}
			};
		If any ctor is implemented, you also lose c-style initialization
			Vec v = {1,2}; // no longer compiles
			Vec v(1,2); // right way

Ex:
```cpp
struct Mystr {
	const int a;
	int &myref; // both must be initialized
};
```
		c++ disallows field initializers
					  Initializing const and reference in ctor body
		WHERE???
		When an object is created:
			1 space is allocated(stack or heap)
			2 field initialization: default ctor for fields that are objects are called
			3 ctor body runs
		NOW LET'S HIJACK STEP 2
			special syntax: member initialization list(MIL)
				*** only availble inside constructors
			struct Mystr
			{
				const int n;
				int &myref;
				Mystr(int c, int& r):n(c),myref(r) {
					//step 3
				}
			};
			MIL is the only way to initialize a field const and references
			If a MIL does not initialize a field that is an object, the default ctors will be called
			Initialization always occurs in declaration order, inrespective of the orders in MIL

		Ex:
			Student bobby(..,..,..);
			Student billy = bobby; // Studnet billy(bobby);
		Copy ctor constructs a new object as a copy of an existing object
			*** you get a build-in copy ctor
			*** in C++ Every class has:
				- default ctor
				- copy ctor
				- dectructor
				- copy assignment operator
For a class C, the copy ctor takes one parametre, a reference to const of type C
```cpp
			struct Student
			{
				Student(const Student& other):assn(other.assn),mt(other.mt),final(other.final){}
				// build-in copy ctor
			};
```
Ex: 
```cpp
			struct Node
			{
				int data;
				Node* next;
				Node(int data,Node* n):data(data),next(n){} // default copy ctor
			};
			Node* np = new Node(...);
			Node m = *np; // copy ctor
			Node* npCopy = new Node(*np);
```
The default copy ctor does a shallow copy
Whenever dynamic memory is involved(the object is not continues), you most likely want a deep copy
```cpp
				Node(const Node& other) {
					data = other.data;
					if(other.next) {
						next = new Node(*other.next);
					}
					else next = NULL;
				}
				Node(const Node& other): data(other.data),next(other.next? new Node(*other.next):NULL) {}
```
			A copy ctor is called:
				1 when contructing an object as a copy of another
				2 when an object is passed by value
				3 when an object is returned by value
				*** reason for this rule: parametre is reference type

			Single parametre constructors:
				stuct Node{
					Node(int data): data(data),next(NULL){}
				};
			single parametre ctors create implicit conversions
			Node n(4);
			Node m = 4;
			*** string str = "hello"; std::string has a 1 parametre ctor, with type const char*

## Destructor:
when an object is detroyed, a special method called the Destructor runs
Stack allocated object is destroyed when it goes out of scope(stack popped out)
Heap allocated object is destroyed when it is deleted
All classes come with a dtor calls the dtors on fields that are Objects
	- each class has one dtor which takes 0 parametre
	- dtor name is the name of the class prefixed with ~
Ex:
```cpp
			Node* np = ...;
			delete np; // cause leak on further nodes
			~Node() {
				delete next;
			} // recursive call, calling delete on NULL is safe
```
	**** Node:: in the context of Node(inside .cc files)

## Assignment operator:
		Student bobby(billy); // call copy ctor
		Student jane; // call 0 parameter ctor
		jane = bobby; // => jane.operator=(bobby) assignment operator
		Assignment operator updates an existing object as a copy of another

		Student& operator=(const Student& other) 
		Node& Node::operator=(const Node& other) {
			data = other.data;
			next = other.next; // shallow copy, leaking memory
			return *this;
			// if assignment operator and copy ctor almost the same => wrong
		}
		We need deep assignment as below:
		Node& Node::operator=(const Node& other) {
			if(this == &other)
				return *this; // self assignment safe
			data = other.data;
			delete next; // do not automatically set to NULL
			next = other.next ? new Node(*other.next):NULL;
			// can fail when no memory for heap, will go somewhere else
			// NOT exception safe
			return *this; //cascading
		}
		Since not exception safe, revised as below:
		Node& Node::operator=(const Node& other) {
			if(this == &other)
				return *this; // self assignment safe
			Node* tmp = next;
			next = other.next ? new Node(*other.next):NULL;
			data = other.data; // if exception occurs, this node is not changed
			delet tmp;
			return *this; //cascading
		}
	A better way: Copy and Swap
		struct Node
		{
			void swap(Node& others) {
				int tdata = data;
				data = others.data;
				others.data = tdata;
				Node* t = next;
				next = others.next;
				others.next = t; 
			}
			Node& operator=(const Node& others){
				Node tmp = other; // copy ctor, deep copy
				swap(tmp); // this->swap(tmp)
				return *this;
			} // dtor is called for tmp(orginal this)

	Rule of 3:
		If you need to write a custom 
			- copy ctor or
			- dtor or
			- operator= (must be implemented as method)
		then usually you need to write all 3

	Operation overloading as method:
		When an operator is implemented as method, LHS is represneted by 'this'
		*** cannot change order
		Vec Vec::operator*(const Vec& rhs) {
			Vec v(x+rhs.x,y+rhs.y);
			return v;		
		}
		Vec Vec::operator*(int k) {
			Vec v(x*k,y*k);
			return v;		
		} // can have v*5 but 5*v not allowed => use functions 
		for input/output operator: use functions!
			v2 << (v1 << cout); //UGLY

# Lecture 15 Const, static, design patterns, singleton pattern, encapsulation
Const:
```cpp
	struct Student {
		mutable int numcalls;
		float grade() const { // promise will not change field 
			++numcalls;
		}
	};
		const Student billy(60,70,60);
		billy.grade(); //without mutable, const method, does not change but compiler won't allow
```
	
	const method does not change the fields
	const object can only call const methods
	a field that is mutable can always be changed

Static keyword:
	static field: associates the field with the class, not any object of the class
	Ex:
		struct Student
		{
			static int numObjects; //declaration
			Student() {
				++numObjects;
			}
		};
	a static field must be defined external to its type definition
		int Student::numObjects = 0;

	static member functions:
		- do not need a object to call a member function that is static
		 (static methods is common incorrect terminology)
		 Ex:
		 	static void printObjectCount() {
		 		cout << numObjects << endl;
		 	}
		 	int main() {
		 		Student s1(...);
		 		Student s2(...);
		 		Student::printObjectCount();
		 	}
		- static member functions do not have access to "this" pointer
		- static member functions can only access other static member functions and static fields


