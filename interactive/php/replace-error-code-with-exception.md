replace-error-code-with-exception:php

###

1. Найдите все вызовы метода, возвращающего код ошибки, и оберните его в <code>try</code>/<code>catch</code> блоки вместо проверки кода ошибки.

2. Внутри метода  вместо возвращения кода ошибки выбрасывайте исключение.

3. Измените сигнатуру метода так, чтобы она содержала информацию о выбрасываемом исключении (секция <code>@throws</code>).



###

```
class Account {
  // ...
  private $balance;

  /**
   * Withdraw money from account.
   * @param int $amount Amount to withdraw.
   * @return Zero on success, -1 on error.
   */
  public function withdraw($amount) {
    if ($amount > $this->balance) {
      return -1;
    }
    else {
      $this->balance -= $amount;
      return 0;
    }
  }
}

// Somewhere in client code.
if ($account->withdraw($amount) == -1) {
  handleOverdrawn();
}
else {
  doTheUsualThing();
}
```

###

```
class Account {
  // ...
  private $balance;

  /**
   * Withdraw money from account.
   * @param int $amount Amount to withdraw.
   * @throws BalanceException
   */
  public function withdraw($amount) {
    if ($amount > $this->balance) {
      throw new BalanceException();
    }
    $this->balance -= $amount;
  }
}
class BalanceException extends Exception {}

// Somewhere in client code.
try {
  $account->withdraw($amount);
  doTheUsualThing();
} catch (BalanceException $e) {
  handleOverdrawn();
}
```

###

Set step 1

# Рассмотрим рефакторинг на примере метода снятия денег с банковского счета.

Go to "if ($amount > $this->balance) {|||"

#<+ В нашем случае, при попытке снять больше денег, чем есть на счету, генерируется код ошибки (<code>-1</code>)

Select "$account->withdraw($amount) == -1"

#= ...который затем проверяется в клиентском коде.

# Заменим все это выбрасыванием исключения, с последующим «отловом» его в клиентском коде.

Go to after "Account"

# Итак, первым делом можно создать новый класс исключения, который будет легче отлавливать.

Print:
```

class BalanceException extends Exception {}
```

# Затем, обернём код вызова нашего метода в <code>try</code>/<code>catch</code> блоки.

Select:
```
if ($account->withdraw($amount) == -1) {
  handleOverdrawn();
}
else {
  doTheUsualThing();
}
```

Wait 500ms

Print:
```
try {
  $account->withdraw($amount);
  doTheUsualThing();
} catch (BalanceException $e) {
  handleOverdrawn();
}
```

Set step 2

# После этого изменяем метод так, чтобы он выбрасывал исключение, вместо возврата кода ошибки.

Select:
```
      return -1;
```

Wait 500ms

Print:
```
      throw new BalanceException();
```

Wait 500ms

Select:
```
      return 0;

```

Remove selected

Select:
```
|||    else {
|||      $this->balance -= $amount;
|||    }
|||
```

# Этот код можно немного упростить, убрав <code>else</code>.

Remove selected

Select:
```
      $this->balance -= $amount;
```

Deindent

Select name of "Account"

# Неудобство этого шага в том, что мы вынуждены изменить все обращения к методу и сам метод за один шаг, иначе компилятор нас накажет. Если мест вызова много, то придётся выполнять большую модификацию без промежуточных компиляции и тестирования.

# В таких случаях лучше создать новый метод, переместить в него код старого, включив в него исключения. Код старого метода заменить <code>try</code>/<code>catch</code> блоками, которые возвращают коды ошибки. После этого код останется рабочим, а вы можете один за другим заменять обработчики кодов ошибок вызовами нового метода и блоками <code>try</code>/<code>catch</code>.

Set step 3

Select:
```
@return Zero on success, -1 on error.
```
# Как бы то ни было, нам осталось только обновить документацию метода, сообщив, что метод теперь выбрасывает исключение.

Print:
```
@throws BalanceException
```

#C Запускаем финальное тестирование.

#S Отлично, все работает!

Set final step

#Q На этом рефакторинг можно считать оконченным. В завершение, можете посмотреть разницу между старым и новым кодом.