# Design Patterns
	If you have "this" kind of problem, then "this" technique might be useful
## Singleton Pattern:
		We have a class C, we want to enforce that only one instance of C is ever created
		EX: Finances
			wallet - only one, a singleton
			expense - all need access to the wallet
			Wallet *myWallet = Wallet::getInstance(); // static member function
	In <cstdlib> we have a function atexit, it takes a pointer to a function which has not arguments and void return type
		- multiple calls to atexit will stack up the functions LIFO
		Since the wallet ctor is accessible, anyone can create a wallet; that breaks the singleton Pattern

## Encapsulation:
		Treat objects like black boxes/capsules
			- hide implementation details
			- client code has access to an exposed interface

```cpp
		struct Vec{ //default visibility: public
			Vec();
		private:
			int x,y;
		public:
			Vec operator+(const Vec& v1) {...}
		};
		Vec v1(1,2);
		cout << v1.x << v1.y; // won't compile
```
		Advice: at least make all fields private

		C++ "class" keyword: default visibility is private
			class Vec{
				int x,y;
			public:
				Vec();
				Vec operator+();
			};
		Making fields private:
			+ maintain class invarients e.g: x,y must be positive
			+ flexibility to change implementation e.g: implement as polar coordinates
			class Vec{
				int x,y;
			public:
				Vec();
				Vec operator+();
				int getX() const{return x;}
				int getY() const {return y;} //accessor(getter)
				void setX(int v) {x = v;}
				void setY(int v) {y = v;} // mutators(setters)
			};

		Suppose - private fields
				- no accessors
				- wnat an output operator
		C++ provides the "friend" keyword
			class Vec{
				int x,y;
			public:
				Vec();
				Vec operator+();
				friend std::ostream& operator<<(ostream&,const Vec&);
			};
			ostream& operator<<(ostream& out, const Vec& v) {
				// can use private fields in Vec
			}

	System modeling
		- identify the abstractions/entities(classes)
		- relationship between entities/classes
		UML: unified modeling language
			- a class in UML is a box with 3 sections:
				| classname|
				------------
				| fields   | // optional
				------------
				| methods  | // optional
				------------
			Ex:
				| Vec      
				------------
				| -x:integer
				| -y:integer   
				------------
				| +getX():integer  // + for public, - for private
				------------

# Relationship between classes:
## Composition:
			embed an object within another object
			1 provide a 0 parametre ctor
			2 pre-empt the call to the 0 parametre ctors of Vec by calling some other ctor
			class Plane {
				Vec v1,v2; // call default 0 parameter ctor of Vec but there's no 0 parametre ctor
			};
			class vec {
				int x,y,z;
			public:
				Vec(int, int ,int);
			};
			Plane::Plane():v1(1,2,3),v2(4,5,6){}
			Plane p; // now compile
			The composition relationship creates a "owns a" relationship
				A owns B if 
					- if B cannot exist on its own
					- if A is destroyed, B is destroyed
					- if A is copied, B is copied
## Aggregation:
		parts in a catalog
			- if when A is destoryed, B continues to live
			- if A is copied, B is not copied
			class Pond {
				Duck* ducks[MAXDUCKS]; //ducks in a pond
			};

