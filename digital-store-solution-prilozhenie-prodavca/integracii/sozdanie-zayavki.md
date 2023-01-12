# Создание заявки

{% hint style="danger" %}
Мы работаем только через **POST**-запросы
{% endhint %}

## Подключение

{% hint style="info" %}
Для подключения обратитесь к личному менеджеру в IMSHOP.IO\
Вам необходимо получить:

* URL для подключения
* API ключ
{% endhint %}

{% hint style="info" %}
Ключ передается как `Bearer` токен в HTTP заголовке `Authorization`
{% endhint %}

{% hint style="info" %}
Запросы передаются методом `POST`c типом данных `application/json` в теле запроса&#x20;
{% endhint %}

## Поддерживаемые типы заявок

* Сборка товара
* Передача заказа в доставку / передача в другой магазин
* Прием заказа в магазин для последующей выдачи
* Отмена заявки

## Запрос

### Описание формата

* **`type`** - тип заявки. Возможные значения:
  * **`prepare`** -  сборка заказа
  * **`ship`** - передача заказа в доставку
  * **`receive`** - получение заказа для выдачи в магазине
  * **`release`** - расформировать собранный заказ
  * **`cancel`** - отмена ранее созданной заявки
* **`shouldBeAcceptedInTimeout`** - таймаут на принятие заявки сотрудником
* **`steps`** - список шагов / подзадач
  * **`id`** - идентификатор
  * **`title`** - название
  * **`description`** - описание, что необходимо сделать. этот текст увидит продавец.
  * **`cta`** - текст кнопки "далее"
  * **`displayCart`** - отображать состав заказа на данном шаге
  * **`scanToConfirm`** - запрашивать сканирование каждого товара для подтверждения (**`true`** / **`false`**)
  * **`datamatrix`** - сканировать на данном шаге datamatrix коды для каждого товара (**`true`** / **`false`**)
  * **`timeout`** - максимальное время в секундах на завершение
  * **`scanPackage`** - требуется сканирование кода с пакета, сверяем с номером заказа
* **`ticketId`** - идентификатор ранее созданной заявки (только для запросов **`cancel`** на отмену заявки)
* **`orderId`** - номер заказа из системы управления заказами
* **`storeId`** - идентификатор магазина, в который направляется заявка
* **`staffUserId`** - (опционально) идентификатор сотрудника, который увидит заявку; для остальных она будет скрыта; должен соответствовать **`<user id="..."`** фида сотрудников
* **`staffGroupId`** - (опционально) идентификатор группы сотрудников, которая увидит заявку; для остальных она будет скрыта; должен соответствовать `<user groupId="..."` фида сотрудников
* **`buyer`** - информация о покупателе
  * **`name`** - имя
  * **`phone`** - телефон
  * **`email`** - электронная почта (опционально)
* **`delivery`** (только для заявок типа ship и prepare, если заказ надо сразу передать в доставку после сборки)
  * **`type`** - тип доставки
    * **`regular`** - обычная
    * **`express`** - экспресс или день в день
    * **`store`** - передача в другой магазин
    * **`custom`** - собственный тип доставки. при custom доставке надо передать отдельное поле **`customType`** с идентификатором доставки
  * **`provider`** - идентификатор службы доставки (для всех типов кроме store)
  * **`address`** - адрес доставки адрес магазина&#x20;
    * **`region`** - регион / область
    * **`city`** - город
    * **`settlement`** - поселение
    * **`postalCode`** - индекс
    * **`street`** - улица
    * **`house`** - дом
    * **`block`** - корпус / строение
    * **`apt`** - квартира
    * **`fiasId`** - идентификатор из ФИАС
* **`cart`** - состав заказа
  * **`configurationId`** - идентификатор торгового предложения
  * **`quantity`** - количество
  * **`price`** - цена продажи за 1шт
  * **`datamatrixRequired`** - запрашивать datamatrix для этого товара (**`true`** / **`false`**)
  * **`discount`** - скидка на всю позицию
  * **`subtotal`** - итого за позицию, с учетом скидки и количества
  * **`discountRules`** - описание примененных акций и скидок
    * **`id`** - идентификатор акции
    * **`description`** - описание
* **`bonuses`** - информация о бонусных баллах (опционально)
  * **`canSpend`** - можно потратить на заказ
  * **`willBeSpent`** - разрашено к списыванию / будет списано при выдаче заказа
  * **`willEarn`** - будет начислено
* **`promocode`** - примененный промокод
* **`price`** - цена&#x20;
* **`discount`** - примененная скидка на заказ
* **`deliveryPrice`** - оплачиваемая покупателем стоимость доставки
* **`finalPrice`** - цена итого
* **`payment`** - информация об оплате заказа
  * **`paid`** - флаг оплачен ли заказ
  * **`method`** - способ оплаты
  * **`comment`** - любой комментарий к оплате, например последние 4 цифры карты, чтобы продавец мог верифицировать личность человека, получающего оплаченный заказ
* **`comment`** - любой комментарий

### Пример запроса

```javascript
{
    "type": "prepare",
    "shouldBeAcceptedInTimeout": 900,
    "steps": [
        {
            "id": "prepare-order",
            "title": "Сборка заказа",
            "cta": "Завершить сборку",
            "displayCart": true,
            "scanToConfirm": true,
            "datamatrix": true,
            "timeout": 1200
        },
        {
            "id": "pack-order",
            "cta": "Завершить упаковку",
            "title": "Упаковка"
        }
    ],
    "orderId": "a-12345",
    "storeId": "16784",
    "staffUserId": "0001234",
    "staffGroupId": "1234000",
    "buyer": {
        "name": "Петр Иванов",
        "phone": "+79997771234",
        "email": "some@mail.com"
    },
    "delivery": {
        "type": "regular",
        "provider": "cdek",
        "address": {
            "region": "Москва",
            "city": "Москва",
            "settlement": null,
            "postalCode": "125080",
            "street": "Волоколамское шоссе",
            "house": "2",
            "block": null,
            "apt": null,
            "fiasId": "6cd79dbd-375d-4aaa-ad66-75200adac327"
        }
    },
    "cart": [{
        "configurationId": "63a8dd3e-ce7a-41f9-8572-0f9236234918",
        "quantity": 2,
        "price": 4999,
        "discount": 1000,
        "datamatrixRequired": true,
        "subtotal": 8998,
        "discountRules": [
            {"id": "1c_first_purchase", "description": "Скидка 1000 руб на первую покупку онлайн"}
        ]
    }],
    "promocode": "FIRST1000",
    "price": 9998,
    "discount": 1000,
    "deliveryPrice": 0,
    "finalPrice": 8998,
    "payment": {
        "paid": true,
        "method": "card",
        "comment": "Последние 4 цифры карты: 1234"
    },
    "comment": "Комментарий"
}
```

## Ответ

Сервер ответит следующим HTTP статусом

* **`200`** если заявка принята
* **`400`** если допущена ошибка в запросе
* **`403`** если указан невалидный токен авторизации
* **`500`** в случае другой ошибки

### Формата ответа

* **`success`** - флаг успеха
* **`ticketId`** - идентификатор созданной / отмененной заявки
* **`status`** - статус заявки (**`created`** / **`cancelled`**)
* **`errorCode`** - код ошибки, в сучае если заявка не была создана
* **`errorMessage`** - описание ошибки, если заявка не была создана

### Пример ответа

```javascript
{
    "success": true,
    "status": "created",
    "ticketId": "rsp-1746761"
}
```