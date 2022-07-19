## Задача 1. Установите golang.
1. Инструкцию прочитали, но версия уже была установлена до этого. Только обновили ее.
```
[root@dz ~]# go version
go version go1.17.10 linux/amd64
```

## Задача 2. Знакомство с gotour.
У Golang есть обучающая интерактивная консоль [https://tour.golang.org/](https://tour.golang.org/). 
Рекомендуется изучить максимальное количество примеров. В консоли уже написан необходимый код, 
осталось только с ним ознакомиться и поэкспериментировать как написано в инструкции в левой части экрана.  

## Задача 3. Написание кода. 
Цель этого задания закрепить знания о базовом синтаксисе языка. Можно использовать редактор кода 
на своем компьютере, либо использовать песочницу: [https://play.golang.org/](https://play.golang.org/).

1. Напишите программу для перевода метров в футы (1 фут = 0.3048 метр). Можно запросить исходные данные 
у пользователя, а можно статически задать в коде.
    Для взаимодействия с пользователем можно использовать функцию `Scanf`:
    ```
    package main
    
    import "fmt"
    
    func main() {
        fmt.Print("Enter number a meters: ")
        var input float64
        fmt.Scanf("%f", &input)
    
        output := input * 2
    
        fmt.Println("Change to foot: ", output)    
    }
    ```
    Ответ: Исправляем ошибку в коде, немного добавляем информативности при вводе значений и выводе
    ```
    package main
    
    import "fmt"
    
    func main() {
        fmt.Print("Enter a meter: ")
        var input float64
        fmt.Scanf("%f", &input)
    
        output := input * 0.3048
    
        fmt.Println(output)    
    }
    ```
    ```
    [root@dz golang]# go run conversion_meter_to_foot.go
    Enter number a meters: 546
    Change to foot:  1092
    ```

1. Напишите программу, которая найдет наименьший элемент в любом заданном списке, например:
    ```
    x := []int{48,96,86,68,57,82,63,70,37,34,83,27,19,97,9,17,}
    ```
    ```
    package main
    
    import "fmt"
    
    func main() {
        x := []int{48,96,86,68,57,82,63,70,37,34,83,27,19,97,9,17,}
        minimal := x[0]
        for _, y := range x {
            if y < minimal {
                minimal = y
            }
        }
        fmt.Println("Minimal number is: ", minimal")
    }
    ```
    ```
    [root@dz golang]# go run search_minimal_number.go
    Minimal number is:  9
    ```

1. Напишите программу, которая выводит числа от 1 до 100, которые делятся на 3. То есть `(3, 6, 9, …)`.
    ```
    package main
    
    import "fmt"
    
    func main() {
        for x := 3; x <= 100; x = x +3{
           fmt.Print(x,", ") 
        }
    }
    ```
    ```
    [root@dz golang]# go run test.go
    3, 6, 9, 12, 15, 18, 21, 24, 27, 30, 33, 36, 39, 42, 45, 48, 51, 54, 57, 60, 63, 66, 69, 72, 75, 78, 81, 84, 87, 90, 93, 96, 99,
    ```