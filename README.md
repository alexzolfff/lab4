МИНИСТЕРСТВО НАУКИ  И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ  

Федеральное государственное автономное образовательное учреждение высшего образования  

"КРЫМСКИЙ ФЕДЕРАЛЬНЫЙ УНИВЕРСИТЕТ им. В. И. ВЕРНАДСКОГО"  

ФИЗИКО-ТЕХНИЧЕСКИЙ ИНСТИТУТ  

Кафедра компьютерной инженерии и моделирования
<br/><br/>
### Отчёт по лабораторной работе № 4<br/> по дисциплине "Программирование"
<br/>
​Cтудента 1 курса группы ПИ-б-о-192(2)<br/>
Золотой Александр Витальевич<br/>
направления подготовки 09.03.04 "Программная инженерия"  
<br/>


<br/>
<table>

<tr><td>Научный руководитель<br/> старший преподаватель кафедры<br/> компьютерной инженерии и моделирования</td>

<td>(оценка)</td>

<td>Чабанов В.В.</td>

</tr>

</table>

<br/><br/>

​

<center> Симферополь, 2020 </center>

<br/>

## Лабораторная работа №4.Иксики-нолики

***Цель :***  

1. Закрепить навыки работы с перечислениями;
2. Закрепить навыки работы с структурами;
3. Освоить методы составления многофайловых программ.

***Ход работы:***

------

Lab4.cpp

```c++
#include <iostream>
#include "pch.h"
#include "Lab4.h"
#include <ctime> 

using namespace std;

int main() {
	cout << "Lets play" << endl;
	setlocale(LC_ALL, "Russian");
	srand(std::time(NULL));
	char h;
	while (1) {
		cout << "Plese choise 0 or x" << endl;
		cin >> h;
		if (h == 'X' || h == 'O' || h == '0' || h == 'x') break;
	}
	


Game game = initGame(h);

if (game.isUserTurn) {
	while (1) {
	userTurn(&game);
	updateDisplay(game);
		if (updateGame(&game)) {
			break;
		}
	botTurn(&game);
	updateDisplay(game);
		if (updateGame(&game)) {
			break;
		}
	}
}
else {
	while (1) {
		botTurn(&game);
	updateDisplay(game);
		if (updateGame(&game)) {
			break;
		}
	userTurn(&game);
	updateDisplay(game);
		if (updateGame(&game)) {
			break;
		}
	}
}
switch (game.status) {
case USER_WIN:
	cout << "Вы победили";
	break;
case BOT_WIN:
	cout << "Вы проиграли";
	break;
case NOT_WIN:
	cout << "Ничья";
	break;
}
return  0;
}
```

Lab4Header.h
```c++
#ifndef Lab3;
#define Lab3

enum Status {
	PLAY,            // Игра продолжается
	USER_WIN,        // Игрок победил
	BOT_WIN,         // Бот победил
	NOT_WIN          // Ничья. Победителя нет, но и на поле нет свободной ячейки
};

struct Game {
	char bord[3][3];  // Игровое поле
	bool isUserTurn;  // Чей ход. Если пользователя, то isUserTurn = true
	char userChar;    // Символ которым играет пользователь
	char botChar;     // Символ которым играет бот
	Status status;
};

Game initGame(char userChar);
void updateDisplay(const Game game);
void botTurn(Game* game);
void userTurn(Game* game);
bool updateGame(Game* game);
#endif
```

