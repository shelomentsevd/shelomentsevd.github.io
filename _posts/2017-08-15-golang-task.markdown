---
layout: post
title: "Задача по Go для собеседования"
subtitle: ""
date: 2017-08-15 11:00:00
categories: [useful, golang]
---
Сейчас раздумывал над тем как получше спроектировать сервер для соревнования по High load и в процессе придумал небольшую задачку для собеседования Go-разработчиков.

# Задача
На сервере есть структура для хранения информации о пользователе
{% highlight go %}
type User struct {
    ID           int `json:"id"`,
    SecurityKey  string `json:"id"`
    Name         string `json:"name"`,
    Age          int `json:"age"`
}
{% endhighlight %}
сервер может обрабатывать два вида запросов:
1. GET /user/id - отдает пользователю структуру User перекодированную в json
2. POST /user/id - обновляет информацию о пользователе раскодируя json внутрь объекта User

Нужно изменить модель данных и запросы так, что бы данные по прежнему хранились в объекта User, но на POST клиент не мог изменить ID и SecurityKey. 

# Неправильное решение
Принять json, декодировать его в объект User и перед окончательной записью в базу проверить, что ID и SecurityKey не изменились, либо записать данные только из полей Name и Age.

# Как правильно
Структуру User можно разбить на две структуры для приватных полей(не могут быть изменены клиентом):<br/>
{% highlight go %}
type UserPrivate struct {
    ID          int `json:"id"`
    SecurityKey string `json:"id"`
}
{% endhighlight %}
И публичные полей(могут меняться клиентом):<br/>
{% highlight go %}
type UserPublic struct {
    Name string `json:"name"`,
    Age  int `json:"age"`
}
{% endhighlight %}
А саму структуру User скомпоновать из структур выше:<br/>
{% highlight go %}
type User struct {
    UserPrivate
    UserPublic
}
{% endhighlight %}
Теперь на GET /user/id можно кодировать в json как и раньше весь тип User, а на POST json раскодировать в объект UserPublic:<br/>
{% highlight go %}
// GET
user := User{UserPrivate{10}, UserPublic{"Ivan", 24}}
bytes, _ := json.Marshal(user)
// POST
update := UserPublic{}
err := json.Unmarshal(bytes, &update)
user.UserPublic = update
{% endhighlight %}
# Заключение
Это неплохая проверка на понимание того умеет ли пользоваться программист структурами в Go, а также, наверное, неплохой фильтр для людей ни разу не писавших Web сервер на этом языке т.к. этот кейс там встречается повсеместно.<br/>
# PS
Очень хорошо, что меня не спросили такое недели три назад так как до меня только сейчас дошло зачем в коде многих проектов на Go некоторые сущности так разбиты :D<br/>
[Конкурс](https://highloadcup.ru) о котором я говорил, постараюсь написать пост о своем участии, когда он завершится.
