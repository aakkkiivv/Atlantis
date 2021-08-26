# Общие положения

# Версионирование

## Что это такое?

Заранее определенная методика назначения версий для разрабатываемых пакетов, модулей и т.п. Почти во всех случаях используется semver. Обязательно к прочтению: https://semver.org/lang/ru/

## Для чего?

В-нулевых, глядя на версию, можно с легкостью определить характер изменений.
Во-первых, чтобы участники могли получать то состояние системы, которое требуется. Это удобно как для тестирования, так и для разворачивания систем.
Во-вторых, это позволяет контролировать законченность реализуемых фич и принимать решение о публикации наработок.
В-третьих, версии формируют своеобразный таймлайн, поверх которого собирается changelog.

## Как пользоваться?

**Все действия ниже автоматизированны с помощью воркфлоу на гитхабе**

Новые версии создаются с помощью `yarn version`. См. https://yarnpkg.com/cli/version

Сначала всегда идет version воркфлоу. Он проходит по всем измененным воркспейсам и подготавливает их версию к инкрементации:
`yarn workspaces changed foreach --no-private --verbose version patch --deferred`

**Что за команда?**

[yarn workspaces changed](https://github.com/atls/tools/blob/master/yarn/plugin-workspaces/src/workspaces-changed-foreach.command.ts), остальное в доке `yarn version`

После создается **релиз**

Релиз - это publish воркфлоу. Он в свою очередь прогоняет следующий цикл:

**Только для измененных воркспейсов:**

1. [yarn version apply](https://yarnpkg.com/cli/version/apply) - инкрементация версии для "подготовленных" воркспейсов из version
2. npm publish - запушит новую версию пакетов в npm регистр (если они не приватные)
3. Commit в мастер с обновленной версией пакетов

## С какой версии начинать новые пакеты? (см. монорепо)

Новый пакет всегда начинается с 0.0.0. Не 0.0.1, не 1.0.0. Только 0.0.0. `yarn version` сам потом назначит нужную версию, если возникнет такая необходимость.

# Структура репозиториев

В проектах используется монорепо. Т.е. бекенд, фронтенд, девопс и прочее, все находится одном репозитории. Разбивка выполнена в корневой части:

- backend
- devops
- frontend
  и т.д.

### Для чего?

[Доклад Яндекс про ведение разработки в монорепозитории](https://habr.com/ru/company/yandex/blog/469021/)

# Процесс работы с кодом

## Работа с коммитами

- [Conventional Commits](https://www.conventionalcommits.org/ru/v1.0.0-beta.4/)

### Как делать коммиты?

В проектах используется инструмент для создания коммитов, которые соответствуют commit conventional.

Команда для создания коммита: `[yarn commit](https://github.com/commitizen/cz-cli)`. Запустится визард, в котором пошагово нужно будет заполнить необходимые параметры.

Скоупы выбираются в зависимости от модуля и места, в котором были внесены изменения.

Типы веток указываются в соотвествии с типом изменений. Новая фича - feat, фикс - fix, настройка окружения, обновление пакетов - chore. И т.д.

## Работа с пулл реквестами (PR)

### Как создавать?

[Creating a pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request)

### Когда создавать?

Можно сразу после первых коммитов. Для WIP это обязательно. Это позволит следить за процессом и вовремя вносить корректировки.

### Кого ставить в assignee?

Указываем себя, как автора. Это позволит потом по PR делать фильтрацию и получает предсказуемые результаты.

### Кого указывать в ревьюверах?

Лида, или, если таковой отсутствует - @torinasakura

### Что за статусы проверок?

У проектов может быть настроены проверки проектов, которые выполняются при создании и обновлении PR. В них можно получить всю необходимую информацию о выполнении тестов.

### Можно ли форсить ветку?

Можно, но только если от этого больше пользы, чем вреда.
Когда можно:

1. Когда ревью еще не провели.
2. Когда в ветку еще ничего не вливали (обновленный dev)
3. Когда PR на этой ветке еще нет.

Когда нельзя:

1. Когда ревью провели. Комментарии привязываются к хешу коммитов. Если форснуть, то это уже новые коммиты, даже если в них ничего не менялось. Из-за этого весь прогресс ревью тупо слетает. В итоге ревьюверу нужно потратить время, чтобы определиться, где он оставлял ревью, а где нет.
2. Когда форсом можно ненароком ввести в заблуждение. После форса достаточно муторно посмотреть, что же в итоге изменилось. Можно, конечно, выдрать хеши и пойти в diff, чтобы посмотреть, что изменилось.

Лучше пусть будут лишние коммиты (коммиты лишними не бывают), так процесс выглядит более прозрачным и последовательным.

## Работа с ветками (branch)

### Основной принцип

В целом на таску (issue) заводим одну ветку. В одну ветку можно загнать несколько issue, если они связаны между собой.

### От чего отпочковываться

Все новые ветки создаются от default branch, ей может быть dev - если проект распределённый и имеет два окружения или же master - как в случае с библиотеками и командными утилитами.
Но есть исключение: middlebranch. Если есть задача, в которой неизбежна поломка одной из частей (фронт или бек), то заводится middlebranch. Далее уже от этой ветки формируются ветки для задач и в нее же вливаются наработки через PR. Когда ветка middlebranch укомплектована и работоспособна до такой степени, что фронт или бек не конфликтуют, формируется PR для вливания в dev. Дальнейшие фиксы и фичи готовятся в штатном режиме через dev.

### Обновление веток

Нужно внимательно следить за тем, чтобы ветка вовремя обновлялась (пока задача пилится, dev может измениться). Иначе можно случайно пропустить момент, когда обновления ломают новые наработки.

## Работа с Issues

### Где брать задачи

Следует нажать на Issues в навигационной панели Github и посмотреть задачи там. Если там задач нет, то обратиться к лиду, в крайнем случае к руководителю (@torinasakura)

### Что делать, если ~нет задач~ не знаешь чем заняться

Если такая ситуация возникла, то нужно обратиться к своему лиду, в крайнем случае к руководителю (@torinasakura).

Однако исходя из опыта - такого не может быть, даже если нет прямо-поставленных задач то всегда есть обновление зависимостей, еженедельный дайджест по новым фичам - всегда есть чем заняться на пользу себе и во благо команде.

### Как вести обсуждение задачи или Issue в Github

Что бы обсуждение было наиболее эффективным, следует соблюдать несколько простых правил и рекомендаций:

1. **Используй цитирование.** В Github не предусмотрена система тредов внутри Issue из-за этого в обсуждениях может происходить путаница. Что бы не запутаться мы используем инструмент цитирования — ">". Это позволяет сохранять нить обсуждения и «не распыляться» в обсуждении.
2. **Пожалуйста, старайся писать грамотно.** Ни в коем случае не претендую на великого грамотея и знатока русского языка и кроме этого я не сомневаюсь в личной грамотности членов команды, но кажется что этот пункт должен быть в данном списке.
3. **Старайся четко формулировать вопросы и мысли,** от этого может зависеть качество ответа на вопрос или качество обсуждения идеи. Если спрашивать не очень четко сформулировав вопрос или не очень внятно описав идею, есть высокая вероятность получить такой же невнятный ответ или пустой разговор.
   Очень хороший приём - обязательно перечитать свой текст перед тем как отправлять его.
4. **Используй Markdown.** В дополнении к предыдущему пункту, [советую освоить Markdown.](https://guides.github.com/features/mastering-markdown/) Он позволит твоей мысли быть четкой, ясной и понятной. Такой текст легче воспринимать.
5. **Сдерживай себя, если вам хочется едко (или как говорят - токсично) пошутить в чей-либо адрес.** Процесс совместной работы при помощи Github хотелось бы поставить хорошо и надолго, по этому тут не место. Если очень хочется отчебучить едкую неприличную шутку - это можно сделать в чате или в личном разговоре, Github - для дел и работы.
6. **Придерживайся конвенции.** Вслед за ConventionalCommits вышла следующая, по своей важности, инициатива - [ConventionalComments](https://conventionalcomments.org) - рекомендую прочесть и стараться придерживаться, так как вскоре эта инициатива станет обязательной к использованию как и, в своё время стала инициатива ConventionalCommits.

### Как фильтровать Issues

[Filtering issues and pull requests](https://help.github.com/en/github/managing-your-work-on-github/filtering-issues-and-pull-requests)

### Что за лейблы

Лейблы это метки. Лейблы позволяют эффективно организовать нашу работу и понять с чем каждый из нас работает. Вся необходимая информация о лейблах [лежит здесь.](https://github.com/atlantis-lab/convention/wiki/Labels.-%D0%A0%D0%B0%D0%B1%D0%BE%D1%82%D0%B0-%D1%81-%D0%BB%D0%B5%D0%B9%D0%B1%D0%BB%D0%B0%D0%BC%D0%B8) Но если вкратце то, лейблы состоят из:

- Priority — приоритет
- Scope — скоуп, т.е затрагиваемая «область» issue

### Как понимать приоритеты

Все просто, приоритета у нас два:

- `Critical` — задача выполняется с повышенным приоритетом
- `Blocker` — задача выполняется в неотложном порядке. Все задачи с приоритетом `Critical` идут следом за этими задачами. Blocker - говорит само за себя - задача блокирует дальнейшую работу
