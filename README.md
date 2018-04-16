# calc
Калькулятор, работающий на Обратной польской нотации.

Основной класс - `Expression`.
Он является неизменяемым и представляет скомпилированное арифметическое выражение.
В выражении поддерживаются следующие операции и функции:
* `+` -- сложение
* `-`, `−` -- вычитание
* `*`, [`∙`](https://unicode-table.com/2219), [`∗`](https://unicode-table.com/2217), [`×`](https://unicode-table.com/00D7), [`✕`](https://unicode-table.com/2715) -- умножение
* `/`, [`÷`](https://unicode-table.com/00F7), [`∕`](https://unicode-table.com/2215) -- деление
* `^` -- возведение в степень
* `!` -- факториал
* `%` -- перевод в проценты
* [`√`](https://unicode-table.com/221A), `sqrt` -- радикал (квадратный корень)
* `log` -- логирифм по основанию 10
* `ln` -- натуральный логарифм
* `exp` -- экспонента
* `abs` -- абсолютная величина
* `sgn` -- сигнум
* `sin` -- синус
* `cos` -- косинус
* `tan`, `tg` -- тангенс
* `cot`, `ctg` -- котангенс
* `arcsin` -- арксинус
* `arccos` -- арккосинус
* `arctan`, `arctg` -- арктангенс
* `arccot`, `arcctg` -- арккотангенс
* `sinh`, `sh` -- гиперболический синус
* `cosh`, `ch` -- гиперболический косинус
* `tanh`, `th` -- гиперболический тангенс
* `coth`, `cth` -- гиперболический котангенс

Выражение может содержать в себе именованные константы. Грамматика констант и допустимые символы позаимствованы у языка Java, поэтому все символы, которые могут быть идентификаторами в Java, могут также быть константами в выражении.
Допустимые имена констант:
```
a, b, c, const1
значение1, значение2, высота, ширина
```

Константы не могут иметь те же имена что и функции.
Помимо этого, для выражения зарезервированы математические константы:
* `pi`, [`π`](https://unicode-table.com/03C0), [`𝜋`](https://unicode-table.com/1D70B) -- число Пи;
* `e`, [`𝑒`](https://unicode-table.com/1D452) -- экспонента.

Синтаксис арифметических выражений близок к класическому, но имеет некоторые улучшения.
Символы пробела, переносов строки и табуляции используются как разделители операндов, если
написать два операнда раздельно, это будет эквивалентно операции умножения.
Кроме этого, десятичные дроби можно записывать, опуская ноль перед точкой.
Так, следующие выражения полностью эквивалентны:

```
2a     <==> 2 a <==> 2 * a
sin pi <==> sin(pi)
√-4    <==> √(-4)
0.24   <==> .24
0.     <==> .0 <==> . <==> 0
(2a sin 3)(a b cos c) <==> (2*a*sin(3))*(a*b*cos(c))
```

## Использование
#### Подключение:
```java
import mitasov.calc.Expression;
```
#### Простой пример:

```java
Expression e = new Expression("2+2"); //целые числа
double res1 = e.evaluate(); // == 4

e = new Expression("2.564 * 2"); //дробные
res = e.evaluate(); // == 5.128

e = new Expression("2,5 - 3", ',') //можно указать символ плавающей точки
res = e.evaluate(); // == -0.5
```

#### Пример использования констант

```java
Expression e = new Expression("a+b"); // a и b будут иметь значения null

e.getConstants().put("a", 3);
e.getConstants().put("b", 4);

double res = e.evaluate(); // == 7
```

##### Получение списка констант:

```java
Set<String> names = e.getConstants().names(); //набор имён констант из выражения
Set<Map.Entry<String, Double>> entries = e.getConstants().entrySet(); //набор пар имя=значение
Collection<Double> values = e.getConstants().values(); //коллекция значений


for (String name : names) { //пример итерации по именам
    System.out.println(name); //выведет имя
    System.out.println(e.getConstants().get(name)); //выведет значение
}
```

#### Обработка ошибок

Все исключения, которые может бросить класс `Expression`, наследованы от абстрактного
класса `ExpressionException`. Все исключения делятся на две группы:
* `CompilationException` - ошибки в ходе компиляции (лексические и синтаксические)
* `EvaluationException` - ошибки в ходе вычисления (деление на ноль, отсутствие значения переменной)

Исключения, наследованные от `CompilationException`:
* `InvalidCharacterException`
* `SyntaxException`
    * `OperatorWithoutOperandException`
    * `ParenthesisException`
    * `UnexpectedEndException`

Исключения, наследованные от `EvaluationException`:
* `ConstNotSetException`
* `DivisionByZeroException`
* `WrongArgumentException`

При ошибке в ходе анализа выражения, калькулятор бросает исключение,
производное от `CompileException`.
Объект `CompileException` кроме сообщения содержит информацию о позиции ошибочной
подстроки и о её длине

```java
try {
    Expression e = new Expression("26+*983"); // бросит исключение на символе '*'
} catch (OperatorWithoutOperandException e) {
    System.out.println(e.getMessage());  // выведет "Operator without operand"
    System.out.println(e.getPosition()); // выведет 3
    System.out.println(e.getLength());   // выведет 1
    System.out.println(e.getEndPosition()); // то же что и сумма позиции и длины
}
```

При попытке вычислить выражение, не назначив константам значения, метод
`evaluate()` всегда будет выбрасывать `ConstNotSetException`

В случаях деления на ноль или получения факториала отрицательного числа, метод
`evaluate()` не будет выбрасывать исключения, а будет возвращать `Infinite`
или `NaN`. Чтобы "включить" эти исключения, необходимо вызвать метод `evaluate(true)`:

```java
try {
    Expression e = new Expression("10 / 0");
    double d = e.evaluate(); // бесконечность
    d = e.evaluate(false); // то же самое

    d = e.evaluate(true); // выбросит исключение
} catch (CompileException ignored) {

} catch (DivisionByZeroException e) {
    System.out.println(e.getMessage); // "Division by zero"
} catch (EvaluationException ignored) {

}
```