## Inheritance:
		Consider collection of any type of book
			- array of void*
			- union type
		Inheritance: creats "is a" relationship
			Textbook is a book with an additional topic
			Comis is a book with an additional protag
		Derived class(sub class, child class) inhrit EVERYTHING from base class
			int main() {
				Textbook tb = ...;
				tb.getAuthor();
				tb.author = ...; // cannot change since field is private
			}
			void Textbook::addAuthor(string auth) {author = auth;} //won't compile
			Textbook has inherited private author
			Privates really means private!!!
			Textbook::Textbook(string t,string a, string n,string topic) : title(t),author(a),... {} // still won't compile
			WHY?

			When an object is created 
				1 space is allocated
				2 super class part of the object is created
				3 field initialization in declaration order
				4 ctor body runs
			*** dtor uses the opposite order of creation

			Textbook::Textbook(string t,string a,int n, string topic) : Book(t,a,n),topic(topic) {} // now works

			protected: only code in this is accessible by code in class of subclasses
				EX: if author is protected
					tb.author = ...; //won't compile
					Textbook::addAuthor(...); // compile
					class Book{
					protected:
						string title,author;
						int numPages;
					};
					class TextBook:public Book {
						string topic;
					public:
						void addAuthor(string auth) {author += auth;}
					};
			Advice: keep fields private; provide protected accessors/mutators

			Ex: want a different criteria for determining when a book is heavy
				Textbook and comis will overwrite the inherted isHeavy methods
				class Book{
					int numPages;
				protected:
					int getNumPages() {return numPages;}
				public:
					bool isHeavy() {return numPages > 200;}
				};
				class Textbook:public Book{
				public:
					bool isHeavy() {return getNumPages() > 400;}
				};
				class Comis:public Book{
				public:
					bool isHeavy() {return getNumPages() > 30;}
				};
				in main.cc
				Book b(....,50);
				cout << b.isHeavy(); // call Book::isHeavy => false
				Comic c(...,40,...);
				cout << c.isHeavy(); // call Comic::isHeavy => true
				Book b2 = Comic(....,40,...);
				b2.isHeavy(); // Book::isHeavy is called
				Comic c(.....,40,..);
				Comic *cp = &c;
				cp->isHeavy(); // true Comic::isHeavy
				Book *bp = &c; // no slicing, comic is a book
				bp->isHeavy(); // false Book::isHeavy

			The compiler used the declared type of the pointer to statically determine which method to call
			If you do not want this default behaviour, make the method "virtual"
				- prefix isHeavy in class Book as virtual
			A method that is virtual is dynamically dipatched 
			The actual method to call is determined at runtime type of the object
			Virtual has a small cost because the decision to choose a method is made while the program is running

				Book* collection[100];
				for(int i = 0; i < 100; ++i) {
					cout << collection[i]->getTitle();
					cout << collection[i]->isHeavy();
					// Polymorphic array
				}
## Polymorphism: ability to accommdate multiple types under one abstraction
			When dtor for subclasses runs, the dtor of super class automatically called

		Pure virtual/abstract/concerte
		  Student
		  #numCourses:integer
		  virtual +fees():integer // do not implement this, make this pure virtual
		  ---------------------------------
		                   ^
		                   |
		         --------------------
		         |					|
		    Regular					Coop
		    +fees():interger		+fees():integer
		  //there is no student who's just a student(not regular nor coop)
		   class Student{
		   public:
		   		virtual int fees() = 0;
		   }; // pure virtual method has no implementation(for now)
		   a class has even one pure virtual method is abstract
		   <=> cnanot instantiate objects of an abstract class 
		   		Student s;
		   		s.fees(); // won't compile
		   		//Polymorphism
		   		Student *myclass[100];
		   		myclass[i]->fees();
		a subclass does inherit every pure virtual method
		a class which has no pure virtual methods is "concrete"
		A class is considered to be abstract if it has at least one pure virtual method, and have as many as common methods and virtual methods as you want.
		A abstact class should also have its constructor, since it will be called when a subclass object is created.

		In UML, abstract class name, pure virtual methods, and virtual methods are in *italics*.

	Rule of Thumb: if a class needs to be abstract, but there is no clear pure virtual method, **make the destructor pure virtual**.

	Another Rule: a class must always have a destructor.

	Pure Virtual: can have an implementation, but the subclasses must implement the pure virtual method to be concret.

## Observer Pattern:
	Subject                                   Observer
	- generates data                         - interested in the data
	- provide a subscribe method             - provide a notify method

## Decorator Pattern:
	Decorator is a component.
	Decorator has a (or, owns a) component.

