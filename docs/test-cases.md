# Toolshop API — Test Cases (Тест-кейсы)

## TC-01 — Successful User Registration (Успешная регистрация пользователя)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev

**Test Data (Тестовые данные):**

Body
```json
{
  "first_name": "Ivan",
  "last_name": "Ivanov",
  "password": "QaToolshop_2026_Ivan!",
  "email": "{{registered_email}}"
}
```
Before request 
```before request 
const email = "ivan.qa." + pm.variables.replaceIn("{{$randomUUID}}") + "@example.com";

pm.environment.set("registered_email", email);
```
After request 
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
2. Во вкладке Sripts в Before request и After request, вставить скрипты из Test Data.
3. Отправить запрос:

```http
POST {{base_url}}/users/register
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

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь зарегистрирован.

**Test Data (Тестовые данные):**

Body
```json
{
  "email": "{{registered_email}}",
  "password": "QaToolshop_2026_Ivan!"
}
```
After request 
```after request 
pm.test("Авторизация успешна", function () {
    pm.response.to.have.status(200);
});

pm.test("В ответе есть access_token", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("access_token");
});

pm.test("access_token не пустой", function () {
    const body = pm.response.json();
    pm.expect(body.access_token).to.be.a("string");
    pm.expect(body.access_token.length).to.be.above(0);
});

pm.test("Тип токена Bearer", function () {
    const body = pm.response.json();
    pm.expect(body.token_type).to.eql("bearer");
});

pm.test("Есть срок жизни токена", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("expires_in");
});

const body = pm.response.json();
pm.environment.set("access_token", body.access_token);

const tokenParts = body.access_token.split(".");
const payload = JSON.parse(atob(tokenParts[1]));
pm.environment.set("user_id", payload.sub);
```

**Steps (Шаги):**

1. В теле запроса передать данные из Test Data.
2. Во вкладке Sripts в After request вставить скрипт из Test Data.
3. Отправить запрос:

```http
POST {{base_url}}/users/login
```
4. Проверить статус-код ответа.
5. Проверить наличие `access_token` в ответе.
6. Проверить в Environments наличие переменной access_token и сверить ее со значением в теле ответа.
7. Проверить в Environments наличие переменной user_id.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwczovL2FwaS5wcmFjdGljZXNvZnR3YXJldGVzdGluZy5jb20vdXNlcnMvbG9naW4iLCJpYXQiOjE3OTE4OTEwOTQsImV4cCI6MTc5MTg5MTM5NCwibmJmIjoxNzkxODkxMDk0LCJqdGkiOiJwcW8ycFVRd0lZYmJmWmRRIiwic3ViIjoiMDFtNHRmeTUxd241ZGtjMWFwNjk1bXhiczEiLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3Iiwicm9sZSI6InVzZXIifQ.cTI4dvTSbyq_NTG8Sc4rsQ1z_7vb5P5Ey1715Upl6a8",
    "token_type": "bearer",
    "expires_in": 300
}
```
---

## TC-03 — Create Cart (Создание корзины)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь зарегистрирован и авторизован.
* Токен валиден.

**Test Data (Тестовые данные):**

Authorization
```http
Токен {{access_token}}
```
After request 
```after request 
pm.test("Корзина создана", function () {
    pm.response.to.have.status(201);
});

pm.test("В ответе есть id корзины", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("id");
});

pm.test("ID корзины не пустой", function () {
    const body = pm.response.json();
    pm.expect(body.id).to.be.a("string");
    pm.expect(body.id.length).to.be.above(0);
});

const body = pm.response.json();
pm.environment.set("cart_id", body.id);
```

**Steps (Шаги):**

1. Во вкладке Authorization в Auth type выбрать Bearer Token и вставить переменную токена из Test Data.
2. Во вкладке Sripts After request вставить скрипт из Test Data.
3. Отправить запрос:

```http
POST {{base_url}}/carts
```

3. Проверить статус-код ответа.
4. Проверить наличие идентификатора корзины в ответе.
5. Проверить в Environments наличие переменной cart_id и сравнить ее значение со ответом id корзины из тела ответа.

**Expected Result (Ожидаемый результат):**

Статус-код: `201 Created`.

```json
{
    "id": "01m4tha5pnhhn1h1by55eqbwe7"
}
```
---

## TC-04 — Get Products (Получение списка товаров)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь авторизован.

**Test Data (Тестовые данные):**

