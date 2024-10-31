# 3 курс
# Зарафутдинов Артём Маратович
# Проектная работа "Веб-ларек"

Стек: HTML, SCSS, TS, Webpack

Структура проекта:
- src/ — исходные файлы проекта
- src/components/ — папка с JS компонентами
- src/components/base/ — папка с базовым кодом

Важные файлы:
- src/pages/index.html — HTML-файл главной страницы
- src/types/index.ts — файл с типами
- src/index.ts — точка входа приложения
- src/scss/styles.scss — корневой файл стилей
- src/utils/constants.ts — файл с константами
- src/utils/utils.ts — файл с утилитами

- src/types/product.ts — файл с типом данных для товаров
- src/types/basket.ts — файл с типом данных для корзины
- src/types/order.ts — файл с типом данных для заказа
- src/models/products.ts — файл с моделью для товаров
- src/models/basket.ts — файл с моделью для корзины
- src/views/catalogView.ts — файл с представлением для каталога
- src/views/catalogItemView.ts — файл с представлением товара
- src/index.ts — точка входа приложения

## Установка и запуск
Для установки и запуска проекта необходимо выполнить команды

```
npm install
npm run start
```

или

```
yarn
yarn start
```
## Сборка

```
npm run build
```

или

```
yarn build
```

# Документация

## Общее описание архитектуры

За основу была взята архитектура MVP. Поэтому наличие связей между компонентами должно быть минимально.
Каждый компонент имеет: собственную модель, которая отвечает за хранение и получение данных и логику работы с ними, и отображение, которое показывает пользователю интерфейс.
Модель и отображение связаны презентером, в данном проекте это брокер событий, который вызывает события при изменеии в модели или в отображении и вызывает соответствующие методы.

Было выделено 3 основных компонента:
- Каталог товаров
- Корзина
- Оформление заказа

## Каталог товаров

Сам продукт реализован в виде интерфейсе `IProduct`, с полями соотвествующими тем, которые приходят с сервера, также добавлен флаг на наличие товара в корзине. Этот флаг никак не взаимодействует с корзиной, он влияет лишь на отображение в каталоге:
- `id:string;` - id товара
- `description:string;` - описание товара
- `image:string;` - ссылка на изображение товара
- `title:string;` - название товара
- `category:string;` - категория товара
- `price:number;` - цена товара
- `inBasket:boolean;` - флаг на наличие товара в корзине

Модель каталога `ICatalogModel` отвечает за хранение и получение товаров:
- `items: IProduct[];` - массив товаров
- `getProduct(id: string): IProduct | undefined;` - получение товара по id из модели
- `setProducts(products: IProduct[]): void;` - установка товаров в модель
- `getAllProducts(): IProduct[] | undefined;` - получение всех товаров из модели

Изначально при открытии приложения существует только базовая верстка сайта, которая сейчас показывается при запуске.
В момент открытия приложения создаётся запрос на получение данных о товарах, чтобы коректно передать их в модель, создадим дополнительный тип данных для этого.

Интерфейс `IProductAPIResponse` - тип данных, который приходит с сервера
- `total: number;` - количество товаров
- `items: IProduct[];` - массив товаров

## Корзина

Модель корзины по аналогии с моделью продуктов состоит из массива продуктов (их id и количества) и должна содержать методы для добавления и удаления товаров, увеличение/уменьшение количества товара.
(в видео, предложенном в задании, говорится о возможности увеличения и уменьшения кол-ва товара, хотя интерфейс в figma это никак не отображает)

Получаем следующий интерфейс `IBasketModel`:
- `items: Map<id, number>;` - Map id товаров и их количество
- `totalPrice: number;` - общая цена заказа
- `add(id: string): void;` - добавление товара в корзину
- `remove(id: string): void;` - удаление товара из корзины
- `clear(): void;` - очистка корзины

## Оформление заказа

После наполнение корзины товарами, пользователь может оформить заказ, для этого необходимо в форме указать свои данные в этом нам поможет интерфейс `IUserData`:
В выборе способа оплаты есть всего 2 варианта: онлайн, при получении. Для упрошения кода выделим дополнительный тип оплаты:
- `type Payment = "online" | "on-site";`
Интерфейс `IUserData`:
- `payment: Payment;` - способ оплаты
- `address: string;` - адрес
- `email: string;` - email
- `phone: string;` - телефон

После заполнения формы, пользователь может оформить заказ, для этого необходимо отправить данные на сервер, чтобы сохранить заказ.
Интерфейс `IOrderData`:
- `userData: IUserData;` - данные пользователя
- `total: number;` - общая цена заказа
- `items:string[];` - массив id товаров в корзине


## По умолчанию реализованные классы:

### Api
Класс `Api` предназначен для упрощения работы с API, обеспечивая методы для выполнения HTTP-запросов. 

#### Свойства:
- `baseUrl: string;` - базовый URL API
- `options: RequestInit;` - опции для HTTP-запросов

#### Методы:
- `constructor(baseUrl: string, options: RequestInit = {}): void;` - конструктор класса, инициализирует базовый URL и опции для HTTP-запросов
- `get(uri: string): Promise<object>;` - выполнение GET-запроса
- `post(uri: string, data: object, method: ApiPostMethods = 'POST'): Promise<object>;` - выполнение POST-запроса
- `protected handleResponse(response: Response): Promise<object>;` - обработка ответа от сервера


### EventEmitter
Класс `EventEmitter` реализует паттерн "издатель-подписчик" (publish-subscribe), позволяя объектам подписываться на события и реагировать на них

#### Свойства:
- `_events: Map<EventName, Set<Subscriber>>;` - Map событий и подписчиков

#### Методы:
- `on<T extends object>(event: EventName, callback: (data: T) => void): void;` - подписка на событие
- `off(event: EventName, callback: Subscriber): void;` - отписка от события
- `emit<T extends object>(event: string, data?: T): void;` - вызов события
- `trigger<T extends object>(event: string, context?: Partial<T>): (data: T) => void;` - вызов события с контекстом
- `constructor(): void;` - конструктор класса, инициализирует Map событий и подписчиков
- `offAll(): void;` - отписка от всех событий
- `onAll(callback: (event: EmitterEvent) => void): void;` - подписка на все события

## Созданные классы

Были реализованы основные интерфейсы для каталога товаров, корзины и заказа.
Каталог товара в `src/types/product.ts`, корзина в `src/types/basket.ts`.
Модели для каталога и корзины в `src/models/productModel.ts` и `src/models/basketModel.ts` соотвественно.
Представления для каталога в `src/views/catalogView.ts`.
В `src/index.ts` создаются экземпляры моделей и представлений, а также происходит инициализация приложения.