## Factory Pattern:
	Factory pattern often works well with the singleton pattern.
	Ex:
		Sort
		--------
		+sort()
		-input()
		-output()
		virtual -doSort()
		---------------------------
		           ^
		           |
		-------------------
		|                 |
		----------		-----------
		Merge    |		|Bubble   |
		---------|      |---------|
		-doSort()|      |-doSort()| 
		----------      -----------
		Sort *s = new Merge;
		s->sort();
		void sort() {
			input();
			doSort();
			output();
		}

		Ex2:
			   Enemy
			     ^
			     |
		-----------------
		|				|
		Turtle         Bullet
		  ^
		  |
	-------------
	|			|
	Red        Green
			class Turtle:public Enemy{
			public:
				void draw() {
					drawHead();
					drawShell();
					drawTail();
				}
			private:
				void drawTail();
				virtual void drawShell() = 0;
				void drawHead();
			};
			class Red: public Turtle {
				void drawShell() {
					// draw Red shell
				}
			};

# C++ Templates:
	Template class: generalizing a class by parameterizing on one or more types
			class Node {
				int data;
				Node* next;
			};
			template<typename T>class Node {
				T data;
				Node<T>* next;
			public:
				Node(T data,Node<T>* next):data(data),next(next);
				T getData() const {return data;}
				void setNext(Node<T>* n) {next = n;}
			};
			Node<int>* intList = new Node<int>(1,new Node<int>(2,NULL));
			Node<char>* charList = new Node<char>('a',Node<char>('b',NULL));
			Node<Node<int> >* listOFlists;

