# Основы ООП

## Классы

Объявление класса:
```cpp
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
Конструкторы и методы класса:
```cpp
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
```







