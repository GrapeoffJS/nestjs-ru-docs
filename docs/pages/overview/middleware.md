# Промежуточные обработчики

Промежуточный обработчик — это функция, которая вызывается **перед** обработчиком маршрута. Промежуточные обработчики
имеют доступ к объектам [запроса](https://expressjs.com/en/4x/api.html#req)
и [ответа](https://expressjs.com/en/4x/api.html#res), а также к функции `next()` в цикле запрос-ответ приложения.
Следующий промежуточный обработчик обычно обозначается переменной с именем `next`.

![](https://docs.nestjs.com/assets/Middlewares_1.png)

Промежуточные обработчики в **Nest** по умолчанию являются эквивалентом промежуточных обработчиков
в [express](https://expressjs.com/en/guide/using-middleware.html). Следующее описание из официальной документации
express описывает возможности промежуточных обработчиков:

> Промежуточные обработчики могут выполнять следующие задачи:
> * Выполнять любой код
> * Изменять объекты запроса и ответа
> * Завершать цикл запроса-ответа
> * Вызывать следующий в очереди промежуточный обработчик
> * Если текущий промежуточный обработчик не завершает цикл запроса-ответа, она должна вызвать метод `next()`, чтобы передать управление следующему промежуточному обработчику. В противном случае обработка запроса зависнет.

Вы можете реализовать собственный промежуточный обработчик либо в функции, либо в классе, помеченном
декоратором `@Injectable()`. Класс должен реализовать интерфейс `NestMiddleware`, в то время как функция не имеет
никаких особых требований. Давайте начнём с реализации простого промежуточного обработчика, используя вариант с классом.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* logger.middleware.ts */

import { Injectable, NestMiddleware } from '@nestjs/common';
import { NextFunction, Request, Response } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction) {
        console.log('Request...');
        next();
    }
}

```

#### **JavaScript**

```js
/* logger.middleware.js */

import { Injectable } from '@nestjs/common';

@Injectable()
export class LoggerMiddleware {
    use(req, res, next) {
        console.log('Request...');
        next();
    }
}

```

<!-- tabs:end -->

## Внедрение зависимости

Промежуточные обработчики в **Nest** полностью поддерживают внедрение зависимости. Точно так же, как поставщики или
контроллеры, они могут **внедрять зависимости** которые доступны в рамках этого же модуля. В основном, это делается
через конструктор.

## Регистрация промежуточных обработчиков

В декораторе `@Module()` нет ни единого места, где можно было бы зарегистрировать промежуточный обработчик. Вместо этого
регистрация проходит в методе `configure()` в классе модуля. Модули, которые содержат промежуточные обработчики, обязаны
реализовать интерфейс `NestModule`. Давайте применим `LoggerMiddleware` на уровне `AppModule`.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes('cats');
    }
}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule {
    configure(consumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes('cats');
    }
}
```

<!-- tabs:end -->

В примере выше мы применили `LoggerMiddleware` для обработчиков маршрута `/cats`, которых мы определили до этого
внутри `CatsController`. Мы также можем дополнительно ограничить промежуточный обработчик определённым методом запроса,
передав в метод `forRoutes()` объект, содержащий путь маршрута и метод запроса. Обратите внимание, что в приведённом
ниже примере мы импортируем перечисление `RequestMethod` для указания желаемого типа запроса.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { MiddlewareConsumer, Module, NestModule, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes({ path: 'cats', method: RequestMethod.GET });
    }
}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule {
    configure(consumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes({ path: 'cats', method: RequestMethod.GET });
    }
}
```

<!-- tabs:end -->

> [!NOTE]
> Метод `configure()` может быть сделан асинхронным (т.е. вы можете использовать оператор `await` для ожидания выполнения асинхронной операции внутри тела метода `configure()`).

## Шаблоны маршрутов

Маршруты, основанные на шаблонах также поддерживаются. Например, символ звёздочка используется как **подстановочный
знак** и представляет собой абсолютно любую последовательность символов.

```typescript
forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
```

Последовательности вроде `abcd` `ab_cd` `abecd` и даже `abjjfwfjwfkwnwcnwkneccd` будут соответствовать шаблону `ab*cd`.
Также, такие символы, как `?`, `+`, `*`, и `()` тоже могут использоваться при описании пути к конечной точке.
Дефис (`-`) и точка (`.`) не являются шаблонными и интерпретируются как обычные символы.

> [!WARNING]
> `fastify` использует последнюю версию пакета `path-to-regexp`, которая больше не поддерживает символ звёздочки `*` в качестве подстановочного знака. Вместо этого, вы должны использовать параметры (например: `(.*)`, `:splat*`).

## Middleware consumer

`MiddlewareConsumer` — класс-помощник. Он предоставляет несколько встроенных методов для управления промежуточными
обработчиками. Все они могут быть вызваны в стиле [fluent](https://wikicsu.ru/wiki/Fluent_interface).
Метод `forRoutes()` может принимать в себя одну строку, несколько строк, объект типа `RouteInfo`, класс контроллера и
даже несколько классов контроллеров. В большинстве случаев вы будете просто передавать список **контроллеров**,
разделённых запятыми. Ниже приведён пример с одним контроллером:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
    imports: [CatsModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes(CatsController);
    }
}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
    imports: [CatsModule],
})
export class AppModule {
    configure(consumer) {
        consumer
            .apply(LoggerMiddleware)
            .forRoutes(CatsController);
    }
}
```

<!-- tabs:end -->

> [!NOTE]
> Метод `apply()` может принимать либо один, либо [несколько промежуточных](#Несколько-промежуточных-обработчиков) обработчиков.

## Исключение маршрутов

Иногда мы хотим **исключить** определённые маршруты из области действия промежуточного обработчика. Мы можем очень
просто исключить их при помощи метода `exclude()`. Этот метод может принимать в себя одну строку, несколько строк или
объект типа `RouteInfo`, определяющий исключаемы маршруты, как это показано ниже:

```typescript
consumer
    .apply(LoggerMiddleware)
    .exclude(
        { path: 'cats', method: RequestMethod.GET },
        { path: 'cats', method: RequestMethod.POST },
        'cats/(.*)',
    )
    .forRoutes(CatsController);
```

> [!NOTE]
> Метод `exclude()` поддерживает подстановочные символы благодаря пакету [path-to-regexp](https://github.com/pillarjs/path-to-regexp#parameters).

В приведённом выше примере, `LoggerMiddleware` будет привязан ко всем маршрутам **кроме** тех трех, что переданы
метод `exclude()`

## Функциональные промежуточные обработчики

Класс `LoggerMiddleware`, который мы использовали, был крайне прост. У него не было полей, дополнительных методов, а
также зависимостей. Почему мы не можем сделать его простой функцией? — На самом деле можем. Данный тип промежуточных
обработчиков называется **функциональные промежуточные обработчики**. Давайте сделаем наш `LoggerMiddleware`
в функциональным промежуточным обработчиком, чтобы продемонстрировать различия:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* logger.middleware.ts */

import { NextFunction, Request, Response } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
    console.log(`Request...`);
    next();
};
```

#### **JavaScript**

```js
/* logger.middleware.js */

export function logger(req, res, next) {
    console.log(`Request...`);
    next();
};
```

<!-- tabs:end -->

И зарегистрируем его в `AppModule`:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

consumer
    .apply(logger)
    .forRoutes(CatsController);
```

#### **JavaScript**

```js
/* app.module.js */

consumer
    .apply(logger)
    .forRoutes(CatsController);
```

<!-- tabs:end -->

> [!NOTE]
> Подумайте об использовании более простых **функциональных промежуточных обработчиков** вместо обычных, когда дополнительные зависимости не требуются.

## Несколько промежуточных обработчиков

Как уже упомянуто выше, для того чтобы привязать к маршруту несколько промежуточных обработчиков, просто передайте их в
метод `apply()` через запятую:

```typescript
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

## Глобальные промежуточные обработчики

Если вам нужно привязать промежуточный обработчик ко всем зарегистрированным маршрутам, мы можем использовать
метод `use()`, который поставляется с экземпляром типа `INestApplication`:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* main.ts */

const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```

#### **JavaScript**

```js
/* main.js */

const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```

<!-- tabs:end -->

> [!NOTE]
> Доступ к контейнеру внедрения зависимостей в глобальных промежуточных обработчиках не возможен. Вместо этого, вы можете использовать функциональный промежуточный обработчик. Также, вы можете использовать классовый промежуточный обработчик и зарегистрировать его с помощью `.forRoutes('*')` в рамках `AppModule` (или где-либо ещё).