# Standard Template Library(STL):
	Dynamic resizing array
	std::vector
		#include <vector>
		using namespace std;
		in main() {
			Vector<int> v;
			v.push_back(1); // add to the end 
			v.pop_back(); // drop the last
			for(int i = 0; i < v.size(); ++i) {
				cout << v[i] << endl; // no range checking
				cout << v.at(i) << endl; // range checked; if i not [0,size], the program terminates
			}

		Cool cs136 way of iterating over an array
			for(int *p = arr; p < arr+size; ++p) {
				cout << *p;
			}
		Super Cool cs246 Way !!!!!
			for(Vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
				cout << *it ;
			}
		Iterator is an abstration for a pointer.
			for(Vector<int>::reverse_iterator it = v.rbegin;it != v.rend(); ++it) {
				cout << *it;
			}   
			              -------------------------
			|  |          |  |  |  |  |           |  |  |  |
			   ^          ^           ^           ^
			   |		  |			  |			  |
			   v.rend()   v.begin()   v.rbegin()  v.end()
	std::map
		key -> value
		map<string,int> myMap;
		myMap["abc"] = 5;
		myMap["abc"] = 10;
		cout << myMap["abc"] << endl;
		myMap.count("abc"); // 0 is not found; 1 if found
		for(map<string,int>::interator it = myMap.begin(); it != myMap.end();++it) {
			(*it).first; // key
			(*it).second; // value
		}
		v.erase(v.begin()); // pop out first pair
		v.erase(v.end()-1);
		v.erase(v.rbegin()); // both pop out the last

Visitor Pattern:
	Visitor pattern gives us the ability to do double dispatch by combining overriding and overloading 
			- choose which method to run based on the runtime type of 2 pointers
		Ex:
		Enemy                   Weapon
		 ^						  ^
		 |						  |
	-----------             --------------
	|		  |				|            |
	Turtle   Bullet			Stick		Rock
			Player* p = ..;
			while(p->notDead()) {
				Enemy* e = ...;
				Weapon *w = p->chooseWeapon();
				//strike enemy with chosen weapon
				e->strike(&w);
			}
			class Enemy{
			public:
				virtual void strike(Weapon& w) = 0;
			};
			class Turtle:public Enemy{
			public:
				void strike(Weapon& w) { w.useOn(*this);} // w determined in runtime, *this since we do not have dynamically dispatch
			};
			class Bullet:public Enemy{
			public:
				void strike(Weapon& w) {w.useOn(*this);}
			};
			class Weapon {
			public:
				virtual void useOn(Turtle& t) = 0;
				virtual void useOn(Bullet& b) = 0;
			};
			class Stick:public Weapon{
			public:
				irtual void useOn(Turtle& t); // stick is being used on a turtle
				virtual void useOn(Bullet& b); // stick is being used on a bullet
			};
		Ex2:
			Book
			--------
			+update()
			----------
				^
				|
	    -----------------
	    |				|
	   ----------      ---------
	   |Textbook       |Comic        
	   |---------      |--------
	   |+update()	   |+update()
	   ----------      ----------
	   		Book *collection[100];
	   		for(...)
	   				collection->update();
	   		- requires access to source code
	   		- cluters the classes(No seperations of concerns)
	   	   Prepping Book hierarchy to accept visitors
	   	   	class Book{
	   	   	public:
	   	   		virtual void accept(BookVisitor& v) {v.visit(*this);}
	   	   	};	
	   	   	class TextBook: public Book {
	   	   	public:
	   	   		virtual void accept(BookVisitor& v) {v.visit(*this);}	
	   	   	};
	   	   	class Comic: public Book {
	   	   	public:
	   	   		virtual void accept(BookVisitor& v) {v.visit(*this);}	
	   	   	};
	   	   	class BookVisitor {
	   	   	public:
	   	   		virtual void visit(Book& b) = 0;
	   	   		virtual void visit(TextBook& t) = 0;
	   	   		virtual void visit(Comic& c) = 0;
	   	   	};
	   	   	In use, write class inherit BookVisitor and implemente 3 methods

Lecture 21 compilation Dependencies, Pointers to Implementation+Bridge Pattern,Big Three

Compilation Dependencies:
	alist.h:
		#ifndef
		#define
		class Blist;
		class Alist {
			int data;
			Blist* next;
		};
	blist.h:
		#ifndef
		#define
		class Alist;
		class Blist { 
			char data;
			Alist* next;
		};

	a.h:
		class A {...};
	b.h:
		#include "a.h"
		class B: public A{

		};
	c.h:
		#include "a.h"
		class C {
			A myA;
		};
	d.h:
		class A;
		class D {
			A* myA; // same for reference
			void foo();
		};
	d.cc:
		#include "a.h"
		#include "d.h"
		void D::foo() {
			myA->print();
		}
	e.h:
		class A;
		class E {
			A foo(A a); // not a real function call
		};

Pointer to Implementation:
	window.h:
		class XWindow {
			Display *d;
			GC gc;
			...
		public:
			void drawRectangle();
		};
	client.cc
		#include "window.h"
	Any changes to window.h(even to private members) requires client.cc to recompile
	To prevent this: let us create a pointer to implementation

	windowimpl.h:
		struct XWindowImpl {
			Display* d;
			GC gc;
			....  // private in class XWindow, now all pulic
		};
	window.h:
		class XWindowImpl;
		class XWindow {
			XWindowImpl* pImpl;
		public:
			...
		};
	window.cc:
		#include "window.h"
		#include "windowimpl.h"
		void XWindow::drawRectangle() {
			pImpl->d->update(..);
		}
	client.cc:
		#include "window.h"
		// since window.h does not include windowimpl.h, window.h remains unchanged when windowimpl changes; so client.cc won't re-compile
	Generalization:
		Generalization of the pImpl idiom to accommdate multiple implementation
		=> Bridge Pattern
	 low coupling:
	 	- coding to an interface
	 	- not exposing fields(public)
	 	- no friends
	 high cohesion:
	 	- classes work together to achieve a task
	 	- MVC
	 Big Three presence of inheritance
	 	-Destructors : set base class dtor virtual

	 Ex:
	 	class Book {
	 		string title, author;
	 		int numPages;
	 	public:
	 		...
	 	};
	 	class TextBook:public Book {
	 		string topic;
	 	public:
	 		...
	 	};
	 	TextBook b1 =TextBook(...);
	 	TextBook b2 = b1; //TextBook b2(b1);
	 	TextBook::TextBook(const TextBook& other):Book(other),topic(other.topic) {} //default
	 	Book::Book(const Book& other): title(other.title),author(other.author),numPages(other.numPages) {} // default
	 	b1 = b2;
	 	TextBook& TextBook::operator=(const TextBook& other) {
	 		Book::operator=(other);
	 		topic = other.topic;
	 		return *this;
	 	}
	 	Book* pb1 = new TextBook("cs246","Nomair",200,"c++");
	 	Book* pb2 = new TextBook("cs136","Adam", 100, "c");
	 	*pb1 = *pb2;
	 	*pb1 => cs136,Adam, 100, c++
	 	Book::operator= called(since default operator= is not virtual)
	 	*****Partial Assignment Problem***
	 	Solution: make operator virtual

	 	class Book {
	 	public:
	 		virtual Book& operator=(const Book&);
	 	};
	 	class TextBook:pulic Book {
	 	public:
	 		virtual TextBook& operator=(const Book&); // parameter not dispatch
	 	};
	 	// also works on Comic
	 	*****Mixed Assignment Problem*****

Lecture 22 Big Three, Casting, Exceptions
	Advice: construct hierarchies where the base class is abstract
		EX:
			class AbstructBook {
				string title, author;
				int numPages;
			protected:
				AbstructBook& operator=(const AbstructBook&);
			};
			class NormalBook: public AbstructBook{
			public:
				NormalBook& operator=(const NormalBook& other) {
					AbstructBook::operator=(other);
					return *this;
				}
			};
			AbstructBook *pl = new NormalBook(...);
			AbstructBook *p2 = new NormalBook(...);
			*p1 = *p2;
			// won't compile since operator= is protected in AbstructBook

Casting:
	Node n;
	int* p = (int*)&n;
	1 static_cast
		int m = 9;
		int n = 2;
		cout << m/n ; //4
		cout << static_cast<double>(m)/n; // print 4.5
		Book* pb = ...;
		TextBook *ptb = static_cast<TextBook*>(pb);
		ptb->getTopic();
		Downcase: move from a superclass to a subclass
		Unchecked cast: "Trust me" said you to the compiler
		If you are wrong => undefined behaviour
	2 const_cast: to add/move const property
		void library(int *p);
		void myCode(const int* q) {
			library(q); // won't compile, type does not match
			library(const_cast<int*>(q));
		}
		If library changes *p, behaviour is undefined.
	3 reinterpret_cast
		Vec v; // 3d vector x,y,z
		Student* sp = reinterpret_cast<Student*>(&v);
		sp->grade(); // 0.4*1st + 0.2*2nd + 0.4*3rd depends on compiler
			- Dependent on low-level details of how objects are arranged in memory
			- Not portable
	4 dynamic_cast
		Book* collection[100];
		for(...) {
			Book* bp = collection[i];
			TextBook *tbp = dynamic_cast<TextBook*>(bp);
			if(tbp) { // really points to a textbook
				cout << tbp->getTopic();
			}
			else 
				cout << "Not a textbook";
		}
		Using dynamic_cast requires that hierarchy have at least one virtual method
		RTTI: Runtime Type Information
			void whatIsIt(Book* bp) {
				if(dynamic_cast<TextBook*>(bp))
					Textbook
				else if(dynamic_cast<Comic*>(bp))
					Comic
				else Book
			}
		****Highly coupled to Book hierarchy****

Lecture 23 Exceptions
	When an error occurs, an exception is throw/raised
		- if the exception is not handled, then the program terminates.
		- you can handle these exceptions using try/catch blocks
	Stack unwinding: the process of popping the call stack repeatedly until a handler for an exception is found
	Error recovery in multiple waysL
		try{...}
		catch(SomeExn& a) { // better use reference instead of value
			/*partial recovery
				1 throw SomeOtherExn(...);
				2 throw a; // throw the copy(including subtype of a)
				3 throw; // throw the original exception again(subclass slicing)
			*/
		}
	All exceptions from the std c++ library inherit from a class "exception"
	However, user defined exceptions need not inherit from "exception" 
		try{}
		catch(exception) {...}
	Special syntax to catch all exceptions
		try{}
		catch(...) {......}
	We can throw any value e.g: int, char, bool
	Exception handling is typically much slower than normal control flow
		class SomeExn {};

	Common C++ Exceptions:
		bad_cast: dynamic_cast to reference fails
		bad_alloc: new fails(no more heap space)
		out of range: index out of bounds(Vector::at(i))

	Ex:
		class BaseExn {};
		class DerivedExn:public BaseExn {};
		DerivedExn n = ...;
		BaseExn &b = n;
		try {throw b;}
		catch(DerivedExn) {}
		catch(BaseExn) {} // catch here
		catch(...) {} 
	Ex:
		Book* b1 = new TextBook(...);
		Book* b2 = new TextBook(...);
		*b1 = *b2;
	make Book::operator= virtual
		TextBook& TextBook::operator=(const Book& other) {
			TextBook& other = dynamic_cast<TextBook*>(other);
			Book::operator=(other);
			topic = other.topic;
			return *this;
		}
	Exception changes everthing:
		void foo() {
			Myclass* p = new Myclass;
			Myclass q;
			bar(); // leak memory for p if bar() throws exception
			delete p; 
		}
	Need to write exception safe code
		- exceptions should not cause memory leaks, dangling pointers, etc

C++ gurantee: during stack unwinding, all stack allocated memory is reclaimed
	(q space is reclaimed, dtor runs)
		void foo() {
			Myclass* p = new Myclass;
			Myclass q;
			try { bar();}
			catch(...) { // tedious and error prone!!!
				delete p;
				throw;
			}
			delete p;
		}


Lecture 24 Exception
	void f(){
		Mycost *p = ;
		Myclass g; 
		try{
			g();
	
		} catch(...) {
			delete p;
			throw;
		}
	}

never let the ctor throw an exception
	- dtors are called during stack unwinding

RAII: Resource Acquisition Is Initialization
	should be wrapped

c++03 provide the template class:auto-ptr
	- the ctor for auto-ptr expects a pointer to dynamic memory
	- the dtors for auto-ptr will call delete on underlying ptr 

void f() {
	auto_ptr<Myclass> p(new MyClass);
	MyClass q;
	g();
}

class C{.....};

auto_ptr<c> p(new C);
auto_ptr<c> q = p; //get double free
 p->.... XXXXX
 copy ctor/operator= actually steal the ptr to the underlying dynamic memory

c++11 calls auto_ptr;
	unique_ptr

tr1::shared_ptr
	uses reference count to track how many pointers are pointing to the same dynamic memory

levels if exception safty
	1 basic guarantee: an exception does not cause a function to leave the program in an invalid state(e.g no memory leak,no dangling ptr)
	2 strong grarantee: if an exception occurs, its as if the function is never called(program state is unchanged)
	3 no throw guarantee(highest) - function never fails, always achieve the task

		class A{};
		class B{};
		class C{
			A a;
			B b;
			void f() {
					a.g(); //both g and h give strong gurantee
					b.h();
			}
		};
		Assume g and h have local side effects
		void  C::f(){
			A tempa  = a;
			B tempb = b;
			tempa.g(); //only change tempa not a
			tempb.h(); 
			a = tempa;
			b = tempb;
		}
		//solution:: ptr
		struct CImpl{A a; B b;};
		class C{
			auto_ptr<CImpl> pImpl;
			void f(){
				//strong gurantee
				auto_ptr<CImpl> temp(new CImpl(*pIml));
				temp->a.g(); // changes only within a
				temp->b.h();
				//suceeded
				std::swap(pImpl,temp);
			}
		};

Function Call
	-compiler load the address of the code of the function
	-jump to that address
Non-virtual Methods
	-treated as function calls 
Virtual Methods(could lead different addresses in memory,no one know until runtime)
	-create vTable
	--------------------|
	"Book"
	---------
	isHeavy          ------------------>Book::isHeavy()
										(actual code somewhere in memory) 
	--------
	favourite        ------------------>Book:favourite()

	Book *b = new Book(...) 
	b ----->Vptr------>book Vtable(one VTable per class)
	  | = new TextBook
	  |------>vptr -----> textbook VTable

	Why dtor is not default virtual: extra Vtable needed
	no reference to Vtable(no virtual)
