
``` python 
#Импорт библиотек в самом начале кода

#случайные величины, время, система

import random, time, os

#import words

import tws

  

class Hangman:

  

    #Конструктор класса

    def __init__(self, words_param):

        self.words = words_param

        self.mainloop()

  

    def _new_game(self):

        self.attempts = 12

        self.selected_word = random.choice(self.words).lower()

        self.unguessed_string = "_" * len(self.selected_word)

        self.guessed_string = self.unguessed_string

  

    def mainloop(self):

  

        os.system("cls")

        is_running = True

        while is_running == True:

            self._new_game()

            print("Добро пожаловать в игру Висельник!")

            print("Выйти из игры можно только после её окончания.")

            user_input = input("Enter для продолжения или \"exit\" для выхода. \n")

            if user_input == "exit":

                is_running = False

                break

  

            while self.attempts != 0:

  

                if self.guessed_string.count("_") == 0:

                    print("Победа! Загаданное слово - ", self.selected_word)

                    break

  

                os.system("cls")

                print("Осталось попыток:", self.attempts)

                print(self.guessed_string)

                letter = input("Введите букву: ")

  

                if self.selected_word.count(letter) != 0:

  

                    self.unguessed_string = ""

                    for i in range(len(self.selected_word)):

                        if self.selected_word[i] == letter or self.guessed_string[i] != "_":

                            self.unguessed_string += self.selected_word[i]

                        else:

                            self.unguessed_string += "_"

                    self.guessed_string = self.unguessed_string

  

                else:

                    self.attempts -= 1

  

    def test(self):

        os.system("cls")

        print(self.words)

        print(self.attempts)

        print(self.selected_word)

        print(self.guessed_string)

  

words_list = tws.read_words("words.txt")

print(words_list)

s = Hangman(words_list)

s.test()

s.mainloop()
```


### **1. Инициализация:**

- Игра начинается с создания объекта класса `Hangman`, которому передаётся список слов
    
- Сразу запускается основной игровой цикл
    

### **2. Настройка новой игры:**

- Выбирается случайное слово
    
- Создаётся строка с подчёркиваниями (например, "____" для слова из 4 букв)
    
- Устанавливается счётчик попыток (12)
    

### **3. Игровой процесс:**

1. **Отображение состояния:**
    
    - Показывает количество оставшихся попыток
        
    - Отображает текущий прогресс (угаданные буквы и пропуски)
        
2. **Ввод буквы:**
    
    - Пользователь вводит одну букву
        
    - Если буква есть в слове:
        
        - Обновляется строка отображения (открываются все вхождения этой буквы)
            
    - Если буквы нет:
        
        - Уменьшается количество попыток на 1
            
3. **Условия завершения:**
    
    - **Победа:** когда все буквы угаданы (нет подчёркиваний в строке)
        
    - **Поражение:** когда закончились попытки
        
    - Игра продолжается, пока не выполнится одно из условий
        

### **4. Особенности реализации:**

- **Интерфейс:** консольный, с очисткой экрана между ходами
    
- **Слова:** загружаются из внешнего файла через модуль `tws`
    
- **Управление:** можно выйти из игры только между раундами
    
- **Логика обновления:** аккуратно обрабатывает множественные вхождения букв в слове
    

### **5. Критические моменты:**

- Нет обработки некорректного ввода (более одной буквы, цифры и т.д.)
    
- Нет отображения "виселицы" как графического элемента
    
- После победы/поражения игра автоматически начинается заново 