After request 
```after request 
pm.test("Статус-код 200", function () {
    pm.response.to.have.status(200);
});

pm.test("В ответе есть массив товаров", function () {
    const body = pm.response.json();
    pm.expect(body.data).to.be.an("array");
});

pm.test("Список товаров не пустой", function () {
    const body = pm.response.json();
    pm.expect(body.data.length).to.be.above(0);
});

pm.test("У первого товара есть id и name", function () {
    const body = pm.response.json();
    pm.expect(body.data[0]).to.have.property("id");
    pm.expect(body.data[0]).to.have.property("name");
});

const body = pm.response.json();

pm.test("В списке есть товары", function () {
    pm.expect(body.data).to.be.an("array");
    pm.expect(body.data.length).to.be.above(0);
});

if (body.data && body.data.length > 0) {
    pm.environment.set("product_id", body.data[0].id);
}
```

**Steps (Шаги):**

1. Во вкладке Sripts After request вставить скрипт из Test Data.
2. Отправить запрос:

```http
GET {{base_url}}/products
```
3. Проверить статус-код ответа.
4. Проверить наличие массива товаров в ответе.
5. Проверить наличие `id` у товаров.
6. Проверить в Environments наличие переменной product_id и сравнить ее значение со ответом id товара из тела ответа.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
    "current_page": 1,
    "data": [
        {
            "id": "01M4TEQH1WZ557PYBH7DN47EP7",
            "name": "Combination Pliers",
            "description": "Versatile combination pliers designed for gripping, bending, and cutting wire with ease. Featuring chrome vanadium steel construction with induction-hardened cutting edges, these pliers deliver excellent grip and leverage for a wide range of tasks. The precision-machined jaws combine flat gripping surfaces with a pipe-grip section and integrated wire cutter for true multi-purpose functionality. Ergonomic bi-component handles reduce hand fatigue during extended use and provide a secure hold even with oily or gloved hands. The joint is precisely fitted to eliminate play and ensure smooth operation over thousands of cycles. Ideal for electricians, mechanics, and DIY enthusiasts tackling everyday projects around the workshop or job site.",
            "price": 14.15,
            "is_location_offer": false,
            "is_rental": false,
            "co2_rating": "D",
            "in_stock": true,
            "is_eco_friendly": false,
            "product_image": {
                "id": "01M4TEQH11R2GJNW3ZYW178HSN",
                "by_name": "Helinton Fantin",
                "by_url": "https://unsplash.com/@fantin",
                "source_name": "Unsplash",
                "source_url": "https://unsplash.com/photos/W8BNwvOvW4M",
                "file_name": "pliers01.avif",
                "title": "Combination pliers"
            },
            "category": {
                "id": "01M4TEQH0EXKP1RPRR0VSQTHJR",
                "name": "Pliers",
                "slug": "pliers"
            },
            "brand": {
                "id": "01M4TEQGN16P19GJVV00PSPJBH",
                "name": "ForgeFlex Tools"
            }
        }
}
```
---

## TC-05 — Add Product to Cart (Добавление товара в корзину)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь авторизован.
* Токен валиден.
* Корзина создана.

**Test Data (Тестовые данные):**

Authorization
```http
Токен {{access_token}}
```
Body
```json
{
  "product_id": "product_001",
  "quantity": 1
}
```
After request 
```after request 
pm.test("Товар добавлен в корзину", function () {
    pm.response.to.have.status(200);
});

pm.test("API подтверждает добавление", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("result");
});

pm.test("Результат: item added or updated", function () {
    const body = pm.response.json();
    pm.expect(body.result).to.eql("item added or updated");
});
```
**Steps (Шаги):**

1. Во вкладке Authorization в Auth type выбрать Bearer Token и вставить переменную токена из Test Data.
2. Во вкладке Sripts After request вставить скрипт из Test Data.
3. Отправить запрос:

```http
POST {{base_url}}/carts/{{cart_id}}
```
4. В теле запроса передать данные из Test Data.
5. Проверить статус-код ответа.
6. Проверить тело ответа.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
    "result": "item added or updated"
}
```
---

## TC-06 — Get Cart with Added Product (Получение корзины с добавленным товаром)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь авторизован.
* Токен валиден.
* Корзина создана.
* В корзину добавлен товар с известным `product_id`.

**Test Data (Тестовые данные):**

