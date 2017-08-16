---
layout: post
title: "Ограничиваем количество горутин в Go"
subtitle: ""
date: 2017-08-15 15:41:00
categories: [useful, golang]
---

Если вам в Go нужно обрабатывать какие-то данные в не более чем пяти горутинах, то это можно сделать вот так:<br/>
{% highlight go %}
var wg sync.WaitGroup
// Create bufferized channel with size 5
goroutines := make(chan struct{}, 5)
// Read data from input channel
for data := range input {
    // 1 struct{}{} - 1 goroutine
    goroutines <- struct{}{}
    wg.Add(1)
    go func(data string, goroutines <-chan struct{}, results chan<- result, wg *sync.WaitGroup) {
        // Process data
        results <- process(data)
        // Read from "goroutines" channel to free space for new goroutine
        <-goroutines
        wg.Done()
    }(data, goroutines, results, &wg)
}

wg.Wait()
close(goroutines)
{% endhighlight %}

Максимальное количество горутин определяется размером канала goroutines. Когда из канала input поступают новые данные в канал отправляется пустая структура, затем создается горутина, которая после завершения читает один раз из goroutines. Таким образом, если у нас уже работает пять горутин, то цикл зависнет на седьмой строчке, пока внутри goroutines не освободится место.<br/>
Пришел к этому решению пока делал тестовое задание для Mailru, на тестовое задание и работу кода выше можно посмотреть [тут](https://github.com/shelomentsevd/interview-task-mail).