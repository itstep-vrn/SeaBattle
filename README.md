# Игра "Морской бой"

***
Учебный проект по игре "Морской бой".
***

## Описание игры
«Морской бой» — игра для двух участников, в которой игроки по очереди называют координаты на неизвестной им карте соперника. Если у соперника по этим координатам имеется корабль (координаты заняты), то корабль или его часть «топится», а попавший получает право сделать ещё один ход. Цель игрока — первым потопить все корабли противника.

Игровое поле имеет площадь 10x10. На игровом поле размещаются:  
1 корабль — ряд из 4 клеток («четырёхпалубный»; линкор)  
2 корабля — ряд из 3 клеток («трёхпалубные»; крейсера)  
3 корабля — ряд из 2 клеток («двухпалубные»; эсминцы)  
4 корабля — 1 клетка («однопалубные»; торпедные катера)  

При размещении корабли не могут касаться друг друга сторонами и углами. Расстояние между кораблями 1 клетка.

Данная игра работает по сети. Покдлючение к другому компьютеру осуществляется через IP компьютера.  
Закрашенная фигура - Корабль или часть корабля целая  
Заштрихованная фигура - Корабль подбит (ранен)  
\* - Промах  
X - Корабль уничтожен  

## Описание кода программы

Подключаем библиотеки и пространство имен:  
```cpp
#include "pch.h" // Библитоека предкомпилированных заголовков
#include <iostream> // Библиотека ввода и вывода информации
#include <winsock2.h> // Библиотека для работы с сокетами (работе по сети)
#pragma comment(lib, "ws2_32.lib") // Подключаем 
#pragma warning(disable: 4996) // Игнорируем предупреждение

using namespace std;
```
Объявляем глобально наши массивы которые будут выполнять функцию игрового поля, переменные и сокет 
```cpp
const int N = 11;
int corX, corY, winValue; // Объявлем переменные которые будут являться координатами наших кораблей и переменную для проверки выигрыша
int fieldPlayer[N][N];
int fieldComp[N][N];
SOCKET Connection; // Подключаем соккет для работе по сети
```
Функции 
```cpp
void showEmptyField(int numberArray); // Функция отрисовки игрового поля

void enterLinkor();     // Функция для размещения на игровом поле 4-х палубного корабля
void enterCruiser();    // Функция для размещения на игровом поле 3-х палубного корабля
void enterDestroyer();  // Функция для размещения на игровом поле 2-х палубного корабля
void enterBoat();       // Функция для размещения на игровом поле 1-о палубного корабля

int ClientHandler();    // Функция для получения сообщения с координатами хода противника и отправки сообщения с изменившимися координатми на игровом поле и проверкой попадания или промаха

int shoting();          // Функция для отправки координат стрельбы и получения результата

void chekWounded();     // Функция для проверки раненых кораблей и их перезаписи в убитых

int winGame();          // Функция проверки проигрыша и выигрыша игрока
```
Вызываем код программы в котором выполняется функционал нашей игры
```cpp
int main()
{
	int temp = 0;
	int buff = 0;
  // Заполняем игровое поле игрока 0
	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < N; j++)
		{
			fieldPlayer[i][j] = 0;
		}

	}
  // Заполняем игровое поле противника 0
	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < N; j++)
		{
			fieldComp[i][j] = 0;
		}

	}
	enterLinkor();        // Расставляем 4-х полабуный корабль
	enterCruiser();       // Расставляем 3-х полабуный корабль
	enterCruiser();       // Расставляем 3-х полабуный корабль
	enterDestroyer();     // Расставляем 2-х полабуный корабль
	enterDestroyer();     // Расставляем 2-х полабуный корабль
	enterDestroyer();     // Расставляем 2-х полабуный корабль
	enterBoat();          // Расставляем 1-о полабуный корабль
	enterBoat();          // Расставляем 1-о полабуный корабль
	enterBoat();          // Расставляем 1-о полабуный корабль
	enterBoat();          // Расставляем 1-о полабуный корабль

  // Подключаемся по сети
	WSAData wsaData;
	WORD DLLVersion = MAKEWORD(2, 1);
	if (WSAStartup(DLLVersion, &wsaData) != 0) {
		std::cout << "Error" << std::endl;
		exit(1);
	}
	SOCKADDR_IN addr;
	int sizeofaddr = sizeof(addr);
	addr.sin_addr.s_addr = inet_addr("192.168.88.138");
	addr.sin_port = htons(1111);
	addr.sin_family = AF_INET;

	Connection = socket(AF_INET, SOCK_STREAM, NULL);
  // Проверка подключения по сети между игроками
	if (connect(Connection, (SOCKADDR*)&addr, sizeof(addr)) != 0)
	{
		cout << "Error: failed connect to server.\n";
		return 1;
	}
  // Выполняем программу, которая будет работать пока не выполнится условие победы и проигрыша
	do
	{
    // Выполняем программу до тех пор пока противник не промахнется
		do
		{
			temp = ClientHandler();
			chekWounded();
		} while (temp != 2);
    // Выполняем программу до тех пор пока стреляющий не промахнется
		do
		{
			buff = shoting();
			chekWounded();
		} while (buff != 2);
		winValue = winGame();
	} while (winValue != 1);
}
```
