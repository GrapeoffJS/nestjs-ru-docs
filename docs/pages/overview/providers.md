# Поставщики

Поставщики — фундаментальная концепция в **Nest**. Множество базовых классов **Nest** могут быть рассмотрены в качестве
провайдеров — сервисы, репозитории, фабрики, помощники и так далее. Главная идея поставщиков заключается в том, что они
могут быть **внедрены** в качестве зависимости; это значит, что объекты могут создавать различные отношения друг с
другом, а функция «подключения» экземпляров этих объектов может быть в значительной степени передана системе
выполнения **Nest**.

![](https://docs.nestjs.com/assets/Components_1.png)

В предыдущей главе, мы сделали простенький `CatsController`. Контроллеры могут обрабатывать **HTTP** запросы и
передавать более сложные задачи поставщикам. Поставщики являются простыми **JavaScript** классами, которые объявлены в
массиве `providers` в [модуле]().

> [!NOTE] Так как **Nest** даёт возможность проектировать и организовывать зависимости в более объектно-ориентированном стиле, мы крайне рекомендуем вам следовать принципам [SOLID](https://inlnk.ru/l0E0eX)).

## Сервисы

Давайте начнём с создания простого `CatsService`. Этот сервис будет ответственен за хранение и возвращение данных и
предназначен для использования в `CatsController`, что делает его отличным кандидатом в поставщики.

**_cats.service.ts_**


<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
    private readonly cats: Cat[] = [];

    create(cat: Cat) {
        this.cats.push(cat);
    }

    findAll(): Cat[] {
        return this.cats;
    }
}
```

#### **JavaScript**

```js
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
    constructor() {
        this.cats = [];
    }

    create(cat) {
        this.cats.push(cat);
    }

    findAll() {
        return this.cats;
    }
}
```

<!-- tabs:end -->


<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
    private readonly cats: Cat[] = [];

    create(cat: Cat) {
        this.cats.push(cat);
    }

    findAll(): Cat[] {
        return this.cats;
    }
}
```

#### **JavaScript**

```js
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
    constructor() {
        this.cats = [];
    }

    create(cat) {
        this.cats.push(cat);
    }

    findAll() {
        return this.cats;
    }
}
```

<!-- tabs:end -->

> [!NOTE] Чтобы создать интерфейс при помощи **CLI**, просто запустите эту команду: `$ nest g service cats`

Наш `CatsService` является простым классом с одним полем и двумя методами. Единственное из того, что мы не использовали
ранее — это декоратор `@Injectable`. Данный декоратор прикрепляет к классу метаданные, которые говорят,
что `CatsService` это класс, который может быть обработан контейнером инверсии контроля **Nest**
(Nest
[IoC](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D1%8F_%D1%83%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F)
container) Кстати, в этом примере также используется интерфейс `Cat`, который может выглядеть как-то так:

```typescript
export interface Cat {
    name: string;
    age: number;
    breed: string;
}
```

Сейчас, когда у нас есть сервис дя получения котиков, давайте используем его внутри `CatsController`:

**_cats.service.ts_**