Authorization
```http
Токен {{access_token}}
```
After request 
```after request 
pm.test("Корзина получена", function () {
    pm.response.to.have.status(200);
});

pm.test("В ответе есть ID корзины", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("id");
});

pm.test("ID корзины совпадает с cart_id", function () {
    const body = pm.response.json();
    pm.expect(body.id).to.eql(pm.environment.get("cart_id"));
});

pm.test("В корзине есть товары", function () {
    const body = pm.response.json();
    pm.expect(body.cart_items).to.be.an("array");
    pm.expect(body.cart_items.length).to.be.above(0);
});

pm.test("Добавлен нужный товар", function () {
    const body = pm.response.json();
    const expectedProductId = pm.environment.get("product_id");
    const item = body.cart_items.find(function (item) {
        return item.product_id === expectedProductId;
    });

    pm.expect(item, "Товар с product_id не найден в корзине").to.not.be.undefined;
});

pm.test("Количество товара равно 1", function () {
    const body = pm.response.json();
    const expectedProductId = pm.environment.get("product_id");
    const item = body.cart_items.find(function (item) {
        return item.product_id === expectedProductId;
    });

    pm.expect(item.quantity).to.eql(1);
});

pm.test("У товара есть название", function () {
    const body = pm.response.json();
    const item = body.cart_items[0];

    pm.expect(item.product).to.have.property("name");
    pm.expect(item.product.name).to.be.a("string");
    pm.expect(item.product.name.length).to.be.above(0);
});

pm.test("У товара есть цена", function () {
    const body = pm.response.json();
    const item = body.cart_items[0];

    pm.expect(item.product).to.have.property("price");
    pm.expect(item.product.price).to.be.a("number");
});
```

**Steps (Шаги):**

1. Во вкладке Authorization в Auth type выбрать Bearer Token и вставить переменную токена из Test Data.
2. Отправить запрос:

```http
GET {{base_url}}/carts/{{cart_id}}
```
3. Проверить статус-код ответа.
4. Проверить наличие идентификатора корзины в ответе.
5. Проверить наличие массива `cart_items`.
6. Проверить, что добавленный товар отображается в `cart_items`.
7. Проверить, что количество товара равно `1`.

**Expected Result (Ожидаемый результат):**

Статус-код: `200 OK`.

```json
{
    "id": "01m4tjpptt4rrea9b141e7ad8d",
    "additional_discount_percentage": null,
    "lat": null,
    "lng": null,
    "cart_items": [
        {
            "id": "01m4tjptn27e0x8b49w0f6sea9",
            "quantity": 1,
            "discount_percentage": null,
            "cart_id": "01m4tjpptt4rrea9b141e7ad8d",
            "product_id": "01M4TJ5HJNPEF22R3P2MRSA05T",
            "product": {
                "id": "01M4TJ5HJNPEF22R3P2MRSA05T",
                "name": "Combination Pliers",
                "description": "Versatile combination pliers designed for gripping, bending, and cutting wire with ease. Featuring chrome vanadium steel construction with induction-hardened cutting edges, these pliers deliver excellent grip and leverage for a wide range of tasks. The precision-machined jaws combine flat gripping surfaces with a pipe-grip section and integrated wire cutter for true multi-purpose functionality. Ergonomic bi-component handles reduce hand fatigue during extended use and provide a secure hold even with oily or gloved hands. The joint is precisely fitted to eliminate play and ensure smooth operation over thousands of cycles. Ideal for electricians, mechanics, and DIY enthusiasts tackling everyday projects around the workshop or job site.",
                "price": 14.15,
                "is_location_offer": false,
                "is_rental": false,
                "co2_rating": "D",
                "in_stock": true,
                "is_eco_friendly": false
            }
        }
    ]
}
```
---

## TC-07 — Remove Product from Cart (Удаление товара из корзины)

**Preconditions (Предусловия):**

* Postman открыт.
* API доступен.
* Окружение - Toolshop Dev
* Пользователь авторизован.
* Токен валиден.
* Создана корзина и известен её `cart_id`.
* В корзину добавлен товар с известным `product_id`.

**Test Data (Тестовые данные):**

Authorization
```http
Токен {{access_token}}
```
After request 
```after request 
pm.test("Товар удалён из корзины", function () {
    pm.response.to.have.status(204);
});

pm.test("У успешного удаления нет тела ответа", function () {
    pm.expect(pm.response.text()).to.eql("");
});
```
**Steps (Шаги):**

1. Во вкладке Authorization в Auth type выбрать Bearer Token и вставить переменную токена из Test Data.
2. Отправить запрос:

```http
DELETE {{base_url}}/carts/{{cart_id}}/product/{{product_id}}
```
3. Проверить статус-код ответа.
4. Проверить коризну.

**Expected Result (Ожидаемый результат):**

Статус-код запроса удаления: `204 No Content`.

Тело ответа отсутствует.

Статус-код запроса корзины: `200 OK`.

```json
{
  "id": "cart_67890",
  "user_id": "user_12345",
  "cart_items": []
}
```
