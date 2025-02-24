Здесь я пишу заметки по управлению дисплеем МЭЛТ MT-20S4M в среде GCBASIC с помощью микроконтроллера ATmega162. Учитывая что среда GCBASIC поддерживает большое количество контроллеров можно использовать любой поддерживаемый.  

БАЗА
=====

Для начала назначим переменные на выводы соответствующие подключению:
    
    #chip mega162, 16   
    
    #define Melt_AO PortD.4
    #define Melt_RW PortD.5
    #define Melt_E PortD.6
    #define Melt_LED PortE.1
    #define Melt_DB PortC

    Dir Melt_AO Out
    Dir Melt_RW Out
    Dir Melt_E Out
    Dir Melt_LED Out

    Dir Melt_DB.0 out
    Dir Melt_DB.1 out
    Dir Melt_DB.2 out
    Dir Melt_DB.3 out
    Dir Melt_DB.4 out
    Dir Melt_DB.5 out
    Dir Melt_DB.6 out
    Dir Melt_DB.7 out

    //первоначальные установки портов
    set Melt_DB = 0b00000000
    set Melt_AO = false
    set Melt_RW = false
    set Melt_E = false
    set Melt_LED = false

Инициализация
=============
В соответствии с диаграммой и алгоритмом указных в документации на дисплей, проведем инициализацию(используется 8 битный режим):

    set Melt_E = true
    set Melt_DB = 0b00110000
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 40 us

    set Melt_E = true
    set Melt_DB = 0b00110000
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 40 us

    set Melt_E = true
    set Melt_DB = 0b00110000
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 40 us

    set Melt_E = true
    set Melt_DB = 0b00111000
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1000 us

    set Melt_E = true
    set Melt_DB = 0b00001000
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1000 us

    set Melt_E = true
    set Melt_DB = 0b00000001
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1000 us

    set Melt_E = true
    set Melt_DB = 0b00000110
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1000 us

Выбор вида курсора. Значение двух младших битов может принимать четыре значения 00 01 10 11. В текущем значении курсор неотображается, для остальных смотреть документацию на дисплей.
    
    set Melt_E = true
    set Melt_DB = 0b00001100 //<<Изменяемое значение
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1000 us

Выводим надпись на дисплей
==========================
Установка курсора

    set Melt_E = true
    set Melt_DB = 0b11000000 //<<задается позиция курсора
    set Melt_E = false
    set Melt_DB = 0b00000000
    Wait 1 ms

Так как адреса позиции курсора имеют не последовательный набор при переходе на следующую строку и для общего удобства, так как придется этот параметр писать часто, сделаем подпрограмму:

    Sub MoveCursor (in ln, in ro)
        set Melt_E = true
        Dim DB as Byte
        if ln = 1 and ro => 0 and ro <= 19 then
            set DB = 0b10000000
            set Melt_DB = DB + ro
        end if

        if ln = 2 and ro => 0 and ro <= 19 then
            set DB = 0b11000000
            set Melt_DB = DB + ro
        end if

        if ln = 3 and ro => 0 and ro <= 19 then
            set DB = 0b10010100
            set Melt_DB = DB + ro
        end if

        if ln = 4 and ro => 0 and ro <= 19 then
            set DB = 0b11010100
            set Melt_DB = DB + ro
        end if
        set Melt_E = false
        set Melt_DB = 0b00000000
        Wait 1 ms
    End Sub

Подпрограмма для вывода символа, код символа будет соответствовать таблице из документации на дисплей

    Sub SimbolNum (in num as byte)
      set Melt_AO = true
      set Melt_E = true
      set Melt_DB = num //<<Установка значения
      set Melt_E = false
      set Melt_DB = 0b00000000 
      set Melt_AO = false
      Wait 2 ms
    End sub

Вызовем вышеописанные подпрограммы для вывода надписи на дисплей:

    MoveCursor (2, 4)
    SimbolNum (0x48) //H
    SimbolNum (0x65) //e
    SimbolNum (0x6C) //l
    SimbolNum (0x6C) //l
    SimbolNum (0x6F) //o
    SimbolNum (0x20) //Space
    SimbolNum (0x57) //W
    SimbolNum (0x6F) //o
    SimbolNum (0x72) //r
    SimbolNum (0x6C) //l
    SimbolNum (0x64) //d
    SimbolNum (0x21) //'!'

![Вывод на экран "Hello World"](https://github.com/user-attachments/assets/bb4537d1-fcb0-4d70-ba8c-6b84715495da)


Новый символ
============
Дисплей позволяет создавать новый символ. Сначала устанавливается адрес первого пикселя. Подпрограмма:

    Sub NewSimbAdr (in adr)
        set Melt_E = true
        set Melt_DB = adr
        set Melt_E = false
        set Melt_DB = 0b00000000
        Wait 4 ms
    end Sub

Подпрограмма рисования символа

    sub NewSimb (in Write)
        set Melt_AO = true
        set Melt_E = true
        set Melt_DB = Write
        set Melt_E = false
        set Melt_DB = 0b00000000
        set Melt_AO = false
        Wait 100 us
    end Sub

Рисуем символы

    NewSimbAdr (0b01000000)
    //Первый
    NewSimb (0b00000100)
    NewSimb (0b00000100)
    NewSimb (0b00000101)
    NewSimb (0b00000110)
    NewSimb (0b00001100)
    NewSimb (0b00010100)
    NewSimb (0b00000100)
    NewSimb (0b00000100)
    //Второй
    NewSimb (0b00011111)
    NewSimb (0b00000100)
    NewSimb (0b00000100)
    NewSimb (0b00000100)
    NewSimb (0b00011111)
    NewSimb (0b00011111)
    NewSimb (0b00011111)
    NewSimb (0b00011111)
    //Третий
    NewSimb (0b00011111)
    NewSimb (0b00010001)
    NewSimb (0b00010001)
    NewSimb (0b00010001)
    NewSimb (0b00000100)
    NewSimb (0b00001010)
    NewSimb (0b00010001)
    NewSimb (0b00001010)
    //Четвертый
    NewSimb (0b00001010)
    NewSimb (0b00010101)
    NewSimb (0b00001010)
    NewSimb (0b00000100)
    NewSimb (0b00011011)
    NewSimb (0b00010101)
    NewSimb (0b00001010)
    NewSimb (0b00010101)

И выводим на дисплей
    
    MoveCursor (3, 8)
    SimbolNum (0x00) //первый
    SimbolNum (0x01) //второй
    SimbolNum (0x02) //третий
    SimbolNum (0x03) //четвертый
    
![Вывод на экран дополнительных символов](https://github.com/user-attachments/assets/443fb228-6967-4c73-8347-fbd792b893cf)