<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Body, Controller, Get, Post } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
    constructor(private catsService: CatsService) {}

    @Post()
    async create(@Body() createCatDto: CreateCatDto) {
        this.catsService.create(createCatDto);
    }

    @Get()
    async findAll(): Promise<Cat[]> {
        return this.catsService.findAll();
    }
}
```

#### **JavaScript**

```js
import { Controller, Get, Post, Body, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
    constructor(catsService) {
        this.catsService = catsService;
    }

    @Post()
    @Bind(Body())
    async create(createCatDto) {
        this.catsService.create(createCatDto);
    }

    @Get()
    async findAll() {
        return this.catsService.findAll();
    }
}
```

<!-- tabs:end -->

`CatsService` был **внедрён** через конструктор. Заметьте, что мы использовали модификатор доступа `private`. Это
сокращение позволяет нам объявить и сразу же инициализировать `catsService` в одном и том же месте. Данная запись
эквивалентна приведённой снизу:

```typescript
import { Controller } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
    private catsService: CatsService

    constructor(catsService: CatsService) {
        this.catsService = catsService;
    }

    // ...
}
```

## Внедрение зависимости

**Nest** построен вокруг мощного паттерна проектирования, известного под названием **Внедрение зависимости**. Мы
рекомендуем прочитать о нём замечательную статью в официальной документации
[**Angular**](https://angular.io/guide/dependency-injection).

В **Nest**, благодаря возможностям **TypeScript**, очень просто управлять зависимостями потому что они разрешаются
только по типу. В примере ниже, **Nest** разрешит зависимость от `СatsService` при помощи создания и возвращения
экземпляра `CatsService` (или, в случае обычного singleton, при помощи возвращения уже существующего экземпляра). Эта
зависимость разрешена и передана в конструктор вашего контроллера (или присвоена указанному свойству).

```typescript
import { Controller } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller()
class CatsController {
    constructor(private catsService: CatsService) {}
}
```

## Области внедрения

Обычно, поставщики имеют время жизни ("область внедрения"), синхронизированное с жизненным циклом приложения. Когда
приложение подготовлено к запуску, каждая зависимость должна быть разрешена, и, следовательно, каждый поставщик должен
быть проинициализирован. Точно также, когда приложение прекращает свою работу, каждый поставщик будет уничтожен. Тем не
менее есть способы сделать так, чтобы областью внедрения вашего поставщика был запрос. Вы можете прочитать об этом
подробнее [здесь]().

## Собственные поставщики

**Nest** имеет встроенную поддержку контейнера инверсии контроля («IoC») который разрешает зависимости между
поставщиками. Такая функциональность лежит в основе механизма внедрения зависимости, который был описан выше, но на
самом деле он даёт куда мощнее, чем вам может показаться после нашего описания. Есть несколько способов создания
поставщика: вы можете использовать простые значения, классы и даже синхронные или асинхронные фабрики. Мы представили
больше примеров [тут]().

## Необязательные поставщики

Время от времени, у вас могут быть зависимости, не требующие обязательного разрешения. К примеру, ваш класс может
зависеть от **объекта конфигурации**, но если он не был передан, то должно использоваться значение по умолчанию. В таком
случае зависимость становится необязательной, потому что её отсутствие не приведёт к ошибкам.

Чтобы обозначить поставщика как необязательного используйте декоратор `@Optional()` во входящих в конструктор
параметрах.

```typescript
import { Inject, Injectable, Optional } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
    constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

Заметьте, что в данном примере мы используем собственного поставщика, поэтому мы внедряем его при помощи собственного
**токена**. Предыдущие примеры показали нам внедрение зависимости, основанное на объявлении зависимости в конструкторе
класса. Больше информации о собственных поставщиках и токенах, связанных с ними, доступно [здесь]().

## Внедрение на основе поля

Техника, которую мы использовали до этого момента, называется внедрением на основе конструктора, так как поставщики в
таком случае внедряются через конструктор. В особых случаях может быть полезно внедрение на основе свойств. Например,
если ваш класс верхнего уровня зависит от одного или нескольких поставщиков, то передача их наверх при помощи
вызова `super()` в подклассах может быть очень утомительной. Чтобы избежать этого, вы можете использовать
декоратор `@Inject()` на уровне поля.

```typescript
import { Inject, Injectable } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
    @Inject('HTTP_OPTIONS')
    private readonly httpClient: T;
}
```

> [!WARNING] Если ваш класс не расширяет другого поставщика, то вам следует отдать предпочтение внедрению **на основе конструктора**.

## Регистрация поставщика

Теперь, когда мы определили поставщика (`CatsService`) и у нас есть пользователь данного сервиса (`CatsController`), нам
необходимо зарегистрировать сервис в **Nest** чтобы мы он смог внедрить его. Мы сделаем это при помощи редактирования
файла модуля (`app.module.ts`) и добавления сервиса в массив `providers` в декораторе `@Module()`.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class AppModule {}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class AppModule {}
```

<!-- tabs:end -->


Теперь **Nest** сможет разрешить зависимости `CatsController`.

Вот как выглядит наша структура директорий на данный момент:

[//]: # (@formatter:off)

```
src/
├── cats/
│   ├── dto/
│   │   └── create-cat.dto.ts
│   ├── interfaces/
│   │   └── cat.interface.ts
│   ├── cats.controller.ts
│   └── cats.service.ts
├── app.module.ts
└── main.ts
```

[//]: # (@formatter:on)

## Ручное создание экземпляра

До текущего момента, мы говорили о том, как **Nest** автоматически обрабатывает большинство нюансов разрешения
зависимостей. При определённых обстоятельствах, вам может понадобиться выйти за пределы встроенной системы внедрения
зависимости и вручную извлекать или создавать поставщиков. Ниже мы кратко обсудим две данные темы.

Чтобы получить существующие экземпляры или создавать их динамически, вы можете использовать [Ссылки на модули]().

Чтобы получить поставщиков внутри функции `bootstrap()` (например, в автономном приложении без контроллеров, или, чтобы
использовать конфигурацию сервиса в процессе подготовки приложения к запуску) смотрите статью
об [Автономных приложениях]().
