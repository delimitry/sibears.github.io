---
layout: post
categories: writeups
date: "2017-10-21 14:34:23 +0300"
title: "[FlareOn2017] pewpewboat"
author: groke
tags: [writeup, FlareOn2017]
---

В этом году я решил поучавствовать в FlareOn2017 и даже смог его
успешно завершить на 9м месте:)

Собственно решил написать небольшой разбор того как я решал 5-е задание, потому
что мое решение слегка отличается от того, которое было представлено в
официальном разборе.

Нам дан файлик pewpewboat.exe и это 64-битный elf :)

Если его немного поковырять, то можно понять что это игра "Морской бой" и цель
участника потопить все корабли N раз. При этом расположение самих кораблей не
меняется, но поле хранится в некой хитрой структурке, которая раскодируется в
зависимости от того как ты настрелял в прошлом раунде (или типо того, я особо не
успел разобрать алго ;[ )

Из интересного:

1.  По адресу `0x4038ba` вызывается некая хитрая функция, которая делает
    какие-то странные манипуляции с введенной строкой и md5

2.  В структуре, которая хранит текущее поле, корабли лежат в битовом векторе длины 64
    (четверное слово) по смещению 0.

3.  После того как мы потопили все корабли на текущем поле, мы получаем от
        бинаря строчку и должны ему отдать `!md5(string)`, то есть посчитать md5
        от нее и применить логическое НЕ к каждому байту. Эта функция находится
        по адресу `0x403411`

4.  На каждом поле корабли расположены в форме буквы латинского алфавита
    

Я даже думал начать разбирать алгоритм декодирования поля, но потом мне стало лень, и я подумал: "А почему бы мне не использовать магию GDB?"
 

В итоге получился вот такой скриптик:

{% highlight python %}import gdb

COUNTER = 1

def write_picture(pos):
   lett = "ABCDEFGH"
   dig = "12345678"
   comm = [" " for i in range(64)]
   for i in pos:
       comm[8*lett.index(i[0])+dig.index(i[1])] = 'X'
   out = ""
   for i in range(8):
       out += "".join(comm[8*i:8*(i+1)])+"\n"
   print(out)
   return out


class PewPewSolver (gdb.Command):

  def __init__ (self):
    self.counter = 0
    self.cnt = 1
    super (PewPewSolver, self).__init__ ("ships", gdb.COMMAND_USER)

  def invoke (self, arg, from_tty):
    val = 0xffffffff & gdb.parse_and_eval("*$rdi")
    val3 = 0xffffffff & gdb.parse_and_eval("*($rdi+4)")
    val = (val3 * 2**32) + val
    lett = "ABCDEFGH"
    dig = "12345678"
    ans = ""
    self.counter += 1
    if self.cnt == self.counter:
        arr_ship = []
        for i in range(64):
            if (val >> i) & 1:
                pos = (lett[i>>3]+dig[i&7])
                ans += pos
                arr_ship.append(pos)
                ans += "\n"
        f = open("letter_{}".format(self.cnt), "w")
        letter = write_picture(arr_ship)
        f.write(letter)
        f.close()
        f = open("pew_input.txt", "a")
        f.write(ans)
        f.close()
        self.cnt = self.counter + 1
        self.counter = 0
        print("Restarting!")
        gdb.execute("r < pew_input.txt")
    else:
        print("skipped")
        gdb.execute("c")

PewPewSolver ()
{% endhighlight %}

Это реализация команды, которая парсит текущее расположение кораблей и затем
пишет его во входной файл и перезапускает прогу, подавая на вход этот файл. При
этом она считает кол-во сработавших брейкпоинтов, чтобы записывать в файл только
новые :) И каждое расположение кораблей пишет в отдельный файлик

Далее мы нопаем вызов функции с md5 (плохой подход, но эта функция никак не
влияет на следующее поле, так что не страшно)

Запускаем gdb следующей командой: `gdb -x gdbscr ./pewpewboat.exe`

Где `gdbscr` имеет вид:

{% highlight python %}source pew_solver.py
br 0x403c0d
commands
ships
end
r
{% endhighlight %}

После запуска проверяем наши файлики: 
{% highlight python %}user@ubuntu:~$ cat letter_1
        
   XXXX 
   X    
   X    
   XXXX 
   X    
   X    
        
user@ubuntu:~$ cat letter_2
        
   X   X
   X   X
   X   X
   XXXXX
   X   X
   X   X
        
{% endhighlight %}

Помимо этого бинарь выплюнул нам следующую строчку:
`Aye!PEWYouPEWfoundPEWsomePEWlettersPEWdidPEWya?PEWToPEWfindPEWwhatPEWyou'rePEWlookingPEWfor,
PEWyou'llPEWwantPEWtoPEWre-orderPEWthem:PEW9,PEW1,PEW2,PEW7,PEW3,PEW5,PEW6,PEW5,PEW8,PEW0,PEW2,
PEW3,PEW5,PEW6,PEW1,PEW4.PEWNextPEWyouPEWletPEW13PEWROTPEWinPEWthePEWsea!
PEWTHEPEWFINALPEWSECRETPEWCANPEWBEPEWFOUNDPEWWITHPEWONLYPEWTHEPEWUPPERPEWCASE.`

Убираем все PEW (`%s/PEW/ /g`):

`Aye! You found some letters did ya? To find what you're looking for, you'll
want to re-order them: 9, 1, 2, 7, 3, 5, 6, 5, 8, 0, 2, 3, 5, 6, 1, 4. Next  you
let 13 ROT in the sea! THE FINAL SECRET CAN BE FOUND WITH ONLY THE UPPER CASE.`

Дальше переупорядочиваем буквы и отдаем бинарю, на что он в ответ даст нам флаг:)

 Небольшая демонстрация здесь: <https://asciinema.org/a/TvFVXdurqpNrztoZY281TfVel>
