# Расчет корзины, скидок, баллов

{% hint style="danger" %}
Мы работаем только через **POST**-запросы.
{% endhint %}

{% hint style="danger" %}
Дополнительные интеграции вводятся в эксплуатацию после завершения основных интеграций:

* [Оформление заказа](../../osnovnye-integracii/oformlenie-zakaza.md)
* [Доставки](../../osnovnye-integracii/dostavki.md)
* [Оплаты](../../osnovnye-integracii/oplaty.md)

Для подключения дополнительных интеграций обратитесь к вашему менеджеру в IMSHOP.IO
{% endhint %}

IMSHOP.IO позволяет при помощи webhook подключить уже существующую на вашей стороне систему пересчета корзины с учетом акций, предложений, бонусных баллов итд.

{% hint style="info" %}
Пересчет корзины через webhook означает, что для каждого изменения состава корзины IMSHOP.IO будет отправлять данные о заказе в систему клиента и ожидать в ответ данные об итоговой стоимости заказа и примененных маркетинговых акциях.
{% endhint %}

## Формат запроса и примеры

### Пример запроса

```json
{
    "city": "Москва",
    "cityKladr": "7700000000000",
    "fiasCode": "77000000000000000000000",
    "deliveryId": null,
    "paymentId": null,
    "deliveryPickupId": "a1",
    "items": [
        {
          "name": "Тестовый товар 1",
          "id": "00a03026-412a-54fe-a9df-dcf9325f8618",
          "privateId": "3464",
          "configurationId": "3464",
          "price": 3735,
          "quantity": 1,
          "subtotal": 3735,
          "addons": [
            {
              "configurationId": "1234"
            }
          ]
        },
        {
          "name": "Тестовый товар 2",
          "id": "605e0108-dc95-5dab-95a2-7f459da6aade",
          "privateId": "29117",
          "configurationId": "29117",
          "price": 14540,
          "quantity": 1,
          "subtotal": 14540
        },
        {
          "name": "Тестовый товар 3",
          "id": "3ccd380e-1f40-5056-8a7a-ef6e8a9582b5",
          "privateId": "34607",
          "configurationId": "34607",
          "price": 5723,
          "quantity": 2,
          "subtotal": 11446
        },
        {
          "name": "Тестовый товар 4",
          "id": "523036eb-b776-5795-b24b-0f224f2d8b17",
          "privateId": "6527",
          "configurationId": "6527",
          "price": 29336,
          "quantity": 1,
          "subtotal": 29336
        },
        {
          "name": "Кабель питания Baseus",
          "id": "02ebe9c7-20f0-4cf0-8160-396ef2604daa",
          "privateId": "1234567800",
          "configurationId": "1234567800",
          "price": 2000,
          "quantity": 2,
          "appliedDiscounts": ["dd771099"]
        }
    ],
    "externalUserId": "XXXXXXX",
    "promocode": "12345",
    "giftCards": [
      {
        "number": "123",
        "pin": "123",
      },
    ],
    "promoGroup": [
        {
          "id": "dd771099",
          "gifts": [
            {
              "id": "56789",
              "quantity": 1
            }
          ]
        }
    ],
    "installId": "ed6b1eec-082f-4880-9100-bcd8e2016cd1",
    "groupId": "1f3a5ced-2a79-4dda-b432-05b8188475fd",
    "loyaltyCard": "12345",
    "preferredPickupId": null
}
```

### Описание формата запроса

