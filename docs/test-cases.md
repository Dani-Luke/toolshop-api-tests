# Toolshop API — Test Cases (Тест-кейсы)

## TC-01 — Successful User Registration (Успешная регистрация пользователя)

**Preconditions (Предусловия):**

* Postman открыт.
* Выбран environment `Toolshop Dev`.
* API доступен.

**Test Data (Тестовые данные):**

* First Name: `Ivan`
* Last Name: `Ivanov`
* Password: `QaToolshop_2026_Ivan!`
* Email: уникальный email, формируемый через `{{$randomUUID}}`

**Steps (Шаги):**

1. Открыть запрос `POST register user`.
2. Убедиться, что выбран environment `Toolshop Dev`.
3. В Pre-request Script убедиться, что уникальный email сохраняется в переменную `registered_email`.
4. В Body убедиться, что передаются `first_name`, `last_name`, `password` и `{{registered_email}}`.
5. Отправить запрос.
6. Проверить статус-код ответа.
7. Проверить, что email в ответе совпадает со значением `registered_email`.
8. Проверить наличие идентификатора созданного пользователя.

**Expected Result (Ожидаемый результат):**
Пользователь успешно зарегистрирован. API возвращает `201 Created`, данные созданного пользователя и его email. Email в ответе совпадает с отправленным.

---

## TC-02 — Successful User Login (Успешная авторизация пользователя)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Email: email зарегистрированного пользователя.
* Password: валидный пароль пользователя.

**Steps (Шаги):**

1. Открыть запрос `POST login user`.
2. Убедиться, что выбран environment `Toolshop Dev`.
3. В Body указать валидные `email` и `password`.
4. Отправить запрос.
5. Проверить статус-код ответа.
6. Проверить наличие `access_token` в ответе.
7. Проверить сохранение `access_token` и `user_id` в environment.

**Expected Result (Ожидаемый результат):**
Пользователь успешно авторизован. API возвращает `200 OK` и `access_token`. Токен и `user_id` сохраняются в environment для последующих запросов.

---

## TC-03 — Create Cart (Создание корзины)

**Preconditions (Предусловия):**

* Пользователь зарегистрирован и авторизован.
* `access_token` сохранён в environment.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Authorization: `Bearer {{access_token}}`

**Steps (Шаги):**

1. Открыть запрос `POST create cart`.
2. Убедиться, что в Authorization используется Bearer Token `{{access_token}}`.
3. Отправить запрос.
4. Проверить статус-код ответа.
5. Проверить наличие идентификатора корзины в ответе.
6. Проверить сохранение `cart_id` в environment.

**Expected Result (Ожидаемый результат):**
Корзина успешно создана. API возвращает `201 Created` и идентификатор корзины. `cart_id` сохраняется в environment для последующих запросов.

---

## TC-04 — Get Products (Получение списка товаров)

**Preconditions (Предусловия):**

* API доступен.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Отсутствуют.

**Steps (Шаги):**

1. Открыть запрос `GET products`.
2. Убедиться, что выбран environment `Toolshop Dev`.
3. Отправить запрос.
4. Проверить статус-код ответа.
5. Проверить наличие массива товаров в ответе.
6. Проверить наличие идентификатора `id` у товаров.
7. Проверить сохранение `product_id` первого товара в environment.

**Expected Result (Ожидаемый результат):**
Список товаров успешно получен. API возвращает `200 OK` и массив товаров. `product_id` первого товара сохраняется в environment для добавления товара в корзину.

---

## TC-05 — Add Product to Cart (Добавление товара в корзину)

**Preconditions (Предусловия):**

* Пользователь авторизован.
* Корзина создана.
* `product_id` получен из списка товаров.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Cart ID: `{{cart_id}}`
* Product ID: `{{product_id}}`
* Quantity: `1`
* Authorization: `Bearer {{access_token}}`

**Steps (Шаги):**

1. Открыть запрос `POST add item to cart`.
2. Убедиться, что в URL используется `{{cart_id}}`.
3. Убедиться, что в Authorization используется Bearer Token `{{access_token}}`.
4. В Body указать `{{product_id}}` и `quantity: 1`.
5. Отправить запрос.
6. Проверить статус-код ответа.
7. Открыть запрос `GET cart by id` и отправить его.
8. Проверить наличие добавленного товара в `cart_items`.
9. Проверить, что количество товара равно `1`.

**Expected Result (Ожидаемый результат):**
Товар успешно добавлен в корзину. В ответе запроса корзины товар с переданным `product_id` отображается в `cart_items`, а его количество равно `1`.

---

## TC-06 — Get Cart with Added Product (Получение корзины с добавленным товаром)

**Preconditions (Предусловия):**

* Пользователь авторизован.
* Корзина создана.
* Товар добавлен в корзину.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Cart ID: `{{cart_id}}`
* Product ID: `{{product_id}}`
* Authorization: `Bearer {{access_token}}`

**Steps (Шаги):**

1. Открыть запрос `GET cart by id`.
2. Убедиться, что в URL используется `{{cart_id}}`.
3. Убедиться, что в Authorization используется Bearer Token `{{access_token}}`.
4. Отправить запрос.
5. Проверить статус-код ответа.
6. Проверить наличие идентификатора корзины в ответе.
7. Проверить наличие массива `cart_items`.
8. Проверить наличие добавленного товара в `cart_items`.
9. Проверить, что количество товара равно `1`.

**Expected Result (Ожидаемый результат):**
Корзина успешно получена. API возвращает `200 OK`, данные корзины и массив `cart_items`. Добавленный товар отображается в корзине с количеством `1`.

---

## TC-07 — Remove Product from Cart (Удаление товара из корзины)

**Preconditions (Предусловия):**

* Пользователь авторизован.
* Корзина создана.
* Товар добавлен в корзину.
* Выбран environment `Toolshop Dev`.

**Test Data (Тестовые данные):**

* Cart ID: `{{cart_id}}`
* Product ID: `{{product_id}}`
* Authorization: `Bearer {{access_token}}`

**Steps (Шаги):**

1. Открыть запрос `DELETE remove product from cart`.
2. Убедиться, что в URL используются `{{cart_id}}` и `{{product_id}}`.
3. Убедиться, что в Authorization используется Bearer Token `{{access_token}}`.
4. Отправить запрос.
5. Проверить статус-код ответа.
6. Открыть запрос `GET cart by id — after delete`.
7. Убедиться, что в URL используется `{{cart_id}}`.
8. Убедиться, что в Authorization используется Bearer Token `{{access_token}}`.
9. Отправить запрос.
10. Проверить статус-код повторного запроса корзины.
11. Проверить отсутствие удалённого товара в `cart_items`.
12. Проверить, что массив `cart_items` пустой.

**Expected Result (Ожидаемый результат):**
Товар успешно удалён из корзины. Запрос удаления возвращает `204 No Content`. Повторный запрос корзины возвращает `200 OK`, массив `cart_items` пустой.
