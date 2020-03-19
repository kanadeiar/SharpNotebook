# Основы ООП

## Классы

Объявление классов:
```cpp
class TRoad
{
public:
	float Length{};
	int Width{};
	TRoad();
	TRoad(float lenght0, int width0 = 2);
};
class TCar
{
public:
	float X, V;
	int P;
	TRoad * Road;
	void Move();
	TCar();
	TCar( TRoad * road0, int p0, float v0);
};
```
Конструкторы и методы классов:
```cpp
TRoad::TRoad()
{
	Length = 100;
	Width = 1;
}
TRoad::TRoad(float lenght0, int width0)
{
	if (lenght0>0)
		Length=lenght0;
	else Length=1;
	if (width0>0)
		Width=width0;
	else Width=1;
}
TCar::TCar()
{
	Road=NULL; P=0; V=0; X=0;
}
TCar::TCar(TRoad * road0, int p0, float v0)
{
	Road = road0;
	P=p0;
	V=v0;
	X=0;
}
void TCar::Move()
{
	X += V;
	if (X>Road->Length)
		X=0;
}
```
Исползование класса:
```cpp
	TCar car1;
	printf("Data car: %.1f %.1f %d\n", car1.X, car1.V, car1.P);
	TRoad road(60, 3);
	const int N=3;
	TCar * cars[N];
	for (int i=0; i<N; i++)
	{
		cars[i] = new TCar(&road, i+1, 2.0*(i+1));
	}
	do
	{
		for (int i=0; i<N; i++)
			cars[i]->Move();
	}
	while (!_kbhit());
```

## Интерфейсы классов

Открытые члены класса для доступа к закрытым членам класса.

Пример:
```cpp
class TPen
{
private:
	int FColor;
	int val;
public:
	string getColor();
	void setColor(string newColor);
	int getVal() {return val;}
};
//реализация методов доступа
string TPen::getColor() {
	stringstream s;
	s << setfill('0') << setw(6) << hex << FColor;
	return s.str();
}
void TPen::setColor(string newColor)
{
	stringstream s;
	if (newColor.length()!=6)
		FColor=0;
	else
	{
		s << newColor;
		s >> hex >> FColor;
	}
}
```
Использование:
```cpp
	TPen pen;
	pen.setColor("FFFF00");
	cout << "Цвет пера: " << pen.getColor();
```

## Наследование

Абстрактный класс - родительский:
```cpp
class TLogElement //абстрактный метод(=0)
{
private:
	bool FIn1, FIn2;
protected:
	bool FRes;
	virtual void calc()=0; //виртуальный абстрактный(=0) метод
	bool getIn2() {return FIn2;}
	void setIn2(bool newIn2);
public:
	bool getIn1() {return FIn1;}
	void setIn1(bool newIn1);
	bool getRes() {return FRes;}
};
//реализация:
void TLogElement::setIn2(bool newIn2)
{
	FIn2=newIn2;
	calc();
}
void TLogElement::setIn1(bool newIn1)
{
	FIn1=newIn1;
	calc();
}
```
Класс-наследник этого класса:
```cpp
class TNot: public TLogElement
{
protected:
	void calc(); //переопределение
};
//реализация:
void TNot::calc()
{
	FRes=!getIn1();
}
```
Переопределение базового родительского класса:
```cpp
class TLog2In: public TLogElement
{
public:
	TLogElement::setIn2; //открытие защищенного
	TLogElement::getIn2; //открытие защищенного
};
```
Определение классов-наследников этого класса:
```cpp
class TAnd: public TLog2In
{
protected:
	void calc(); //переопределение
};
class TOr: public TLog2In
{
protected:
	void calc(); //переопределение
};
//реализация:
void TAnd::calc()
{
	FRes=getIn1()&&getIn2();
}
void TOr::calc()
{
	FRes=getIn1()||getIn2();
}
```
Использование этих классов:
```cpp
	TNot elNot;
	TAnd elAnd;
	int A, B;
	cout << "A B !(A&B)" << endl;
	cout << "----------" << endl;
	for (A=0; A<=1; A++)
	{
		elAnd.setIn1(A);
		for (B=0; B<=1; B++)
		{
			elAnd.setIn2(B);
			elNot.setIn1(elAnd.getRes());
			cout << " " << A << " " << B << "    " << elNot.getRes() << endl;
		}
	}
```

## Сообщения между классами

Определение базовых классов, чтобы элементы могли передавать друг другу сообщения об изменении своего выхода. 

Пример - выход любого логического элемента может быть подключен к любому входу другого логического элемента. Пример абстрактного класса:
```cpp
class TLogElement //абстрактный метод(=0)
{
private:
	bool FIn1, FIn2;
	//связь с другим классом
	TLogElement * FNextEl; //ссылка на следующий логический элемент
	int FNextIn; //номер входа этого следующего элемента, к которому подключен выход данного элемента
protected:
	bool FRes;
	virtual void calc()=0; //виртуальный абстрактный(=0) метод
	bool getIn2() {return FIn2;} //защищенное
	void setIn2(bool newIn2); //защищенное
public:
	bool getIn1() {return FIn1;}
	void setIn1(bool newIn1);
	bool getRes() {return FRes;}
	//связь с другим классом
	void Link(TLogElement *nextElement, int nextIn=1);
};
//реализация связь с другим классом
void TLogElement::Link(TLogElement* nextElement, int nextIn)
{
	FNextEl=nextElement;
	FNextIn=nextIn;
}
void TLogElement::setIn1(bool newIn1)
{
	FIn1=newIn1;
	calc();
	if (FNextEl) //если следующий элемент задан, то изменение его входов
		switch (FNextIn)
		{
		case 1: FNextEl->setIn1(getRes()); 
			break;
		case 2: FNextEl->setIn2(getRes()); 
			break;
		}
}
```
Использование этих классов с учетом связей:
```cpp
	TNot elNot;
	TAnd elAnd;
	elAnd.Link(&elNot,1);
	for (int A=0; A<=1; A++)
	{
		elAnd.setIn1(A);
		for (int B=0; B<=1; B++)
		{
			elAnd.setIn2(B);
			cout << " " << A << " " << B << "    " << elNot.getRes() << endl;
		}
	}
```





