# HTB---Iterative_Virus-writeup


![1analyze](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/ff2935ab-e199-43ae-8208-a130e7281eb2)

Итак, на входе имеем exe-шник HELLO_WORLD_INFECTED.exe. Открываем в IDA pro и анализируем: 
1ая встречающая нас функция берет из PEB-структуры адрес  загруженной динамической библиотеки(KERNEL32.dll), далее - вычисляет от каждого имени функции из этой либы хэш и сравнивает его с хэшем функции
getProcAddr, тем самым находя адрес этой функции.
 ![hash](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/b6312a5d-0152-4c7d-8b80-c01ef10b36f1)


После этого, с помощью функции GetProcAddr "вирус" ищет еще несколько необходимых для его работы функций.


После нахождения всех необходимых функций, вирус проверяет на какой стадии мы находимся и, в зависимости от этого, помещает в переменную определенную константу(или вызывает код).

Если посмотреть на код, то мы увидим, что он обфусцирован. На этом этапе можно предположить, что мы должны провести над этим кодом ряд операций, а если точнее - 4, т.к. на 5й итерации он вызывается.
![case](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/749a5318-94c7-48aa-bbab-f5d82aa6d023)


Собсно, пойдемте дальше:
С помощью следующих нескольких функций вредоносная программа ищет в текущей директории все exe-файлы, а затем маппит их бинарный код к себе в адресное пространство.
Найдя и разместив exeшник к себе в память, вирус проверяет его на наличие MZ-сигнатуры, разрядности файла, а также на наличие сигнатуры "THIS" - b"53494854", которая указывает на то, что файл можно
инфицировать. 
![check_signature](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/5d2c0b7f-e99f-427e-8977-b1211799e331)


После проверок, описанных выше, вирус копирует секцию .ivir(свое тело) в заражаемый файл, при этом заменяя сигнатуру THIS на сигнатуру b"DEADC0DE", тем самым отмечая тот факт, что файл инфицирован.
Цикл, где реализовано копирование тела вируса в файл, представлен на фото:
![copy_ivir](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/357ac108-5008-407c-8fed-5f64999dd03b)




После завершения этого процесса, вирус модифицирует кусок своей секции размером 408 байт и если присмотреться к начальным байтам модифицируемого куска памяти, то можно обнаружить, что они совпадают
с начальными байтами обфусцированного кода, т.е. можно сделать вывод, что на данном участке кода происходит деобфускация пейлоада.
![deobfuscate_cycle](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/9dd5b4b7-36ae-4697-ab85-c850488fca35)
![deobfuscation_process](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/46fcf3b3-b51f-4ed4-b7e9-2f05167714d9)
Делаем вывод: Исходя из логики работы вируса, нам необходимо выполнить деобфускацию кода, перемножив его содержимое последовательно с константами, представленными в начале(в case-блоке)
Почему последовательно? Потому что после выполнения цикла по деобфускации кода, вирус увеличивает на единицу байт, который влияет на выбор константы, учавствующей в деобфускации:




Для решения этой задачи я решил написать декриптор на ассемблере, т.к. мне не хотелось замарачиваться над адаптацией представления данных ассемблера к языку высокого уровня.
https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/blob/main/decrypt.asm



После снятия дампа файла, закидываем его в IDA pro  и видим наш флажок : )
;![flag](https://github.com/VladimirAnderson/HTB---Iterative_Virus-writeup/assets/57271893/0c2ce039-7888-4841-b551-9096d9337b80)

