# История бонусов

{% hint style="danger" %}
Мы работаем только через **POST**-запросы
{% endhint %}

{% hint style="danger" %}
Дополнительные интеграции вводятся в эксплуатацию после завершения основных интеграций:

* [Оформление заказа](broken-reference)
* [Доставки](broken-reference)
* [Оплаты](broken-reference)

Для подключения дополнительных интеграций обратитесь к вашему менеджеру в IMSHOP.IO
{% endhint %}

## Запрос

`APP SERVER → INFRASTRUCTURE`

{% hint style="info" %}
От вас потребуется URL, на который наш сервер будет слать **POST**-запрос. Да, мы _запрашиваем_ данные через _POST_, а не через _GET_.
{% endhint %}

В вашу систему будут приходить вот такие данные:

```javascript
{
    "userIdentifier": "12345"
}
```

* **`userIdentifier`** — идентификатор пользователя (обязательное поле), полученный от системы клиента на этапе авторизации

#### Ответ

```javascript
{
  "bonuses": [
    {
      "added":  250,
      "expired":  100,
      "definitionMessage": "За покупку Машинка для стрижки волос и бороды Philips HC 3510/15 Hairclipper series 3000",
      "activationDate":  "1627391774032",
      "expirationDate": null
    }
  ]
}
```

* **`bonuses`** - список бонусов
  * **`added`** - кол-во добавленных бонусов
  * **`expiredd`** - кол-во сгоревших бонусов
  * **`definitionMessage`** - описание
  * **`activationDate`** - [timestamp](https://www.unixtimestamp.com/) создания бонуса в милисекундах
  * **`expirationDate`** - [timestamp](https://www.unixtimestamp.com/) сгорания бонуса в милисекундах

{% hint style="info" %}
Следует помнить, что этот запрос будет приходить из нашего доверенного, [авторизованного](uchyotnaya-zapis-polzovatelya.-avtorizaciya./avtorizaciya-po-nomeru-telefona-+-sms.md) сервера; это — не публичный API.
{% endhint %}

Любой другой ответ API будет расцениваться как отсутствие истории бонусов.
