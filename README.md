# Блог команды Sibears

Блог размещён на Github Pages и работает на Jekyll.

## Хочу написать пост

Для размещения статьи в блоге потребуются навыки работы с Git и умение ясно излагать мысли. Хотя хватит только навыков работы с Git. 

Каждая статья представлена файлом в каталоге `_posts`. За основу стоит взять какую-нибудь опубликованную статью.
Но можно и ознакомиться с [официальным руководством](https://jekyllrb.com/docs/posts/).

Код можно размещать несколькими способами:
 - Скриншоты
 - Код в тексте статьи с форматированием 
 ```{% highlight lang %}```
 - Github Gist ```{% gist ... %}```
 
Прикреплённые изображения и файлы нужно сохранить в каталоге 'assets'. Затем на них можно [дать ссылку из статьи](https://jekyllrb.com/docs/static-files/). 
 
Полезное:
 - [Пример разметки](https://raw.githubusercontent.com/sibears/sibears.github.io/master/test.md)
 - [Эмоджи](https://www.webpagefx.com/tools/emoji-cheat-sheet/)

## Хочу проверить как пост выглядит до коммита в репозитарий

Нет ничего страшного в том, чтобы закинуть сырую статью в блог и выточить её внешний вид за несколько коммитов.

Но можно запустить блог локально и отредактировать статью до публикации. Для этого придётся разобраться с [Jekyll](https://jekyllrb.com/docs/home/).

    $ gem install jekyll  
    $ bundle exec jekyll serve
    $ open http://127.0.0.1:4000
     
## Хочу изменить внешний вид блога

Разметка хранится в каталогах `_layouts` и `_includes`, исходные стили – в `_css`. 
Для сборки стилей потребуется postcss:

    $ yarn install && yarn build -w
