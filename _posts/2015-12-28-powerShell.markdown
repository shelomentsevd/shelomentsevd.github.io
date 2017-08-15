---
layout: post
title:  "Парочка полезных сниппетов для PowerShell"
subtitle: ""
date:   2015-12-28 17:09:01
categories: [useful]
---

Нужен был скрипт, для загрузки последней сборки в zip архиве и распаковки содержимого в каталог с сайтом.
Пока писал нашёл для себя пару полезных сниппетов.

Распаковка Zip архива

{% highlight PowerShell %}
function Expand-ZIPFile($file, $destination)
{
    $shell = new-object -com shell.application
    $zip = $shell.NameSpace($file)
    foreach($item in $zip.items())
    {
        $shell.Namespace($destination).copyhere($item, 16)
    }
}
{% endhighlight %}

Логинимся на сайте и загружаем файл

{% highlight PowerShell %}
function DownloadFile($user, $password, $url, $destination)
{
    If (Test-Path $destination) {
        Remove-Item $destination
    }
    $client = new-object System.Net.WebClient
    $client.Credentials = 
        new-object System.Net.NetworkCredential($user, $password)
    $client.DownloadFile($url, $destination)
}
{% endhighlight %}

Вообще, powerShell довольно мощная штука. Жаль мне не удалось найти способа удалённо работать с ним из linux, так бы ему цены не было. 
