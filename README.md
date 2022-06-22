## Окружение

### Установка

```
git clone git@github.com:MorozovP/api_yamdb_final.git
cd api_yamdb

python3 -m venv venv
source venv/bin/activate

python3 -m pip install --upgrade pip
pip3 install -r requirements.txt
pip3 install django-filter
pip3 install djangorestframework-simplejwt

cd api_yamdb
python manage.py migrate
python manage.py import_csv

python3 manage.py runserver
```

### Линтеры

```
pip3 install pytest flake8 pep8-naming flake8-broken-line flake8-return flake8-isort mypy black isort
```

#### Pre-commit hook `.git/hooks/pre-commit`

- Файл должен быть исполняемым: `chmod +x .git/hooks/pre-commit`
- Если `mypy` ругается на внутренние импорты Django, а все остальное правильно,
  можно сделать коммит с ключом `--no-verify`
  (нежелательно): `git commit --no-verify -m 'commit message'`

```
#!/bin/bash

for file in $(git diff --diff-filter=d --cached --name-only | grep -E '\.py$')
do
  git show ":$file" | black -l 79 "$file"
  
  git show ":$file" | isort -e -m 4 -w 79 --profile black "$file"
  
  git show ":$file" | flake8 --format='%(path)s:%(row)d,%(col)d:%(code)s:%(text)s:https://lintlyci.github.io/Flake8Rules/rules/%(code)s.html' "$file"
  if [ $? -ne 0 ]; then
    echo "flake8 failed on '$file'."
    exit 1
  fi

  git show ":$file" | mypy "$file"
  if [ $? -ne 0 ]; then
    echo "mypy failed on '$file'."
    exit 1
  fi
done
```

### Прочее

- Кавычки: двойные!
- Ширина строки: 79 символов

## Процесс работы

### Пул-реквесты

1. В разделе **Pull requests** перейти по
   ссылке [New Pull request](https://github.com/bitbybit/api_yamdb/compare)
2. В меню **base** выбрать `master`
3. В меню **compare** выбрать ветку, в которой выполнена задача
4. Нажать **Create pull request**
5. На странице пул реквеста справа сверху в блоке **Reviewers** добавить двух
   ревьюверов (cross-review)
6. В название пул реквеста скопировать название задачи (если задача выполнена
   частично, то вкратце перечислить, что сделано)
7. На странице таска справа в блоке **Power-Ups** - **GitHub** кликнуть
   **Attach Pull Request**, выбрать созданный пулл реквест
   (Trello сам их слинкует)

### Таски

[Доска Trello](https://trello.com/b/kRfvsbX6/apiyamdb)

### Трекинг времени

В таске в блоке Time можно начать отсчет таймера в режиме реального времени или
указать затраченное на задачу время после работы.

## Проект

### Схема БД

[dbdiagram.io](https://dbdiagram.io/d/6255ba562514c979031aa7f4)

### ТЗ

#### API

[http://127.0.0.1:8000/redoc/](http://127.0.0.1:8000/redoc/)

- Ресурс auth: аутентификация.
- Ресурс users: пользователи.
- Ресурс titles: произведения, к которым пишут отзывы (определённый фильм,
  книга или песенка).
- Ресурс categories: категории (типы) произведений («Фильмы», «Книги»,
  «Музыка»).
- Ресурс genres: жанры произведений. Одно произведение может быть привязано к
  нескольким жанрам.
- Ресурс reviews: отзывы на произведения. Отзыв привязан к определённому
  произведению.
- Ресурс comments: комментарии к отзывам. Комментарий привязан к определённому
  отзыву.

#### Общее описание

Проект YaMDb собирает отзывы (Review) пользователей на произведения (Titles).
Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список
категорий (Category) может быть расширен администратором (например, можно
добавить категорию «Изобразительное искусство» или «Ювелирка»).

В каждой категории есть произведения: книги, фильмы или музыка. Например, в
категории «Книги» могут быть произведения «Винни-Пух и все-все-все» и
«Марсианские хроники», а в категории «Музыка» — песня «Давеча» группы
«Насекомые» и вторая сюита Баха.

Произведению может быть присвоен жанр (Genre) из списка предустановленных (
например, «Сказка», «Рок» или «Артхаус»). Новые жанры может создавать только
администратор.

Благодарные или возмущённые пользователи оставляют к произведениям текстовые
отзывы (Review) и ставят произведению оценку в диапазоне от одного до десяти (
целое число); из пользовательских оценок формируется усреднённая оценка
произведения — рейтинг (целое число). На одно произведение пользователь может
оставить только один отзыв.

---

- Аноним — может просматривать описания произведений, читать отзывы и
  комментарии.
- Аутентифицированный пользователь (user) — может читать всё, как и Аноним,
  может публиковать отзывы и ставить оценки произведениям (
  фильмам/книгам/песенкам), может комментировать отзывы; может редактировать и
  удалять свои отзывы и комментарии, редактировать свои оценки произведений.
  Эта роль присваивается по умолчанию каждому новому пользователю.
- Модератор (moderator) — те же права, что и у Аутентифицированного
  пользователя, плюс право удалять и редактировать любые отзывы и комментарии.
- Администратор (admin) — полные права на управление всем контентом проекта.
  Может создавать и удалять произведения, категории и жанры. Может назначать
  роли пользователям.
- Суперюзер Django должен всегда обладать правами администратора, пользователя
  с правами admin. Даже если изменить пользовательскую роль суперюзера — это не
  лишит его прав администратора. Суперюзер — всегда администратор, но
  администратор — не обязательно суперюзер.

---

- При удалении объекта пользователя User должны удаляться все отзывы и
  комментарии этого пользователя (вместе с оценками-рейтингами).
- При удалении объекта произведения Title должны удаляться все отзывы к этому
  произведению и комментарии к ним.
- При удалении объекта отзыва Review должны быть удалены все комментарии к
  этому отзыву.
- При удалении объекта категории Category не нужно удалять связанные с этой
  категорией произведения.
- При удалении объекта жанра Genre не нужно удалять связанные с этим жанром
  произведения.
