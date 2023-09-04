# Тема Hugo ePub maker
## hugo-epub-maker — тема, позволяющая превратить генератор статических сайтов HUGO в генератор электронных книг (.ePub).

Простой и удобный генератор электронных книг, для создания электронных книг в формате **ePub**, текст которых подготовлен в файлах .md.

## Формат ePub

При разработке темы учитываются **W3C Recommendation [epub3.3](https://www.w3.org/TR/epub-33)**.

Структура файла выглядит следующим образом:

```bash
├── mimetype
├── META-INF
│   └── container.xml
└── OEBPS
    ├── chapter
    │   ├── chapter-1.xhtml
    │   ├── chapter-1
    │   │   ├── image-1-1.jpg
    │   │   └── image-1-2.jpg
    │   ├── chapter-2.xhtml
    │   ├── chapter-3.xhtml
    │   ├── chapter-3
    │   │   └── image-3.jpg
    │   ...
    │
    ├── content.opf
    ├── cover.jpg
    ├── css
    │   └── stylesheet.css
    ├── inhaltsverzeichnis.xhtml
    ├── titelseite.xhtml
    └── toc.ncx
```

### Работа тема epub as HUGO

* `mimetype`, `container.xml` и `stylesheet.css` — это статические файлы, расположенные в каталоге `static/` темы ePub-ReFreshing.
* Изображение обложки `cover.jpg` хранится в каталоге `static/OEBPS`.

Остальные файлы Hugo генерирует из MD-файлов содержащихся в каталоге **content** пользовательского уровня HUGO-проекта. Его структура может иметь следующую структуру:

``` bash
content
├── _index.md
├── inhaltsverzeichnis.md
├── blog
│   ├── chapter-1
│   │   ├── image-1-1.jpg
│   │   ├── image-1-2.jpg
│   │   └── index.md
│   ├── chapter-2.md
│   └── chapter-3
│       ├── image-3.jpg
│       └── index.md
└── titelseite.md
```

В проекте Hugo условно можно выделить два уровня: «высокий» (пользовательский) и «низкий» (тема) 

Важно обратить внимание на служебные файлы ***_index.md***, _content.opf_ и _toc.ncx\_, которые должны всегда находиться в корне каталога **content**. Файл **\_index.md** участвует в создании обязательных файлов _content.opf_ и _toc.ncx_ с учетом сответствующих параметров конфигурационного файла _hugo.yaml_ либо _hugo.toml_ или _hugo.json_ (ранее — _config.toml_ и пр.) и макета темы.  Настройка:

```yaml
cascade:
  _build:
    publishResources: false
```

Установками Front Matter (конфигурация страницы) определяется, что только масштабированные изображения попадут в директорию с электронной книгой, а исходные файлы не будут скопированы вместе с ними — что в противном случае HUGO сделал бы по умолчанию.

Файлы **inhaltsverzeichnis.md** и **titelseite.md** создают страницы, необходимые в EPUB публикации (ebook). Эти файлы обязательные. Они не могут быть удалены или переименованы. Содержимое (content) берется в соответствии с параметрами _config.toml_. В конфигурации страницы (Front Matter) параметр `ebook: toc` соответствует (-ющий) `ebook: cover` определяют, что шаблон _single.xhtml_ обрабатывает эти особые случаи не так как стандартные.


Фактический макет темы в themes/epub/layouts состоит всего из трех файлов:

#### Шаблоны - layouts

Актуальный макет темы (шаблоны) находится в директории **themes/epub/layouts** и состоит из трех файлов:

``` sh
├── _default
│   └── single.xhtml
├── index.ncx
├── index.opf
```

Они предназначены:

1. **single.xhtml** - для создания всех страниц _xhtml_
1. **index.ncx** служит шаблоном для _toc.ncx_
1. **index.opf** создает _content.opf_.

Все они управляются конфигурационным файлом _config.toml_. Менять эти файлы не рекомендуется. Также, все файлы используемые электронной книгой должны быть зарегистрированы в манифесте _manifest_ (`<manifest></manifest>`) файла _content.opf_. Это производится автоматически со всеми файлами директории *content*. Но, если, к примеру, на страницах есть ссылки на статические изображения, их придется добавлять в _manifest_ вручную. Все это обеспечивает _index.opf_.

### Общие настройки - файл config.toml

Конфигурационный файл **config.toml** содержит все метаданные для электронной книги и обеспечивает корректную структуру создаваемого файла:

#### 1. disableKinds

**disableKinds** - отключает почти всё, что Hugo делает по умолчанию. При этом вывод будет генерироваться только для _pages_ и корневого файла *_index.md*.

``` toml
disableKinds = ["RSS", "robotsTXT", "404", "taxonomy", "section", "term", "sitemap"]`
```

#### 2. permalinks

**permalinks** - играет важную роль. Здесь указывается путь для каждого раздела - **section**, содержащегося в директории **content/**. По умолчанию - **blog**. Если нужно включить другие или дополнительные разделы (поддиректории **content/**), то для каждого из них должна быть добавлена соответствующая строка.

``` toml
 [permalinks]
  blog = "/OEBPS/chapter/:filename"  
  "/" = "/OEBPS/:filename"`
```


#### 3.  mediaTypes, outputFormats и outputs

**mediaTypes**, **outputFormats** и **outputs** - определяют форматы терх файлов, динамически генерируемых HUGO: **xhtml**, **opf** и **ncx**. Соответствующие шаблоны находятся в директории темы: _layouts/_.

``` toml
[mediaTypes]
  [mediaTypes."application/xhtml+xml"]
    suffixes = ["xhtml"]
  [mediaTypes."application/opf+xml"]
    suffixes = ["opf"]
  [mediaTypes."application/x-dtbncx+xml"]
    suffixes = ["ncx"]
[outputFormats]
  [outputFormats.XHTML]
    mediaType = "application/xhtml+xml"
    isHTML = true
    permalinkable = true
  [outputFormats.OPF]
    mediaType = "application/opf+xml"
    isHTML = true
    permalinkable = true
    path = "OEBPS"
    baseName = "content"
  [outputFormats.NCX]
    mediaType = "application/x-dtbncx+xml"
    isHTML = true
    permalinkable = true
    path = "OEBPS"
    baseName = "toc"
[outputs]
  page = ["XHTML"]
  home = ["OPF","NCX"]
```

#### 4. markup

**markup** управляет интерпретацией файлов _markdown_. При необходимости настройки можно менять, кроме параметра `xHTML = true`. Это важно!

``` toml
[markup]
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
      typographer = false
    [markup.goldmark.parser]
      attribute = true
      autoHeadingID = false
    [markup.goldmark.renderer]
      hardWraps = false
      unsafe = true
      xHTML = true
```

#### 4. params

**params** - содержит метаданные электронной книги (название книги, автор и пр.). Он также определяет, масштабирования изображений. Пользователи устаревших устройств для чтения электронных книг (например, Kindle ранних выпусков) могут встретить проблемы с файлами jpg

``` toml
[params]
# imgsize = "800x png"
# png works better with older ebook reader
  imgsize = "1200x q70"
# bookid if empty the unix timestamp is taken
# metadata of your ebook
  bookid = ""
  title = "HUGO Epub Theme"
  subtitle = "from weitblick.org" # (optional)
  description = "A simple HUGO theme for creating epub ebooks" # (optional)
  creator = "Klein, Erhard Maria" # format: lastname, firstname
  author = "Erhard Maria Klein" # format: firstname lastname
  publisher = "Weitblick Internetwerkstatt"
  audience = "teenagers, adults"
  subject = ['HUGO', 'Epub', 'Ebook'] # (optional)
```

## Как генерировать файл epub

См. _exampleSite_.  Чтобы создать собственную электронную книгу, вам нужно всего лишь изменить метаданные в _config.toml_ и заменить _Page-Resources_ в папке _content/blog/_ вашим собственным контентом в формате **markdown**. Кроме того, файл  _static/OEBPS/cover.jpg_ следует заменить на файлом обложки публикации.

**Важно помнить требование разработчика:**

>...Do not change file name and format of the cover image.
>
>..._имя файла и формат изображения обложки_ **_не менять_**!

Для создания демонстрационной электронной книги в директории _public_ (пока - в неупакованном виде) необходимо в корневом каталоге проекта HUGO выполнить команду `hugo`. Чтобы получить электронную книгу EPUB, содержимое каталога _public_ необходимо заархивировать в zip-архив с расширением _.epub_.

При этом **важно** чтобы файл _mimetype_ был первым файлом в zip-архиве. Таким образом, вызов zip должен быть таким, чтобы создать электронную книгу с именем _ebook.epub_, расположенную в корневом каталоге HUGO:

``` bash
cd public
zip -rX ../ebook.epub mimetype OEBPS META-INF
```

Чтобы избежать файлового мусора в готовой электронной книге, перед повторным ее созданием удалить директорию _public/_ и файл _zip/epub_. Для среды Linux в тему добавлен скрипт _deploy_ bash, который выполняет все шаги автоматически:

deploy.sh:

``` bash
#!/bin/bash
hugo --cleanDestinationDir
cd public
rm -f ../ebook.epub
zip -rX ../ebook.epub mimetype OEBPS META-INF
```

## Полезно знать

Тема использует [HUGO Page Bundles](https://gohugo.io/content-management/page-bundles/) для автоматического управления **изображениями**. Интегрировать изображения в свои ресурсы, можно с помощью шорткода:

``` hugo
{< image src="имя изображения" caption="заголовок изображения" >}
```

В `src=` достаточно указать лишь имея изображения, при этом требуется чтобы оно было уникальным. В настоящее время тема поддерживает только изображения в формате _.jpg_ и _.png_. Автоматическое определение MIME-типа в манифесте _content.opf_ может нуждаться в некотором улучшении и при необходимости может быть скорректировано в _index.opf_. Кроме того, важно, чтобы в именах всех файлов и директорий не было специальных символов, так как их наличие может привести к ошибкам проверки.

При желании встроить **статические изображения** из директории _static/_, следует учитывать три вещи:

1. Изображения должны находиться в директории _static/OEBPS/_.
2. Каждый файл изображения должен быть указан в манифесте (в шаблоне _index.opf_). В качестве примера можно использовать уже существующую запись для _cover.jpg_.
3. В файлах директории _content_ необходимо указывать относительные пути к изображениям (от корня проекта), например. `![Альтернативный текст](../мое-изображение.jpg)`.

Также можно применить встроенный HUGO shortcode: `![Альтернативный текст]({{rel="Images/мое-изображение"}})`?

Файлы Epub с **ошибками проверки** обычно по-прежнему отображаются устройствами.  Однако лучше, конечно, если электронная книга будет полностью свободна от ошибок.  Настоятельно рекомендуемое программное обеспечение для управления электронными книгами [Calibre](https://calibre-ebook.com/) дает возможность не только [редактировать книги](https://manual.ICALe-ebook.com/de/edit.html) но и проводить поиск ошибок, и отладку. Есть и другие валидаторы:

- [онлайн-валидатор Epub](https://www.ebookit.com/tools/bp/Bo/eBookIt/epub-validator)
- программа проверки pagina EPUB (свободная, с отрытым исходным кодом):
  - [сайт](https://www.pagina.gmbh/produkte/epub-checker/)
  - [GitHub](https://github.com/paginagmbh/EPUB-Checker)

## Установка темы epub

Способы установки темы — с версионным контролем пользовательских каталогов (выше темы). И без него.

### epub for hugo

Создаем новый Hugo сайт:

```sh
$ hugo new site yoursite
```

Из корневой директорий созданного сайта клонируем тему epub в соответствующую директорию:

```sh
$ git clone https://github.com/weitblick/epub.git themes/epub
```

### Тема Hugo ePub maker

Чтобы создать новый проект , с условным именем **ideal-book**, темой **hugo-epub-maker**, со всеми настройками по умолчанию, которые легко изменить, и демонстрационным наполнением пользовательского каталога *content*, необходимо выполнить следующие команды.

```bash
hugo new site ideal-book && cd ideal-book && git init
git submodule add git@github.com:vBuresh/hugo-epub-maker.git themes/hugo-epub-maker
cp -R themes/hugo-epub-maker/exampleSite/* ./ && mv hugo.toml hugo.toml~
hugo server
```

1. Строка 1 — создать новый сайт, перейти в корневой каталог этого сайта и создать в нем новый Git-репозиторий. 
2. Строка 2 — подключить тему **refreshing** как Git-подмодуль 
3. <sup>[1](#fn_1)</sup> Строка 3 — скопировать из каталога **exampleSite** в корневой каталог сайта примеры информационного наполнения (content), и конфигурационный файл (hugo.yaml), а созданный автоматически файл `hugo.toml`, — переименовать в `hugo.toml~`. Если этого не сделать, то сайт сгенерируется неправильно, о чем будет выведено сообщение. В дальнейшем файл `hugo.toml~` можно удалить.
4. Строка 4 — сгенерировать сайт **idealproj**, с темой **ReFreshing**.
---
<sup id="fn_1">1</sup>
При необходимости все действия команд, указанных с строке 3, можно выполнить в любое время. Также можно вручную скопировать файлы в пользовательский каталог `content`, а конфигурационный файл написать самостоятельно, и/или сконвертировать в удобный для пользователя формат (yaml, toml или json).


Лицензия
