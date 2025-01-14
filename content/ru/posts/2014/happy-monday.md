---
title: Happy Monday!
date: "2014-08-11"
tags:
- Bugs
- dotnet
- Hate
- PostSharp
aliases:
- /ru/blog/dotnet/happy-monday/
- /ru/blog/post/happy-monday/
---

Хотелось бы рассказать историю одного волшебного бага. Волшебство его заключалось в том, что он не давал нам отлаживать по понедельникам. Я сейчас совершенно серьёзно: каждый понедельник у нас отваливался Debug mode. Мало того, этот баг с циничным видом желал нам счастливого понедельничка. Для меня это был очень ценный урок в плане того, какие же всё-таки разнообразные бывают проблемы. Возможно, кому-то ещё эта история покажется любопытной.

Итак, как же я впервые встретился с этой багой. Был замечательный вечер воскресенья, ничего не предвещало беды. В понедельник планировался очередной релиз нашей программки (ничего мажорного, но клиенты ждали обещанных мелких фич). На часах отображалось 00:00, и тут мне пришла в голову мысль, что один из пользовательских сценариев для новой фичи мы не проработали. Нужно было дописать всего несколько дополнительных строк, делов минут на 10. Я решил, что сейчас быстренько допишу нужную логику и с чистой совестью лягу спать. Запускаю студию, запускаю билд проекта, жду. И тут моё лицо становится озадаченным, т.к. я вижу ошибку:

```txt
Error connecting to the pipe server.
```

Хм... Странная ошибка-то какая-то.<!--more--> Тем более, что она не даёт сбилдить проект, не сообщая о себе никакой дополнительной информации: ни место, ни причину возникновения. Думаю: ну, мб, я в каких-то пользовательских настройках проекта что-то испортил. Говорю:	`git clean -f -x -d`, очисти мне всё, чего нет в репозитории. Запускаю билд, вижу ошибку:

```txt
Error connecting to the pipe server.
```

Хм... Ну, может чего-нибудь испортилось в самом последнем коммите? Откатываюсь на один коммит назад, на второй, на третий. Откатываюсь на месяц назад к супер-стабильной ревизии, с которой точно всё было хорошо. Запускаю билд, вижу ошибку:

```txt
Error connecting to the pipe server.
```

Хм... Ну, может чего-то в системном окружении сломалось? Перезагружаюсь, запускаю билд, вижу ошибку:

```txt
Error connecting to the pipe server.
```

Хм... Ну, может чего-то в системном окружении совсем-совсем сломалось? Переношу проект на чистый комп, запускаю билд, вижу ошибку:

```txt
Error connecting to the pipe server.
```

Часы начинают показывать 4-ый час ночи, а я ещё даже проект не сбилдил. В порядке протыкивания всего подряд меняю Configuration на Release. И о чудо — билд проходит успешно, приложение запускается. Возвращаю Debug — вижу ошибку:

```txt
Error connecting to the pipe server.
```

Хм... Ситуация начинает меня злить, я запускаю из консоли MSBUILD и начинаю изучать километровый лог. И в нём я нахожу чудесные строчки:

```txt
Starting the pipe server: "C:\ProgramData\PostSharp\3.1.28\bin.Release\postsharp.srv.4.0-x86.exe /tp "postsharp-S-1-5-21-1801181006-371574121-2664876850-1002-4.0-x86-release-3.1.28-a4c26157a4624bb9" /config "C:\ProgramData\PostSharp\3.1.28\bin.Release\postsharp.srv.4.0-x86.exe.config"".
  : info : Executing PostSharp 3.1 [3.1.28.0, 32 bit, CLR 4.5, Release]
  : message : Happy Monday! As every Monday, you're getting all the features of the PostSharp Ultimate for free.
  : message : PostSharp 3.1 [3.1.28.0, 32 bit, CLR 4.5, Release] complete -- 0 errors, 0 warnings, processed in 102 ms
```

Вы только поглядите: [PostSharp](http://www.postsharp.net/)	решил пожелать мне счастливого понедельника, угостив плюшками Ultimate-версии (которые мне не особо нужны), но сломав при этом построение моего проекта! Не ожидал я от него такого, не ожидал. Но почему же только отлаживать было нельзя? А всё потому, что мы использовали PostSharp для логирования в Debug mode, в Release он не попадал.

Если у вас возникла подобная проблема, то лечится она просто. Некоторое гугление даёт нам информацию о том, что в недавних версиях PostSharp-а были различные дефекты (например, такой: [Critical Defect in PostSharp 3.1](http://www.postsharp.net/blog/post/URGENT-ACTION-REQUIRED-Critical-Defect-in-PostSharp-31-process-exits-with-code-199)), которые могли сделать вашу жизнь достаточно грустной. Описанный выше баг является достаточно старым. Я нашёл историю от марта 2012 (PostSharp 2.1) с замечательным названием [«PostSharp bugs that occur only on a Monday? Really? :(»](https://plus.google.com/113181962167438638669/posts/QF5pDB4XY6F). К счастью, в актуальной версии PostSharp-а проблема вроде бы ушла. У меня на 3.1.48 всё заработало хорошо. Счастливого понедельника PostSharp мне всё ещё желает, но pipe server при этом остаётся целым.

### Выводы

Мне сложно сформировать какую-то конкретную мораль этой истории. Скажу только то, что она научила меня шире смотреть на проблемы, которые могу возникнуть. А ещё, если студия плюётся в вас какими-то непонятными ошибками, то в первую очередь нужно не думать о проблеме и о возможных причинах её возникновения, а сразу смотреть лог от MSBUILD-а. А ещё я выяснил, что за последние несколько месяцев никто в нашей команде не отлаживал приложение по понедельникам =)