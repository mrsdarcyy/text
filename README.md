# 2. Эфемерное состояние #

В мире Flutter-разработки управление состоянием приложения играет ключевую роль. Один из важных аспектов этого процесса — понимание __эфемерного состояния__, поэтому в этой статье я предлагаю разобраться с тем, что это такое, чем эфемерное состояние отличается от глобального, а также __когда__ его следует использовать при разработке приложений.


## Что такое эфемерное состояние? ##
__Эфемерное состояние__ (_от англ. ephemeral state_) — это локальное состояние системы управления, которое живет внутри одного виджета или его непосредственных потомков. Его не требуется сохранять между сессиями или при навигации между экранами, а также оно не влияет на другие части приложения и не зависит от них. Проще говоря, это временные данные, необходимые только для текущего отображения или поведения виджета. Эфемерное состояние не нуждается в глобальном управлении и может быть полностью инкапсулировано внутри виджета.

Примером эфемерного состояния является раскрытие или сворачивание элементов списка: статус "раскрыт/свернут" существует только во время текущего взаимодействия. Если вы обновите страницу, это состояние исчезнет, если оно не сохранено. К другим примерам относятся положение ползунка (`Slider`) и выбранная вкладка в `TabBar`.


## Чем эфемерное состояние отличается от глобального? ##
В отличии от эфемерного, __глобальное состояние__ должно быть доступно во многих частях приложения и, возможно, сохраняться между сессиями. Оно может управляться с помощью стейт-менеджеров и следовать архитектурным паттернам вроде MVC и MVVM.

Ниже добавил небольшую табличку, отображающую основные отличия эфемерного и глобального состояний.
| Признак сравнения | Эфемерное состояние | Эфемерное состояние |
|-------------|-------------|-------------|
| Область действия | ограничено одним виджетом или его непосредственными потомками. | доступно во всём приложении или в значительной его части. |
| Сохранение данных | не сохраняется между сессиями или при навигации. | может сохраняться и восстанавливаться при запуске приложения. |
| Влияние на приложение | не влияет на другие части приложения. | может влиять на различные компоненты и логику приложения.    |


## Когда использовать глобальное состояние? ##
Глобальное состояние следует использовать, когда:
1. __Состояние должно быть доступно в разных частях приложения:__ например, информация о пользователе после авторизации.
2. __Требуется сохранять данные между сессиями:__ настройки приложения, избранные элементы.
3. __Необходимо синхронизировать состояние между виджетами:__ обновление корзины товаров в интернет-магазине.


## Когда использовать эфемерное состояние? ##
Эфемерное состояние зачастую используется для создания интерактивных UI элементов, чья логика отрисовки не являяется напрямую частью бизнес-логики приложения. Я выделил 3 случая, когда эфемерное состояние - наиболее подходящее решение:
1. __Состояние относится только к одному виджету:__ если изменение состояния не должно влиять на другие виджеты или экраны.
2. __Нет необходимости сохранять состояние между сессиями:__ например, временные данные формы до её отправки.
3. __Требуется быстрое и простое решение:__ использование `setState()` для управления локальным состоянием упрощает код и ускоряет разработку.

### Пример: Раскрывающийся список (ExpansionTile) ###
```
import 'package:flutter/material.dart';

class FAQItem extends StatefulWidget {
  final String question;
  final String answer;

  const FAQItem({Key? key, required this.question, required this.answer})
      : super(key: key);

  @override
  _FAQItemState createState() => _FAQItemState();
}

class _FAQItemState extends State<FAQItem> {
  bool _isExpanded = false;

  void _toggleExpansion() {
    setState(() {
      _isExpanded = !_isExpanded;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        GestureDetector(
          onTap: _toggleExpansion,
          child: Text(
            widget.question,
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
        ),
        if (_isExpanded)
          Padding(
            padding: const EdgeInsets.symmetric(vertical: 8.0),
            child: Text(widget.answer),
          ),
      ],
    );
  }
}
```
> Пояснение: __FAQItem__ — это виджет, предназначенный для отображения вопроса и раскрытия ответа при нажатии. Управление состоянием `_isExpanded` осуществляется локально с использованием метода `setState()`, что делает компонент полностью независимым. Благодаря этому виджет можно использовать в любом месте приложения без необходимости дополнительных настроек.


## Как использовать эфемерное состояние во Flutter? ##
Во Flutter для управления эфемерным состоянием используется `StatefulWidget` и метод `setState()`. Это простой и наиболее эффективный способ обновления интерфейса при изменении локального состояния.

### Пример: Поле ввода с проверкой ###
```
class EmailInput extends StatefulWidget {
  const EmailInput({Key? key}) : super(key: key);

  @override
  _EmailInputState createState() => _EmailInputState();
}

class _EmailInputState extends State<EmailInput> {
  final TextEditingController _controller = TextEditingController();
  String? _errorText;

  void _validateEmail(String value) {
    setState(() {
      _errorText = value.contains('@') ? null : 'Введите корректный email';
    });
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      decoration: InputDecoration(
        labelText: 'Email',
        errorText: _errorText,
      ),
      onChanged: _validateEmail,
    );
  }
}
```
> __EmailInput__ — это виджет для ввода email с простой валидацией, где состояние ошибки `_errorText` управляется локально. Глобальное состояние не требуется, так как проверка применяется исключительно внутри этого виджета.

## Заключение ##
Эфемерное состояние — это простой и эффективный способ управлять локальными изменениями в интерфейсе, поэтому стоит научиться использовать его правильно. Оно помогает сосредоточиться на задаче, не утяжеляя код стейт-менеджерами там, где это не нужно. Виджеты с локальным состоянием независимы, изолированы и легко переиспользуются, что особенно важно, при работе над UI-китами или созданием компонентов вроде кнопок, переключателей или карточек. Главное помнить: `setState()` — это инструмент для простого обновления интерфейса.

Эфемерное состояние — отличный способ писать чистый и понятный код, где каждый компонент отвечает только за себя. Но нужно использовать его только там, где это оправдано: не всегда стоит разделять локальное и глобальное состояние, чтобы архитектура оставалась лёгкой и поддерживаемой. Научившись работать с этим подходом, вы будете писать не просто рабочий, а красивый и удобный в использовании код.


## Часто задаваемые вопросы ##
#### Что такое эфемерное состояние и чем оно отличается от глобального? ####
Эфемерное состояние — это локальное состояние виджета, необходимое только для его текущего отображения и не влияющее на другие части приложения. Глобальное состояние доступно во многих частях приложения и может сохраняться между сессиями.

#### Когда следует использовать `setState()`? ####
`setState()` следует использовать для управления эфемерным состоянием внутри `StatefulWidget`, когда изменения состояния касаются только этого виджета.

#### Можно ли использовать стейт-менеджеры для эфемерного состояния? ####
Хотя технически это возможно, использование стейт-менеджеров для локальных состояний избыточно и усложняет код. Лучше использовать `setState()` для простоты и эффективности.

#### Как эфемерное состояние помогает создавать переиспользуемые виджеты? ####
Эфемерное состояние позволяет виджетам управлять своим состоянием независимо, что делает их гибкими и легко интегрируемыми в разные части приложения без дополнительных настроек.

#### Как определить, когда использовать локальное или глобальное состояние? ####
Если состояние относится только к одному виджету и не требуется сохранять или делиться им, используйте локальное состояние. Если состояние должно быть доступно во многих частях приложения или сохраняться между сессиями, используйте глобальное состояние со стейт-менеджером.