Lab4Sourse.cpp
```c++
//Вспомогательный файл
#include <iostream>
#include "pch.h"


using namespace std;
enum Status{
	PLAY,            // Игра продолжается
	USER_WIN,        // Игрок победил
	BOT_WIN,         // Бот победил
	NOT_WIN          // Ничья. Победителя нет, но и на поле нет свободной ячейки
};

struct Game {
	char bord[3][3];  // Игровое поле
	bool isUserTurn;  // Чей ход. Если пользователя, то isUserTurn = true
	char userChar;    // Символ которым играет пользователь
	char botChar;     // Символ которым играет бот
	Status status;
};

Game initGame(char userChar) {
	Game game;
	//очистить масссив
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++){
			game.bord[i][j] = ' ';
		}
	}
	if (rand() % 2)   game.isUserTurn = true;//определяем чей ход
	game.userChar = userChar; //Устанавливает симвод для Игрока (Задаётся параметром userChar)
	switch (game.userChar) { //устанавливает символ бота
	case 'O':
		game.botChar = 'X';
		break;
	case 'X':
		game.botChar = 'O';
		break;
	
	case '0':
		game.botChar = 'x';
		break;
	case 'x':
		game.botChar = '0';
		break;
    }
	return game;
} 

void updateDisplay(const Game game) {
	system("cls");
	cout << "0  1  2" << endl;
	// вывод игрового поля
	cout << "----------" << std::endl;
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			cout << game.bord[i][j] << "|  ";
		}
		cout << i << endl << "----------" << endl;
	}
}

/*
Выполняет ход бота. В выбранную ячейку устанавливается символ которым играет бот.
Бот должен определять строку, столбец или диагональ в которой у игрока больше всего иксиков/ноликов и ставить туда свой символ. Если на поле ещё нет меток, бот должен ставить свой знак в центр. В остальных случаях бот ходит рандомно.
*/
void botTurn (Game *game) {
	bool empty  = true;
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			if (game->bord[i][j] != ' ') {
				empty = false;
				break;
			}
		}
	}
		if (empty) {
			game->bord[1][1] = game->botChar;
		}
		else {
			
			for (auto& i : game->bord) {
				int val = 0;
				int len = 0;
				bool flag = false;
				for (int j = 0; j < 3; j++) {
					if (i[j] == game->userChar) {
						len++;
					}
					else if (i[j] != game->botChar) {
						val = j;
						flag = true;
					}
				}
				if ((len == 2) && flag) {
					i[val] = game->botChar;
					return;
				}
			}

			for (int i = 0; i < 3; i++) {
				int val = 0;
				int len = 0;
				bool flag = false;
				for (int j = 0; j < 3; j++) {
					if (game->bord[j][i] == game->userChar) {
						len++;
					}
					else if (game->bord[j][i] != game->botChar) {
						val = j;
						flag = true;
					}
				}
				if ((len == 2) && flag) {
					game->bord[val][i] = game->botChar;
					return;
				}
			}

			{
				int val = 0;
				int len = 0;
				bool flag = false;
				for (int i = 0; i < 3; i++) {
					if (game->bord[i][i] == game->userChar) {
						len++;
					}
					else if (game->bord[i][i] != game->botChar) {
						val = i;
						flag = true;
					}
				}
				if ((len == 2) && flag) {
					game->bord[val][val] = game->botChar;
					return;
				}
			}

			{
				int val = 0;
				int len = 0;
				bool flag = false;
				for (int i = 0; i < 3; i++) {
					if (game->bord[i][2 - i] == game->userChar) {
						len++;
					}
					else  if (game->bord[i][2 - i] != game->botChar) {
						val = i;
						flag = true;
					}
				}
				if ((len == 2) && flag) {
					game->bord[val][2 - val] = game->botChar;
					return;
				}
			}

			int i = 0, j = 0;
			while (game->bord[i][j] != ' ') {
				i = rand() % 3;
				j = rand() % 3;
			}
			game->bord[i][j] = game->botChar;
		}
}




void userTurn(Game *game) { 
	int i, j;
	while (1) {
		cout << "Введите координаты x и у, соответственно:";
		cin >> i >> j;
		
		if (i < 0 || j < 0 || i>3 || j>3|| &game->bord[i][j] == " ")
			cout << "Retry!!!";
		else {
			game->bord[j][i] = game->userChar;
			break;
		}
		
	    
	}
}
/*
Функция определяет как изменилось состояние игры после последнего хода.

Функция сохраняет новое состояние игры в структуре game и передаёт ход другому
игроку.
Функция возвращает true, если есть победитель или ничья, иначе false.
*/

bool updateGame(Game* game) {
	
		for (int i = 0; i < 3; i++) {
			int len2 = 0;
			int len1 = 0;
			for (int j = 0; j < 3; j++) {
				if (game->bord[i][j] == game->userChar) {
					len1++;
				}
				else if (game->bord[i][j] == game->botChar) {
					len2++;
				}
			}
			if ((len1 == 3) || (len2 == 3)) {
				game->status = (len1 == 3 ? USER_WIN : BOT_WIN);
				return true; // есть победитель
			}
		}
		int len = 0;
		for (int i = 0; i < 3; i++) {
			for (int j = 0; j < 3; j++) {
				if (game->bord[i][j] != ' ') {
					len++;
				}
			}
		}
		if (len == 9) {
			game->status = NOT_WIN;
			return true;//ничья
		}
		game->status = PLAY;
		return false;// победителя нет
	
}
```
***Вывод:*** в ходе лабораторной работы я закрепил навыки работы с перечислениями и со структурами, освоил методы составления многофайловых программ. 
