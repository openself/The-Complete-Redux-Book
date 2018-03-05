# Перевод книги [The Complete Redux Book](https://leanpub.com/redux-book) от Ilya Gelman и Boris Dinkevich

## Часть 1. Введение в Redux

### Глава 1. Основные понятия Flux и Redux

Пенициллин, рентген и кардиостимулятор являются известными примерами [непреднамеренных открытий](http://www.businessinsider.com/these-10-inventions-were-made-by-mistake-2010-11?op=1&IR=T). Redux, так же, не должен был стать библиотекой, но оказался отличной реализацией Flux. В мае 2015 года, один из его авторов, [Dan Abramov](https://survivejs.com/blog/redux-interview/), выступал на конференции ReactEurope с докладом о "hot reloading and time travel" (горячей перезагрузке и путешествии во времени). Он признает, что в тот момент понятия не имел, как осуществить time travel. С некоторой помощью [Andrew Clark](https://twitter.com/acdlite) и вдохновленный некоторыми изящными идеями из языка [Elm](http://elm-lang.org/), Dan, в конце концов, придумал очень приятную архитектуру. Когда люди начали понимать это, он принял решение распространять это как библиотеку.

Менее чем за пол года эта маленькая (всего лишь 2KB) библиотека стала основной для React разработчиков, в связи с её крохотным размером, легко читаемым кодом и очень простыми, но аккуратными идеями, с которыми было намного легче справиться, чем с конкурирующей реализаций Flux. Фактически, Redux не является Flux библиотекой, хотя она развилась от идей архитектуры Flux в Facebook. Официальное определение [Redux](https://redux.js.org/) - "_Предсказуемый контейнер состояния для JavaScript приложений_". Это простая идея, которая означает, что вы сохраняете все состояние своего приложения в одном месте и можете узнать какое состояние находится у приложения в данный момент времени.

#### Что такое Flux?

Перед погружением в Redux? мы должны должны ознакомиться с его базой и предшественником - _Flux архитектурой_. "Flux" - это общая архитеутура или паттерн, а не конкретная реализация. Эти идеи были впервые представлены публично Bill Fisher и Jing Chen на конференции Facebook F8 в апреле 2014 года. Flux описывали как переопределение предыдущих идей MVC (Mode-View-Model) и MVVM (Model-View-View-Model) паттернов двусторонней привязки данных, применяемых в других фреймворках. Он предлогал новый паттерн потока событий на frontend стороне - _однонаправленный поток данных._

События во Flux регулируются по одному за раз в круговом потоке с рядом участников: dispatcher, stores, и actions. _Action_ - это структура, описывающая любые изменения в системе: клик мышью, события таймера, Ajax запрос и другие. Action'ы передаются в _dispatcher_ - единственное место в системе, где кто-угодно может отправить action на обработку. Состояние приложения затем сохраняется в _stores_, которые содержат части состояния приложения, и реагируют на команды из dispatcher
 
Это простейший Flux поток:
1. Stores подписываются на подмножество действий.
2. Action отправляется в dispatcher.
3. Dispatcher уведомляет подписанные stores о action.
4. Stores обновляют свое состояние, основываясь на action.
5. Отображаемый контент обновляется в соответствии с новым состоянием в stores.
6. Можно обрабатывать следующий action.

![Flux поток](./source/img/flux-flow.png)

Этот поток обеспечивает легкое понимание того, как actions протекают в системе, что будет причиной изменения состояния и как оно изменится.

Рассмотрим пример приложения на jQuery или AngularJS. Клик по кнопке может привести к вызову нескольких обратных вызовов (callbacks), каждый из которых обрабатывает раличные части системы, что может, в свою очередь, вызвыать обновление в других местах. В этом случае разработчику практически невозможно узнать, как одно событие может изменить состояние приложения и в каком порядке произойдут эти изменения.

Во Flux событие click генерирует один action, который будет изменять store, а затем и отображение. Любые действия, созданные store или другими компонентами во время этого процесса, будут поставлены в очередь и выполняться только после выполнения первого действия и обновления отображения.

Разработчики Facebook изначально не выкладывали в общий доступ их реализацию Flux, а скорее, выложили только её части, например dispatcher. Это привело к тому, что сообщество создало множество реализаций с открытым исходным кодом, некоторые из которых существенно отличаются друг от друга, а некоторые лишь слегка меняют исходные шаблоны. Например, некоторые имеют множество dispatchers, другие вводят зависимости между sores

![Заметка](./source/img/note.png) У Дмитрия Воронянского есть хорошее сравнение различных реализаций Flux на [GitHub](https://github.com/voronianski/flux-comparison).

#### Redux и Flux

Хотя и Redux происходит от концепций Flux, между этими двумя архитектурами существует несколько различий. В отличии от Flux, Redux имееет только один store, который не содержит в себе никакой логики. Actions отправляются непосредственно в store и обрабатываются им же, что убирает необходимость в отдельном dispatcher. Store, в свою очередь, передает actions в функции, которые управляют изменениям состояния. Эти функции называются _reducers_ - новый тип участника потока, добавленный в Redux.

![Reducer пришел на смену Dispatcher](./source/img/dispatcher-reducer.png)

Для лучшего понимания Redux давайте представим приложение, который поможет нам управлять кникой рецептов. В Redux _store_ будет сохранена сама книга рецептов в виде списка рецептов и его деталей.

Приложение позволит нам выполнять различные действия, такие как добавление рецепта, добавление ингредиентов в рецепт, изменение количества ингредиента и другие. Для начала мы можем создать ряд _сервисов_, каждый из которых будет знать как обрабатывать группу из Actions. Например _book service_ будет обрабатывать Actions, которые отвечают за добавление и удаление рецептов. _recipe service_ будет изменять информацию о рецепте, а _recipe-ingredients service_ сможет обрабатывать те actions, которые отвечают за изменение ингридиентов в рецепте. Это позволит нам лучше разделить наш код и, в будущем, с легкостью добавлять поддержку большего количества actions.

Для того, что бы это заработало, наш sore должен связаться с каждым из сервисов и передать им два параметра: текущую книгу рецептоа и действие, которые мы хотим выполнить. Каждый сервис, в свою очередь, как-то изменяет книгу рецептов в зависимости от Actions, которые он умеет обрабатывать. Зачем отправлять action во все сервисы? Возможно, некоторые actions могут влиять более чем на один сервис. Например, смена измерения с грамм на унции приведет к тому, что сервис, отвечающий за ингридиенты, общий вес, а сервис, отвечающий за информацию о рецепте, пометит что рецепт использует имперские измерения. В Redux такие сервисы называют _reducers_.

Возможно мы захотим добавить еще один слой - _middleware_. Каждый action сперва будет пропущен через список middleware. В отличии от reducers, middleware могут модифицировать, останавливать или добавлять больше actions. Как пример можно приветси: логгирующую middleware, авторизациооную middleware, которая проверяет имеет ли пользователь права для выполнения действия или API middleware, которая отправляет что-то на сервер.

Этот простой пример показывает основу Redux. Мы имеем единсвенный store, который контролирует состояние. Actions для описания изменений, которые мы хотим произвести. Reducers (сервисы из прошлого примера), который знает как изменить состояние основываясь на запрошенном action, и middleware для обработки личных задач.

Redux делает особенным то, что reducers никогда не должены изменять state (в нашем случае книгу рецептов), поскольку он неизменный (immutable). Вместо этого reducers должны создать новую копию книги, сделать необходимые изменения в копии и вернуть новую, модифицированную книгу в store. Этот подход позволяет Redux и отображаемой части приложения с логкостью отслеживать изменения. В последующих главах мы подробней обсудим почему и как надо использовать этот подход.

Важно отметить, что все состояние приложения хранится в одном месте, в store. Наличие единого источника данных дает огромные преимущества при отладке, сериализации и разработке, что станет очевидным в примерах в этой книге.

![Redux поток](./source/img/redux-flow.png)

#### Redux Терминология

**Actions и Action Creators**

Единственный способ для приложения изменить состояние - это обработать actions. В большинстве случаев actions в Redux - это не что иное, как простые объекты JavaScript, переданные в store, которые содержат всю информацию, для того, что бы store смог изменить состояние:

Пример объекта action
```
{
  type: 'INCREMENT',
  payload: {
    counterId: 'main',
    amount: -10
  }
}
```

Поскольку эти объекты могут иметь некоторую логику и использоваться в нескольких местах приложения, они обычно обернуты функцией, которая может генерировать объекты на основе параметра:

Функция, которая создает объект action
```
function incrementAction(counterId, amount) {
  return {
    type: 'INCREMENT',
    payload: {
      counterId,
      amount
    }
  };
};
```

Поскольку эти функции создают объект action они названы _action creators_

**Reducers**

Как только действие попадает в store, он должен выяснить как изменить состояние. Для этого он вызывает функцию, передавая ей текущее состояние и полученный action:

Функция, которая высчитывает следующее состояние
```
function calculateNextState(currentState, action) {
  ...
  return nextState;
}
```
Эта функция называется _reducer_. В реальных Redux приложениях существует всего один _корневой reducer_ (root reduce) - функция, которая будет вызывать дополнительные функции reducer для вычесления вложенного состояния:

Пример реализации reducer
```
function rootReducer(state, action) {
  switch (action.type) {

    case 'INCREMENT':
      return { ...state, counter: state.counter + action.payload.amount };

    default:
      return state;
  }
}
```

![Заметка](./source/img/note.png) Reducer никогда не модифицирует state. Он всегда возврщает новую копию c необходимыми изменениями.

**Middleware**

Middleware - это более продвинутая особенность Redux и будет подробно обсуждена в последующих главах. Middleware действуют как перехватчики для действий, прежде чем они достигнут store: они могут модифицировать action, создать больше actions, сдерживать их и многое другое. Поскольку Middleware имеет доступ к actions, функции dispatch() и к store, они являются наиболее гибкими и мощными сущностями в Redux.

**Store**

В отличии от многих других реализаций Flux, Redux имеет только один store, который содержит в себе информацию о приложении, но не содержит пользовательскую логику. Роль store заключается в получении actions, передачи их через все зарегестрированные middleware, а затем исползовать reducers для расчета нового состояния и сохранения его.

Когда он получает action, которое вызывает изменение состояния, store уведомит всех зарегистрированных слушателей о том, что было произведено изменение состояния. Это позволит различным частям системы, таким как пользовательский интерфейс, обновить себя в соответствии с новым состоянием.