* **`city`** - Стандартизированное название города из системы [ФИАС](https://www.alta.ru/fias/)
* **`cityKladr`** - [КЛАДР](https://www.alta.ru/fias/) города
* **`fiasCode`** - [ФИАС](https://www.alta.ru/fias/) код
* **`deliveryId`** - идентификатор выбранной службы доставки в IMSHOP.IO (**`null`** если доставка еще не выбрана)
* **`paymentId`** - идентификатор выбранной службы оплаты в IMSHOP.IO (**`null`** если оплата еще не выбрана)
* **`deliveryPickupId`** - (опционально) выбранный желаемый пункт выдачи / магазин для получения заказа
* **`preferredPickupId`** - (опционально) id любимого пункта самовывоза (если есть),  выбранный в списке любимых магазинов, например на этапе корзины
* **`items`** - состав корзины
  * **`name`** - наименование
  * **`id`** - идентификатор товара в IMSHOP.IO
  * **`privateId`** - идентификатор товарного предложения в системе клиента
  * **`configurationId`** - идентификатор товарного предложения в IMSHOP.IO
  * **`price`** - цена товара на момент добавления в корзину (если товар - это подарок на выбор, то тут передается **`0`**)
  * **`quantity`** - количество
  * **`subtotal`** - итого по позиции (**`subtotal`** = **`price` ** \* ** `quantity`**)
  * **`appliedDiscounts`** - если предыдущий расчет показал наличие маркетинговой акции, или это - подарок на выбор, то передаем идентификатор акции из прошлого ответа от API
  * **`deliveryGroup`** - (опционально) выбранная группа доставки товара, при разбиениии корзины
  * **`addons`** - (опционально, в разработке) доп. товары, например "Основание для кровати"
    * **`configurationId`** - идентификатор товарного предложения из фида
* **`externalUserId`** - идентификатор покупателя в системе клиента, если известен
* **`promocode`** - промокод, введенный покупателем
* **`giftCards`** - подарочные карты, по одной
  * **`number`** - номер карты
  * **`pin`** – пин-код карты
* **`earlyAccess`** -параметр определяющий что заказ из раннего доступа **`true`**
* **`promoGroup`** - (опционально) группа маркетинговых акций, выбранная покупателем. Поля каждой акции:
  * **`id`** - идентификатор акции
  * **`gifts`** - список товаров, выбранных в качестве подарка. Если акция подразумевает только скидку, это поле не будет заполнено
    * **`id`** - идентификатор товарного предложения
    * **`quantity`** - количество единиц товара
* **`installId`** - идентификатор установки приложения
* **`groupId`** - если вы разделяете корзину пользователя на несколько отправлений, используйте groupId, чтобы корректно рассчитывать итоговую стоимость каждого отправления с учетом скидок на всю корзину целиком. У исходной корзины и у каждой части корзины приходит одинаковое значение groupId
* **`loyaltyCard`** - номер карты лояльности (обычно карта лояльности передаваться не будет, так как она часто уже прикреплена к аккаунту покупателя. используется для анонимных заказов, или если клиент не имеет привязки карт лояльности к профилям пользователей)
* **`customSectionValues`** - индивидуальные секции в оформлении заказа (к примеру добавить открытку, если пользователь приобретает товар в подарок)

{% hint style="info" %}
В полях запроса могут быть переданы идентификаторы даты/времени интервала доставки (**в разработке**)
{% endhint %}

* **`deliveryDateIntervalId`** - (опционально) идентификатор, привязанный к датам, полученный в поле **`dateIntervals[<index>].id`** вебхука доставок
* **`deliveryTimeIntervalId`** - (опционально) идентификатор, привязанный ко времени, полученный в поле **`dateIntervals[<index>].timeIntervals[<index>].id`** вебхука доставок

```json
{
    ...
      "deliveryDateIntervalId": "21-06-10",
      "deliveryTimeIntervalId": "10-15",
    ...
}
```

{% hint style="info" %}
Для маркетплейсов.

В полях запроса для каждого товара могут быть переданы идентификаторы магазина/ООО (**в разработке**).
{% endhint %}

* **`warehouseId`** — (опционально) идентификатор склада/магазина/аутлета/ООО из [фида наличия для маркетплейсов](broken-reference)

```json
{    
    ...
    "items": [
        {
            ...
            "warehouseId": "AF-1416"
            ....
        }
    ]
    ...
}
```

## Формат ответа и примеры

### Описание формата

Бекенд клиента должен ответить на вебхук с **`application/json`** в следующем формате:

* **`totalPrice`** - итоговая цена корзины с учетом всех скидок и МА, **без учета бонусов (Покупатель сам решает сколько потратить бонусов. Максимальное количество бонусов для списание контроллируется полем `bonuses.canSpend`)**
* **`appliedPromocode`** - примененный промокод (**`null`** если промокод не применен, или если введенный промокод невозможно применить)
* **promocodeStatus** -статусы раннего доступа:
  * **`success`** -Код прикреплён
  * **`notFound`** -Неверный код
  * **`alreadyRedeemed`** -Код уже использованен
  * **`notRedeemable`** -Условия применения кода не выполнены
* **`discount`** - суммарная скидка на заказ
* **`showDoNotCallMeCheckbox`** - показывать чекбокс **Не перезванивать**
* **skipPayment** - пропустить этап оплаты при оформлении заказа (опционально)
* **`items`** - новый состав корзины (со скидками и подарками)
  * **`id`** - идентификатор товарного предложения в системе клиента (**`configurationID`**)
  * **`price`** - изначальная цена **одного** товара до скидок
  * **`discount`** - примененная скидка **на всю позицию** (на всё количество)
  * **`quantity`** - количество
  * **`subtotal`** - итого (**`subtotal`** = (**`price`** \* **`quantity`**) - **`discount`**)
  * **`gift`** - (опционально) количество товаров данной позиции в подарок
  * **`appliedDiscounts`** - (опционально) JSON список идентификаторов примененных акций (например `[`**`"a1123767", "a98723677"]`**)
  * **`bonuses`** - (опционально) информация по бонусам для данной позиции:
    * **`canSpend`** - максимальное число баллов для списания на данную позицию
    * **`willEarn`** - максимальное число баллов для накопления на данную позицию
  * **`deliveryGroups`** - (опционально; значения: **`default`**, **`express`**, **`large`**) группы доступных доставок, массив; значение по умолчанию: **`["default", "express", "large"]`** . Каждому из товаров можно назначить одну или несколько групп. По умолчанию будет принадлежать той группе, которая передана **первой** в списке. Пользователь сможет перемещать товар между группами, если их передано несколько (например, если доставка у одного из двух товаров недоступна, отказаться от доставки и забрать оба товара самовывозом).

{% hint style="info" %}
Список поддерживаемых групп отправлений и их порядок отображения в корзине настраивается менеджером
{% endhint %}

* **`error`** - (опционально). Текст ошибки, если есть. Если есть хотя бы одна позиция с ошибкой, оформить такой заказ будет невозможно. Ошибка выводится в корзине напротив товара. Пример: "Товар недоступен для заказа"
* **`notice`** - (опционально). Текст сообщения, выводится в корзине около товара, например,  если есть акция 1+1 = 3 на данный товар, чтобы уведомить покупателя.
* **unavailableDeliveryMessage** - (опционально,). Текст сообщения о проблеме с доставкой для этого товара, выводится в корзине около товара, например,  если для товара недоступен самовывоз, чтобы покупатель понял почему у него он не доступен.
* **`addons`** - (опционально,) Доп. товары, например "Основание для кровати"
  * **`configurationId`** - ид товарного предложения из фида&#x20;
* **`warehouseId`** - (опционально) идентификатор склада/магазина/аутлета/ООО из фида наличия для маркетплейсов
* **`bonuses`** - информация по бонусам для данного заказа
  * **`canSpend`** - можно списать бонусов на весь заказ суммарно
  * **`willEarn`** - будет накоплено баллов после заказа
* **`deliveryGroupsBonuses`** - (опционально) информация по бонусам для разделенной корзины (см. пример)
  * **`deliveryGroupId` ** - id группы доставок, представленной сейчас в корзине
    * **`canSpend`** - можно списать бонусов для заказов из группы deliveryGroupId
* **`giftCards`** - подарочные карты, по одной
  * **`number`** - номер карты
  * **`pin`** – пин-код карты
  * **`success`** – возможность привязки карты, boolean
  * **`status`** – комментарий (отображение привязки подарочной карты в корзине, либо сообщение с причиной ошибки)

```json
{
    "totalPrice": 95000,
    "appliedPromocode": null,
    "discount": 1000,
    "skipPayment": false,
    "items": [
        {
            "name": "Apple iPad (2018) 64Gb Wi-Fi, серебристый",
            "id": "12345",
            "price": 29990,
            "discount": 0,
            "quantity": 1,
            "subtotal": 29990,
            "bonuses": { "canSpend": 1990 },
            "addons": [
                {
                    "configurationId": "1234"
                }
            ]
        },
        {
            "name": "Apple Pencil 1-го поколения",
            "id": "22345",
            "price": 8890,
            "discount": 1000,
            "quantity": 1,
            "subtotal": 7890,
            "appliedDiscounts": ["dd76656711"],
            "bonuses": { "canSpend": 0 },
            "unavailableDeliveryMessage": "Для корзины с этим товаром будет недоступен самомывывоз"
        },
        {
            "name": "Чехол Baseus для iPad 2018 черный",
            "id": "32345",
            "price": 2999,
            "discount": 2999,
            "quantity": 1,
            "appliedDiscounts": ["dd1218888"],
            "bonuses": { "canSpend": 0 }
        },
        {
            "name": "Чехол Baseus для iPad 2018 коричневый",
            "id": "42345",
            "price": 1000,
            "discount": 2000,
            "quantity": 3,
            "gift": 2,
            "appliedDiscounts": ["dd771086"],
            "notice": "Товар доступен только на самовывозе",
            "bonuses": { "canSpend": 0 }
        },
        {
            "name": "Чехол Baseus для iPad 2018 черный",
            "id": "87787",
            "price": 1000,
            "discount": 1000,
            "quantity": 1,
            "appliedDiscounts": ["dd771099"],
            "gift": 1,
            "bonuses": { "canSpend": 0 }
        },
        {
            "name": "Apple Mac Pro 2020",
            "id": "80008",
            "price": 5000000,
            "discount": 1000,
            "quantity": 1,
            "bonuses": { "canSpend": 0 },
            "deliveryGroups": ["express"],
            "error": "Позиция отсутствует на складе"
        }
    ],
    "bonuses": {
        "canSpend": 2000,
        "willEarn": 1600
    },
    "deliveryGroupsBonuses": {
        "delivery": {
            "canSpend": 1500
        },
        "pickup": {
            "canSpend": 500
        }
    },
    "giftCards": [
      {
        "number": "123",
        "pin": "123",
        "success": true,
        "status": "Подарочная карта 123 успешно привязана",
      },
      {
        "number": "111",
        "pin": "111",
        "success": false,
        "status": "Невозможно привязать подарочную карту 111",
      },
    ],
}
```



### Маркетинговые акции

Добавляются в стандартный ответ расчета корзины:

* **`promoGroups`** - список групп маркетинговых акций, доступных в корзине на выбор\
  (в каждой группе их может быть несколько)
  * **`id`** - идентификатор акции
  * **`name`** - название акции
  * **`sumsWithBonuses`** - суммируется ли акция с бонусами. Если покупатель выберет акцию с этим флагом в значении**`false`**, ему будет доступна опция **Получить или потратить бонусы** для отказа от участия в акциях.
  * **`gifts`** - список товаров, предлагаемых в виде подарка (не на выбор). Если покупатель примет подарок, то эти товары будут добавлены в корзину.
    * **`id`** - идентификатор товарного предложения
    * **`quantity`** - количество единиц товара
  * **`giftOptions`** - список товаров, предлагаемых в виде подарка на выбор. Если покупатель примет подарок, то товар из этого списка будет добавлен в корзину.
    * **`id`** - идентификатор товарного предложения
    * **`quantity`** - количество единиц товара
  * **`giftOptionsLimit`**- количество вариантов подарков, доступных пользователю на выбор (используется, если список**`giftOptions`** заполнен, **`1`** по умолчанию)
  * **`discount`**- сумма скидки на всю корзину, применяемая при выборе акции (заменяет значение **`discount`** основного тела ответа и не суммируется с ним)

```json
{
    ...
    "promoGroups": [
      [
        {
          "id": "dd76656711",
          "name": "Скидка 1000 рублей на Apple Pencil при покупе iPad",
          "discount": 1000,
          "sumsWithBonuses": false
        }
      ],
      [
        {
          "id": "dd1218888",
          "name": "Чехол Baseus в подарок при покупке iPad и Apple Pencil",
          "sumsWithBonuses": false,
          "gifts": [
            {
              "id": "98765",
              "quantity": 1
            }
          ]
        }
      ],
      [
        {
          "id": "dd771099",
          "name": "Подарок на выбор при заказе от 5000",
          "sumsWithBonuses": true,
          "giftOptions": [
            {
              "id": "12345",
              "quantity": 1
            },
            {
              "id": "56789",
              "quantity": 1
            }
          ],
          "giftOptionsLimit": 1
        }
      ]
    ]
    ...
}
```



### Синхронизация корзины с сайтом и наоборот&#x20;

Добавляется в **запрос** и в **ответ** расчета корзины:

* **`lastSyncDate`** - (опционально) это [**timestamp**](https://www.unixtimestamp.com/), время последней синхронизации (в миллисекундах) корзины

```json
{
    ...
      "lastSyncDate": 1644419057514,
    ...
}
```

Это поле отправляется в **запросе** только тогда, когда пользователь **авторизован**. Если приходит **0 (ноль)** в запросе, то это означает что пользователь ничего не добавлял в корзину пока был неавторизован.

При реализации этой фичи мы ожидаем что вы:

1. Добавите новое поле к примеру тоже **lastSyncDate** в базу данных с корзинами пользователей и изначальное значение этого поля для всех текущих и будущих корзин должно быть **0.**
2. При любом изменении корзины пользователя в базе данных, должны будете обновить поле **lastSyncDate** на текущий [**timestamp**](https://www.unixtimestamp.com/) **** (время в миллисекундах).
3. Добавите это поле в **ответ,** которое будет передавать его.
4. При **запросе** от нас вы должны будете сравнивать две **даты,** которая **пришла** в запросе и которая хранится **у вас** в базе данных, в случае если дата в запросе **свежее,** то необходимо **заменить** весь состав корзины в вашей базе данных из **нашего** **запроса** и **** обновить **** поле **lastSyncDate** на то которое пришло из нашего **запроса,** а если же дата из **вашей** базы данных окажется **свежее,** то вы должны будете **отправить** то состояние корзины которое в **вашей** базе данных и отправить тот **lastSyncDate** которое у вас. Если же **даты** **равные,** то все отлично и запрос можно обработать как **обычно.** Если мы отправили **0** и у вас **0**, то вам необходимо провести **слияние** (merge) нашего и вашего состава корзины и обновить поле **lastSyncDate** на текущий [**timestamp**](https://www.unixtimestamp.com/) **** (время в миллисекундах).
