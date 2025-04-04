---
title: JWT и с чем его едят
---

# Введение.

>[!book] Что такое?
> Токен доступа –  это единица информация, используемая для аутентификации пользователя. Токен предоставляется в зашифрованном виде и содержит информацию о пользователе и его правах.

Есть такой стандарт ([RFC 7519](https://tools.ietf.org/html/rfc7519)) JWT[^1], созданный по сути, для того, чтобы на основе [[JSON]] генерировать токены доступа. Создан он для упрощения генерации и аутентификации по токену. Есть интерактивный инструмент для декодирования JWT и справочник по библиотекам [JWT.io](https://jwt.io/) 

# Как работает?

## Структура самого JSON.

Тут всё довольно просто, сам токен определяется в формате: `header.payload.signature`. 

- *Заголовок (Header).*
	- Здесь содержится **информация о** типе **токена** и алгоритме генерации.
		- Все алгоритмы будут предоставлены в виде [[#Доступные алгоритмы|таблицы]] дальше в статье.
- *Полезные данные (Payload).*
	- Здесь уже содержится **информация о пользователе**, его роли, его правах и т.д. 
- *Подпись (Signature).*
	- Самая важная часть.
	- Как видно из названия содержит **подпись**.

## Пример токена[^2]

**Header:**

```js
header = { "alg": "HS256", "typ": "JWT"}
```

**Payload:**

```js
payload = { "userId": "b08f86af-35da-48f2-8fab-cef3904660bd" }
```

**А генерируется код таким псевдо-кодом:**

```js
const SECRET_KEY = 'cAtwa1kkEy'
const unsignedToken = base64urlEncode(header) + '.' + base64urlEncode(payload)
const signature = HMAC-SHA256(unsignedToken, SECRET_KEY)

const token = unsignedToken + '.' + signature
```

**Итоговый результат (перенос строк для наглядности):**

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.
eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ.
-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM
```

То JWT-токен это закодированный с помощью [[Base64Url]] заголовок и полезная нагрузка (`unsignedToken`) + точка + подпись.

## Проверка подписи

Для проверки подписи на сервере мы запускаем такой код (псевдо-код):

```js
const SECRET_KEY = 'cAtwa1kkEy'

function verifyToken(token, secretKey):
    parts = split(token, ".")
    if length(parts) != 3:
        return false

    headerB64 = parts[0]
    payloadB64 = parts[1]
    signature = parts[2]

    dataToSign = headerB64 + "." + payloadB64
    expectedSignature = HMACSHA256(dataToSign, secretKey)

    if signature != expectedSignature:
        return false

    return true
```

Сначала проверяем, есть ли у нас все три части токена. Это важная проверка, потому что JWT поддерживает подпись `none`, то есть её полное отсутствие. А значит, злоумышленник может просто создать подходящий заголовок и данные и отправить неподписанный токен. Затем подписываем тот заголовок и данные, которые прислал клиент и сравниваем это с подписью, которую прислал нам клиент.

# Доступные алгоритмы

|Алгоритм|Тип|Описание|
|---|---|---|
|`HS256`|Симметричный|HMAC с SHA-256. Использует один секретный ключ для подписи и проверки.|
|`HS384`|Симметричный|HMAC с SHA-384. Использует один секретный ключ для подписи и проверки.|
|`HS512`|Симметричный|HMAC с SHA-512. Использует один секретный ключ для подписи и проверки.|
|`RS256`|Асимметричный|RSA с SHA-256. Использует закрытый ключ для подписи и открытый ключ для проверки.|
|`RS384`|Асимметричный|RSA с SHA-384. Использует закрытый ключ для подписи и открытый ключ для проверки.|
|`RS512`|Асимметричный|RSA с SHA-512. Использует закрытый ключ для подписи и открытый ключ для проверки.|
|`ES256`|Асимметричный|ECDSA с SHA-256. Использует эллиптические кривые для подписи и проверки.|
|`ES384`|Асимметричный|ECDSA с SHA-384. Использует эллиптические кривые для подписи и проверки.|
|`ES512`|Асимметричный|ECDSA с SHA-512. Использует эллиптические кривые для подписи и проверки.|
|`PS256`|Асимметричный|RSA-PSS с SHA-256. Улучшенная версия RSA с PSS-паддингом.|
|`PS384`|Асимметричный|RSA-PSS с SHA-384. Улучшенная версия RSA с PSS-паддингом.|
|`PS512`|Асимметричный|RSA-PSS с SHA-512. Улучшенная версия RSA с PSS-паддингом.|
|`none`|Без подписи|Не используется подпись или ключи (опасно для использования в продакшене).|

[^1]: **Источник:** https://ru.wikipedia.org/wiki/JSON_Web_Token
[^2]: **Источник:** https://habr.com/ru/articles/340146/