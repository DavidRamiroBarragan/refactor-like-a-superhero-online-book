# Низко-висящие фрукты

Для рефакторинга куска кода могут потребоваться разные изменения, и бывает сложно выбрать, с чего начать. Мне нравится ранжировать изменения от простых к сложным и начинать с простых. Это помогает «сдуть с кода пыль» и начать видеть в нём более серьёзные проблемы.

К простым изменениям я обычно отношу форматирование, ошибки линтера и замену самописного кода на возможности языка или окружения, в котором код будет работать. Об этом сейчас и поговорим.

## Форматирование кода

Форматирование — это вкусовщина, но у него есть одна полезная функция. Если код во всём проекте написан _последовательно_, то у читателей уходит меньше времени на его восприятие.

Так работают привычки: знакомый «рисунок» кода помогает сосредоточиться не на словах и буквах, а на смысле.

```
// Код без форматирования.
// При чтении надо «продираться» сквозь него,
// чтобы увидеть смысл за буквами:

function ProductList({ products }) {
   return <ul>{products.map((product) =><li key={product.name}>
       <Product product={product} /></li>)}</ul>}


// Отформатированный код помогает упростить чтение
// и быстрее перейти к смыслу:

function ProductList({ products }) {
  return (
    <ul>
      {products.map((product) => (
        <li key={product.name}>
          <Product product={product} />
        </li>
      ))}
    </ul>
  );
}
```

Форматирование лучше автоматизировать. В примере выше я использовал Prettier,[^prettier] но конкретный инструмент здесь не так важен, как подход в целом. Если команду не устраивает Prettier, можно выбрать другой форматер и использовать его. Суть в _автоматизации процесса_.

Бывает, что форматер ломает работу кода, например, при неосторожном переносе фрагмента на новую строку:

```
// До форматирования:

function setDiscount(discount) {
  if (user.isVip) order.discount = discount; order.total -= discount
}

// После:

function setDiscount(discount) {
  if (user.isVip) {
    order.discount = discount;
  }

  order.total -= discount;
}
```

Чтобы не пропускать такие ошибки, нам нужны тесты. Если тесты открыты в интерактивном режиме рядом с редактором, мы будем видеть, какие ошибки появились после применения форматирования.

Форматирование можно считать отдельной техникой рефакторинга, поэтому результат можно оформить в виде коммита или даже отдельного PR. Наша задача здесь как можно раньше интегрироваться в основной веткой, чтобы не приходилось разруливать сложные конфликты между форматированием и смысловыми изменениями кода от других разработчиков.

## Линтинг кода

Включив линтер и переведя «предупреждения» в «ошибки», мы можем получить список таких ошибок. Этот список можно использовать как список задач для текущей итерации рефакторинга.

Мне нравится оформлять работу над каждым из правил линтера как отдельный коммит или PR. Например, можно удалить весь неиспользуемый код, оформить это как коммит и перейти к следующей проблеме из списка.

<figure>
  <img src="../images/05-linter.png" width="800">
  <figcaption><em>Линтер подсвечивает неиспользуемый код, который можно удалить</em><br><br></figcaption>
</figure>

Если ошибок после включения линтера очень много, то можно включать не все правила сразу, а по одному. Чем мельче будут шаги, тем проще распилить задачу на несколько и решить каждую отдельно.

После исправления каждого правила потребуется проверить, не сломались ли тесты. В будущем я перестану акцентировать внимание на проверке тестов, чтобы сократить текст. Просто договоримся держать в голове, что мы проверяем, не сломались ли тесты, после _каждого_ изменения.

## Возможности языка

Современные языки программирования развиваются и получают обновления. Особенно это применимо к JavaScript, так как спецификация ES обновляется каждый год.[^proposals]

Иногда в новой версии языка появляются фичи, которыми можно заменить старые самописные функции. Как правило, встроенные конструкции языка компактнее, быстрее, надёжнее и понятнее. Мы можем внедрять новые возможности языка, оглядываясь на требуемую поддержку с помощью, к примеру, Caniuse.[^caniuse]

```js
// Самодельный хелпер для проверки начала строки:
const startsWith = (str, chunk) => str.indexOf(chunk) === 0;
const yup = startsWith("Some String", "So");

// ...Можно заменить нативным методом:
const yup = "Some String".startsWith("So");
```

| К слову 💡                                                                                                                                                                                                    |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Если самописная реализация отличается от нативной, и мы не можем заменить её, то я предпочитаю отметить это в документации. Так будет понятно, почему мы используем свои наработки вместо возможностей языка. |

Удалять код выгодно: чем меньше кода, тем меньше потенциальных точек отказа в работе приложения. В целом, при прочих равных я предпочту отдать большую часть работы языку или окружению, чтобы не писать код самостоятельно. Так обычно получается надёжнее.

## Возможности среды разработки

Вместе с возможностями языка ещё хочется выделить возможности редактора или IDE, с которыми мы работаем. Если в них есть автоматизированные средства рефакторинга — стоит научиться ими пользоваться.

“Rename Symbol”, “Extract into Function” и другие инструменты ускоряют работу и снижают когнитивную нагрузку. Например, в VS Code можно изменить имя функции или переменной во всех местах использования сочетанием горячих клавиш:[^vscode]

<figure>
  <img src="../images/05-rename-symbol.png" width="800">
  <figcaption><em>“Rename Symbol” обновляет название сразу и везде</em><br><br></figcaption>
</figure>

Однако, результат применения этих инструментов стоит перепроверять. Например, Rename Symbol может «не заметить» какое-то имя или добавить лишнее переименование:

```tsx
// Например, мы хотим заменить поле `name`
// в типе `AccountProps` на `firstName`:

type AccountProps = { name: string };
const Account = ({ name }: AccountProps) => <>{name}</>;

// После применения Rename Symbol
// может остаться «лишнее переименование»:

type AccountProps = { firstName: string };
const Account = ({ firstName: name }: AccountProps) => <>{name}</>;
```

Чтобы этого избежать я пробегаюсь по диффу изменений с последнего коммита и проверяю, что именно переименовалось и как.

<figure>
  <img src="../images/05-git-diff.png" width="800">
  <figcaption><em>Гит показывает, что именно поменялось с последнего коммита</em><br><br></figcaption>
</figure>

Упростить и максимизировать пользу от такого сравнения помогает стратегия маленьких шагов, о которой мы говорили ранее. Если применять лишь одну технику рефакторинга за коммит, в диффах не будет шума и будет лучше видно, как именно изменения повлияют на код

Линтеры и тесты при этом помогут избежать конфликтов имён и других ошибок. Например, мы можем настроить правила, которые запретят одинаковые имена переменных, и тогда при конфликте имён линтер будет падать с ошибкой. Если он запущен параллельно с редактором, то мы это сразу же увидим и сможем исправить.

[^prettier]: Prettier, an opinionated code formatter, https://prettier.io
[^proposals]: List of EcmaScript Proposals, https://proposals.es
[^caniuse]: Can I Use, support tables for web, https://caniuse.com
[^vscode]: Refactoring Source Code in VSCode, https://code.visualstudio.com/docs/editor/refactoring
