# Toolshop API — Test Cases (Тест-кейсы)

## TC-01 — Successful User Registration (Успешная регистрация пользователя)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev

**Test Data (Тестовые данные):**

```json
{
  "first_name": "Ivan",
  "last_name": "Ivanov",
  "password": "QaToolshop_2026_Ivan!",
  "email": "{{registered_email}}"
}
```
```before request 
const email = "ivan.qa." + pm.variables.replaceIn("{{$randomUUID}}") + "@example.com";

pm.environment.set("registered_email", email);
```
```after request 
pm.test("Пользователь создан", function () {
    pm.response.to.have.status(201);
});

pm.test("В ответе есть id пользователя", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("id");
});

pm.test("Email совпадает с отправленным", function () {
    const body = pm.response.json();
    const sentEmail = pm.environment.get("registered_email");

    pm.expect(body.email).to.eql(sentEmail);
});

pm.test("Пароль не возвращается в ответе", function () {
    const body = pm.response.json();
    pm.expect(body).to.not.have.property("password");
});
```

**Steps (Шаги):**

1. В теле запроса передать данные из Test Data.
2. Во вкладке Sripts в before request и after request, вставить скрипты из Test Data.
3. Отправить запрос:

```http
POST {{base_url}}/users/register
Content-Type: application/json
```
4. Проверить статус-код ответа.
5. Проверить наличие идентификатора созданного пользователя.
6. Проверить остальные поля ответа.

**Expected Result (Ожидаемый результат):**

Статус-код: `201 Created`.

```json
{
    "first_name": "Ivan",
    "last_name": "Ivanov",
    "email": "ivan.qa.c0765582-aaa2-4b94-943a-306e3e9911cd@example.com",
    "id": "01m4tfy51wn5dkc1ap695mxbs1",
    "created_at": "2026-10-13 11:21:34",
    "address": {
        "street": null,
        "house_number": null,
        "city": null,
        "state": null,
        "country": null,
        "postal_code": null
    }
}
```
**Примечание:**
email генерируется случайным образом посредством {{$randomUUID}} в before request скрипта, поэтому в теле ответа, email будет генерироваться с каждым разом новый.
---

## TC-02 — Successful User Login (Успешная авторизация пользователя)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован.
* Известны email и пароль пользователя.

**Test Data (Тестовые данные):**

```json
{
  "email": "ivan.qa.3f8b1c2e-9d47-4f6a-8f21-6b9d2a7c1234@example.com",
  "password": "QaToolshop_2026_Ivan!"
}
```

**Steps (Шаги):**

1. Отправить запрос:

```http
POST {{base_url}}/users/login
Content-Type: application/json
```

2. В теле запроса передать данные из Test Data.
3. Проверить статус-код ответа.
4. Проверить наличие `access_token` в ответе.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## TC-03 — Create Cart (Создание корзины)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован и авторизован.
* Получен валидный `access_token`.

**Test Data (Тестовые данные):**

```http
Authorization: Bearer {{access_token}}
```

**Steps (Шаги):**

1. Отправить запрос:

```http
POST {{base_url}}/carts
Authorization: Bearer {{access_token}}
```

2. Проверить статус-код ответа.
3. Проверить наличие идентификатора корзины в ответе.

**Expected Result (Ожидаемый результат):**

Статус-код: `201 Created`.

```json
{
  "id": "cart_67890",
  "user_id": "user_12345",
  "cart_items": []
}
```

---

## TC-04 — Get Products (Получение списка товаров)

**Preconditions (Предусловия):**

* API доступен.

**Test Data (Тестовые данные):**

```http
Отсутствуют
```

**Steps (Шаги):**

1. Отправить запрос:

```http
GET {{base_url}}/products
```

2. Проверить статус-код ответа.
3. Проверить наличие массива товаров в ответе.
4. Проверить наличие `id` у товаров.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
  "data": [
    {
      "id": "product_001",
      "name": "Toolshop Screwdriver",
      "price": 19.99
    }
  ]
}
```

---

## TC-05 — Add Product to Cart (Добавление товара в корзину)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован и авторизован.
* Получен валидный `access_token`.
* Создана корзина и известен её `cart_id`.
* Получен валидный `product_id`.

**Test Data (Тестовые данные):**

```json
{
  "product_id": "product_001",
  "quantity": 1
}
```

**Steps (Шаги):**

1. Отправить запрос:

```http
POST {{base_url}}/carts/{{cart_id}}/items
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

2. В теле запроса передать данные из Test Data.
3. Проверить статус-код ответа.
4. Отправить запрос:

```http
GET {{base_url}}/carts/{{cart_id}}
Authorization: Bearer {{access_token}}
```

5. Проверить, что добавленный товар отображается в `cart_items`.
6. Проверить, что количество товара равно `1`.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
  "cart_id": "cart_67890",
  "product_id": "product_001",
  "quantity": 1
}
```

---

## TC-06 — Get Cart with Added Product (Получение корзины с добавленным товаром)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован и авторизован.
* Получен валидный `access_token`.
* Создана корзина и известен её `cart_id`.
* В корзину добавлен товар с известным `product_id`.

**Test Data (Тестовые данные):**

```http
Authorization: Bearer {{access_token}}
```

**Steps (Шаги):**

1. Отправить запрос:

```http
GET {{base_url}}/carts/{{cart_id}}
Authorization: Bearer {{access_token}}
```

2. Проверить статус-код ответа.
3. Проверить наличие идентификатора корзины в ответе.
4. Проверить наличие массива `cart_items`.
5. Проверить, что добавленный товар отображается в `cart_items`.
6. Проверить, что количество товара равно `1`.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
  "id": "cart_67890",
  "user_id": "user_12345",
  "cart_items": [
    {
      "product_id": "product_001",
      "quantity": 1
    }
  ]
}
```

---

## TC-07 — Remove Product from Cart (Удаление товара из корзины)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован и авторизован.
* Получен валидный `access_token`.
* Создана корзина и известен её `cart_id`.
* В корзину добавлен товар с известным `product_id`.

**Test Data (Тестовые данные):**

```http
Authorization: Bearer {{access_token}}
```

**Steps (Шаги):**

1. Отправить запрос:

```http
DELETE {{base_url}}/carts/{{cart_id}}/items/{{product_id}}
Authorization: Bearer {{access_token}}
```

2. Проверить статус-код ответа.
3. Отправить запрос:

```http
GET {{base_url}}/carts/{{cart_id}}
Authorization: Bearer {{access_token}}
```

4. Проверить статус-код повторного запроса корзины.
5. Проверить, что удалённый товар отсутствует в `cart_items`.
6. Проверить, что массив `cart_items` пустой.

**Expected Result (Ожидаемый результат):**

Статус-код запроса удаления: `204 No Content`.

Тело ответа отсутствует.

Статус-код повторного запроса корзины: `200 OK`.

```json
{
  "id": "cart_67890",
  "user_id": "user_12345",
  "cart_items": []
}
```
