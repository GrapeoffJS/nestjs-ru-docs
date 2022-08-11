# Контроллеры

Контроллеры ответственны за обработку входящих **запросов** и возвращение **ответов** клиенту.

![](https://docs.nestjs.com/assets/Controllers_1.png)

Цель контроллеров — получать запросы к приложению. Механизм **маршрутизации** определяет, какой контроллер какие запросы
получает. Зачастую, каждый контроллер имеет больше одного маршрута, а разные маршруты могут выполнять разные действия.

Чтобы создать базовый контроллер, мы используем классы и **декораторы**. Декораторы прикрепляют к классу необходимые
метаданные и позволяют **Nest** создать карту маршрутизации (связать запросы с соответствующими контроллерами).

> [!NOTE] Для быстрого создания **CRUD** контроллера с включённой валидацией для входящих данных, вы можете использовать **CRUD Generator** из **NestJS CLI**: `$ nest g resource [name]`.

## Маршрутизация

В примере ниже мы используем специальный декоратор — `@Controller()`. Таким декоратором должен быть помечен каждый
класс, что выполняет функции контроллера. Мы также определим необязательный префикс пути для контроллера котов.
Использование префикса пути в качестве аргумента, принимаемого декоратором `@Controller()`, позволяет нам сгруппировать
набор связанных маршрутов и сократить количество повторяющегося кода. Например, мы можем сгруппировать набор маршрутов,
которые управляют взаимодействием с клиентом при помощи маршрута `/customers`. В таком случае, мы можем сразу определить
данный префикс пути в декораторе `@Controller()` контроллера клиентов, чтобы не повторять каждый раз эту часть пути при
определении маршрутов в текущем файле.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get()
    findAll(): string {
        return 'This action returns all cats';
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get() findAll() {
        return 'This action returns all cats';
    }
}
```

<!-- tabs:end -->

> [!NOTE] Чтобы создать контроллер при помощи **CLI**, просто выполните такую команду: `$ nest g controller cats`.

Декоратор `@Get()`, который мы поместили перед реализацией метода `findAll()`, говорит **Nest** создать обработчик
данной конечной точки. Конечная точка соответствует методу **HTTP** запроса (в данном случае это `GET`) и пути маршрута.
Что такое путь маршрута? — Путь маршрута определяется совмещением (необязательного) префикса пути и любого пути,
указанного внутри декоратора метода. С тех пор как мы определили префикс пути для всех маршрутов (`cats`) и больше
никакой информации не добавляли, **Nest** прикрепит маршрут `GET /cats` к данному обработчику. Как уже упоминалось
ранее, путь к конечной точке строится из префикса пути, который указан в декораторе `@Controller()` **и** любой строчки,
указанной в декораторе, которым помечен метод-обработчик запроса. К примеру, префикс пути, совмещённый с
декоратором `@Get('profile')` даст нам такой путь к конечной точке: `GET /customers/profile`.

В том примере, как только `GET` был сделан запрос к данной конечной точке, **Nest** направляет запрос на обработку
методом `findAll()`, который определён пользователем. Заметьте, от имени метода ничего не зависит и оно может быть
**абсолютно любым**. Понятное дело, что мы должны объявить метод для обработки маршрута, но **Nest** не придаёт никакого
значения тому, какое имя вы выбрали для этого метода.

Этот метод вернёт `200-ый` статус код вместе с соответствующим ответом, который в данном случае является простой
строкой. Почему так происходит? Чтобы ответить на этот вопрос, мы сперва окунёмся в концепцию, согласно которой **Nest**
использует два **разных** способа для управления ответами.

[//]: # (@formatter:off)
| Способ                      | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Стандартный (рекомендуется) | Используя этот встроенный метод, когда обработчик запроса возвращает **JavaScript** объект или массив, он будет **автоматически** приведён к **JSON**. Однако когда обработчик возвращает **JavaScript** примитив (строка, число, булево-значение), **Nest** не будет пытаться преобразовать его в **JSON**. Это делает обработку ответов простой: просто верните значение и дальше **Nest** сам обо всём позаботится. <br><br> Более того `200-ый` **статус код** — код по умолчанию для всех ответов, кроме `POST` запросов, которые используют код `201`. Мы можем очень просто изменить данное поведение просто прикрепив декоратор `@HttpCode(...)` к методу-обработчику. |
| Специфичный для библиотеки  | Мы можем использовать специфичный для библиотеки [объект ответа](https://expressjs.com/en/api.html#res) (к примеру, Express) который может быть внедрён при помощи декоратора `@Res()` в параметрах метода (`findAll(@Res() response)`). С таким подходом у вас есть возможность использовать родные для библиотеки методы обработки ответа, которые предоставляет этот объект. Например, если вы используете Express, вы можете отправить код ответа таким образом: `response.status(200).send()`.                                                                                                                                                                            |

[//]: # (@formatter:on)

> [!WARNING]
> **Nest** определяет, что обработчик использует `@Res()` или `@Next()` и их использование указывает на то, что вы выбрали специфичный для библиотеки способ. Если одновременно используются оба подхода, стандартный подход **автоматически выключается** для этого маршрута и больше не будет работать должным образом. Если вы хотите использовать оба подхода одновременно (к примеру, вы внедряете объект ответа, только чтобы вернуть cookies или установить заголовки ответа, но хотите, чтобы **Nest** всё ещё сам управлял ответом), то в таком случае вам нужно присвоить значение `true` параметру `passthrough` в декораторе `@Res({ passthrough: true })`.

## Объект запроса

Обработчики запросов часто требуют доступ к информации о **запросе** клиента. **Nest** предоставляет доступ
к [объекту запроса](https://expressjs.com/en/api.html#req) используемой платформы (по умолчанию — Express). Мы можем
получить к нему доступ применив декоратор `Req()` к одному из параметров обработчика.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
    @Get()
    findAll(@Req() request: Request): string {
        return 'This action returns all cats';
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Bind, Get, Req } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get()
    @Bind(Req()) findAll(request) {
        return 'This action returns all cats';
    }
}
```

<!-- tabs:end -->


> [!NOTE] Чтобы воспользоваться преимуществами типизации Express (как это показано выше), установите пакет `@types/express` как зависимость для разработки.

Объект запроса представляет собой **HTTP** запрос и имеет свойства для строки запроса, параметров, заголовков и тела (
подробнее [здесь](https://expressjs.com/en/api.html#req)). В большинстве случаев нет необходимости доставать эти
свойства вручную. Вместо этого, мы можем использовать специально выделенные декораторы, вроде `@Body()` или `@Query()`,
которые доступны прямо из коробки. Ниже перечислены доступные декораторы и то что они представляют внутри своей
платформы.

[//]: # (@formatter:off)
| Декоратор                 | Внедряемое значение                  |
|---------------------------|--------------------------------------|
| `@Request(), @Req()`      | `req`                                |
| `@Response(), @Res()*`    | `res`                                |
| `@Next()`                 | `next`                               |
| `@Session()`              | `req.session`                        |
| `@Param(key?: string)`    | `req.params` / `req.params[key]`     |
| `@Body(key?: string)`     | `req.body` / `req.body[key]`         |
| `@Query(key?: string)`    | `req.query` / `req.query[key]`       |
| `@Headers(name?: string)` | `req.headers` / `req.headers[name]`  |
| `@Ip()`                   | `req.ip`                             |
| `@HostParam()`            | `req.hosts`                          |

[//]: # (@formatter:on)

*&nbsp;Для совместимости с типами базовых **HTTP** платформ (вроде Express или Fastify) **Nest** предоставляет
декораторы `@Res()` и `@Response()`. Декоратор `@Res()` попросту является сокращением для `@Response()`. Оба из них дают
вам доступ к объекту ответа используемой платформы. Когда вы используете их, вам также следует импортировать типы для
данной платформы (допустим, `@types/express`) чтобы воспользоваться всеми её преимуществами. Заметьте, что когда вы
внедряете объект ответа при помощи `@Res()` или `@Response()` в параметры метода-обработчика, вы переводите
**Nest** в **специфичный для используемой платформы режим** в контексте данного обработчика, после чего уже **ВЫ**
становитесь ответственным за управление ответом. Если вы решили пойти таким путём, то вы берёте на себя обязательство
вернуть какой-либо ответ, вызвав соответствующий метод у объекта ответа (к примеру `res.json(...)` or `res.send(...)`),
в противном случае, ваш **HTTP** просто зависнет.

> [!NOTE] Чтобы узнать, как создавать свои собственные декораторы, загляните [сюда]().

## Ресурсы

Ранее, мы определили конечную точку для доступа к ресурсу с котиками (`GET` маршрут). Также, мы обычно хотим
предоставлять пользователям конечную точку, которая создаёт новые записи. Давайте создадим обработчик `POST` запросов,
чтобы это реализовать.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Post()
    create(): string {
        return 'This action adds a new cat';
    }

    @Get()
    findAll(): string {
        return 'This action returns all cats';
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Post() create() {
        return 'This action adds a new cat';
    }

    @Get() findAll() {
        return 'This action returns all cats';
    }
}
```

<!-- tabs:end -->


Как видите, это очень просто. **Nest** предоставляет декораторы для всех стандартных **HTTP** методов: `@Get()`
, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, и `@Head()`, а также `@All()`, который позволяет
определить обработчик, реагирующий на все типы запросов.

## Шаблоны маршрутов

Маршруты, основанные на шаблонах также поддерживаются. Например, символ звёздочка используется как подстановочный знак,
и представляет собой абсолютно любую последовательность символов.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller()
class SomeController {
    @Get('ab*cd')
    findAll() {
        return 'This route uses a wildcard';
    }
}
```

Последовательности вроде `abcd` `ab_cd` `abecd` и даже `abjjfwfjwfkwnwcnwkneccd` будут соответствовать шаблону `ab*cd`.
Также, такие символы, как `?`, `+`, `*`, и `()` тоже могут использоваться при описании пути к конечной точке.
Дефис (`-`) и точка (`.`) не являются шаблонными и интерпретируются как обычные символы.

## Статус код

Как упоминалось ранее, **код ответа** по стандарту всегда `200`, исключение лишь для **POST** запросов, где об успехе
сообщается кодом `201`. Но мы всегда можем очень просто поменять такое поведение при помощи добавления
декоратора `@HttpCode(...)` к обработчику маршрута.

```typescript
import { Controller, HttpCode, Post } from '@nestjs/common';

@Controller()
class SomeController {
    @Post()
    @HttpCode(204)
    create() {
        return 'This action adds a new cat';
    }
}
```

Довольно часто, ваши обработчики должны возвращать различные статус коды в зависимости от ситуации. В таком случае, вы
можете использовать объект **ответа** используемой платформы (внедряется при помощи `@Res()`) или, в случае ошибки,
выбросить исключение.

## Заголовки

Чтобы добавить к запросу свой заголовок, вы можете использовать либо декоратор `@Header()`, либо объект ответа
используемой платформы (и напрямую вызвать у него соответсвующий метод: `res.header()`).

```typescript
import { Controller, Header, Post } from '@nestjs/common';

@Controller()
class SomeController {
    @Post()
    @Header('Cache-Control', 'none')
    create() {
        return 'This action adds a new cat';
    }
}
```

## Перенаправление

Чтобы перенаправить запрос на определённый **URL**, вы можете использовать декоратор `@Redirect()` или объект ответа
используемой платформы (и напрямую вызвать у него соответсвующий метод: `res.redirect()`).

`@Redirect()` принимает в себя два необязательных аргумента, `url` и `statudCode`. По умолчанию, значение `statusCode`
— `302` (`Ресурс найден`), если другое значение не было присвоено.

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

Иногда вам может понадобиться сконструировать **HTTP** статус код или перенаправить динамически. Для этого верните в
качестве ответа объект с такой структурой:

```
{
    "url": string,
    "statusCode": number
}
```

Возвращённые значения перезапишут те данные, которые вы передали в декораторе `@Redirect()`. Например:

```typescript
import { Controller, Redirect, Get, Query } from '@nestjs/common';

@Controller()
class SomeController {
    @Get('docs')
    @Redirect('https://docs.nestjs.com', 302)
    getDocs(@Query('version') version) {
        if (version && version === '5') {
            return { url: 'https://docs.nestjs.com/v5/' };
        }
    }
}
```

## Параметры маршрута

Маршруты со статическими путями не подходят нам, когда необходимо получать какие-либо **динамические данные** как часть
запроса (например, `GET /cats/1` чтобы получить котика с id `1`). Чтобы реализовать параметризованные маршруты, нам
нужно добавить специальный **токены** параметров в путь нашего маршрута, который будет связан с динамическими данными,
передаваемыми в него во время запроса. Пример использования данного параметра внутри декоратора `@Get` продемонстрирован
ниже. Параметр маршрута, объявленный таким образом может быть получен при помощи декоратора `@Param()`, который должен
быть добавлен в сигнатуру метода.

<!-- tabs:start -->

#### **TypeScript**

```typescript
@Controller()
class SomeController {
    @Get(':id')
    findOne(@Param() params): string {
        console.log(params.id);
        return `This action returns a #${params.id} cat`;
    }
}
```

#### **JavaScript**

```js
@Controller() class SomeController {
    @Get(':id')
    @Bind(Param()) findOne(params) {
        console.log(params.id);
        return `This action returns a #${params.id} cat`;
    }
}
```

<!-- tabs:end -->

`@Param()` используется чтобы помечать параметры метода (`params` в примере выше) и делает параметры **маршрута**
доступными в качестве свойств этого декорированного параметра метода в теле данного метода. Из примера видно, что мы
получаем доступ к `id` с помощью обращению к полю с таким именем. Вы также можете получить конкретный параметр из
запроса по его токену, передав его в качестве аргумента в декораторе `@Param()`.

<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Controller, Get, Param } from '@nestjs/common';

@Controller()
class SomeController {
    @Get(':id')
    findOne(@Param('id') id: string): string {
        return `This action returns a #${id} cat`;
    }
}
```

#### **JavaScript**

```js
import { Controller, Get, Param } from '@nestjs/common';

@Controller() class SomeController {
    @Get(':id')
    @Bind(Param('id')) findOne(id) {
        return `This action returns a #${id} cat`;
    }
}
```

<!-- tabs:end -->

## Поддоменная маршрутизация

Декоратор `@Controller` может принимать в себя параметр `host` чтобы требовать от **HTTP** хоста входящих запросов
совпадение переданному значению.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller({ host: 'admin.example.com' })
export class AdminController {
    @Get()
    index(): string {
        return 'Admin page';
    }
}
```

> [!ATTENTION]
> Поскольку **Fastify** не поддерживает вложенные маршрутизаторы при использовании поддоменов, вам стоит использовать адаптер Express для этих целей.

Подобно пути маршрута, поле `host` может использовать токены для получения динамического значения в имени хоста. В таком
случае, данные параметры могут быть получены при помощи декоратора `@HostParam()`, который должен быть добавлен в
параметры метода.

```typescript
import { Controller, Get, HostParam } from '@nestjs/common'

@Controller({ host: ':account.example.com' })
export class AccountController {
    @Get()
    getInfo(@HostParam('account') account: string) {
        return account;
    }
}
```

## Области внедрения

Для людей, пришедших из других языков программирования может быть неожиданностью узнать, что в **Nest** почти всё
распределяется между входящими запросами. У нас есть набор соединений к базе данных, сервисы-одиночки (речь идёт о
singleton)
с глобальным состоянием и так далее. Запомните, **Node.js** не следует многопоточной модели запрос/ответ без сохранения
состояния (ориг. «The request/response Multi-Threaded Stateless Model»), где каждый запрос обрабатывается в отдельном
потоке. Следовательно, использование экземпляров одиночек (ориг. singleton instances) полностью **безопасна** для наших
приложений.

Однако есть крайние случаи, когда вам может понадобиться контроллер, чьё время жизни будет зависеть от запроса,
например, кеширование каждого запроса в приложении **GraphQL**, отслеживание запросов
или [мультиарендность](https://ru.wikipedia.org/wiki/%D0%9C%D1%83%D0%BB%D1%8C%D1%82%D0%B8%D0%B0%D1%80%D0%B5%D0%BD%D0%B4%D0%BD%D0%BE%D1%81%D1%82%D1%8C)
. Почитать об управлении областями внедрения вы можете [здесь]().

## Асинхронность

Мы любим современный JavaScript и мы знаем, что извлечение данных в основном **асинхронно**. Именно поэтому **Nest**
имеет внутри себя отличную поддержку асинхронных функций.

> [!NOTE] Узнать больше о `async` / `await` вы можете [тут](https://kamilmysliwiec.com/typescript-2-1-introduction-async-await).

Каждая асинхронная функция должна возвращать `Promise`. Это означает, что вы можете вернуть отложенное значение,
которое **Nest** обработает самостоятельно. Давайте взглянем на этот пример:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get } from '@nestjs/common';

@Controller()
class CatsController {
    @Get()
    async findAll(): Promise<any[]> {
        return [];
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get } from '@nestjs/common';

@Controller() class CatsController {
    @Get()
    async findAll() {
        return [];
    }
}
```

<!-- tabs:end -->

Данный код совершенно корректен. Более того, обработчики маршрутов в **Nest** даже ещё мощнее, чем вы думаете — они
могут возвращать наблюдаемые потоки RxJS
(RxJS [observable streams](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)).
**Nest** самостоятельно подпишется на них и заберёт последнее возвращенное значение (как только поток будет завершён).

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Controller()
class CatsController {
    @Get()
    findAll(): Observable<any[]> {
        return of([]);
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Controller() class CatsController {
    @Get() findAll() {
        return of([]);
    }
}
```

<!-- tabs:end -->

Оба приведённых подхода отлично работают и вы можете использовать всё, что соответствует вашим требованиям.

## Получение данных из запроса

Наш предыдущий пример обработчика **POST** запросов не принимал никаких данных. Давайте поменяем это поведение при
помощи добавления в него декоратора `@Body()`.

Но сначала (если вы используете TypeScript), нам нужно определить объект передачи данных (DTO; англ. Data Transfer
Object). DTO — это объект, который определяет, как данные будут переданы по сети. Мы можем сделать DTO при помощи
интерфейсов **TypeScript** или при помощи классов. Что примечательно, мы рекомендуем вам использовать именно **классы**.
Знаете почему? — Классы являются частью стандарта _ES6_ языка **JavaScript**, следовательно, на этапе транспиляции
**TypeScript** они будут сохранены как реальные объекты. С другой стороны, так как интерфейсы **TypeScript** удаляются в
процессе транспиляции, **Nest** не может получить к ним доступ во время работы программы. И эта разница имеет значение,
ведь такие вещи, как **Конвейеры** (ориг. Pipes) открывают вам дополнительные возможности, когда у них есть доступ к
метатипу переменной в процессе выполнения программы, что в случае с интерфейсами, просто невозможно.

Итак, давайте создадим наш класс `CreateCatDto`:

**_create-cat.dto.ts_**

```typescript
export class CreateCatDto {
    name: string;
    age: number;
    breed: string;
}
```

У него всего три основных свойства. После этого, мы можем использовать наш новоиспечённый **DTO**
внутри `CatsController`:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Body, Controller, Post } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto'

@Controller()
class CatsController {
    @Post()
    async create(@Body() createCatDto: CreateCatDto) {
        return 'This action adds a new cat';
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Body, Controller, Post } from '@nestjs/common';

@Controller() class CatsController {
    @Post()
    @Bind(Body())
    async create(createCatDto) {
        return 'This action adds a new cat';
    }
}
```

<!-- tabs:end -->

> [!NOTE]
> `ValidationPipe` может отфильтровывать свойства, которые обработчик маршрута не ожидает получить. Простыми словами, те свойства, которых нет в **DTO**, будут удалены из результирующего объекта. В примере с `CreateCatDto`, разрешёнными для отправки свойствами являются `name`, `age`, и `breed`. Подробнее о валидации вы можете прочитать [здесь]().

## Обработка ошибок

У нас есть [отдельная глава](), посвящённая обработке ошибок (то есть работе с исключениями).

## Пример полного ресурса

Ниже приведён пример использования доступных декораторов, для того чтобы создать базовый контроллер. Этот контроллер
предоставляет набор методов для доступа и управления внутренними данными.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Body, Controller, Delete, Get, Param, Post, Put, Query } from '@nestjs/common';
import { CreateCatDto, ListAllEntities, UpdateCatDto } from './dto';

@Controller('cats')
export class CatsController {
    @Post()
    create(@Body() createCatDto: CreateCatDto) {
        return 'This action adds a new cat';
    }

    @Get()
    findAll(@Query() query: ListAllEntities) {
        return `This action returns all cats (limit: ${query.limit} items)`;
    }

    @Get(':id')
    findOne(@Param('id') id: string) {
        return `This action returns a #${id} cat`;
    }

    @Put(':id')
    update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
        return `This action updates a #${id} cat`;
    }

    @Delete(':id')
    remove(@Param('id') id: string) {
        return `This action removes a #${id} cat`;
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get, Query, Post, Body, Put, Param, Delete, Bind } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Post()
    @Bind(Body()) create(createCatDto) {
        return 'This action adds a new cat';
    }

    @Get()
    @Bind(Query()) findAll(query) {
        console.log(query);
        return `This action returns all cats (limit: ${query.limit} items)`;
    }

    @Get(':id')
    @Bind(Param('id')) findOne(id) {
        return `This action returns a #${id} cat`;
    }

    @Put(':id')
    @Bind(Param('id'), Body()) update(id, updateCatDto) {
        return `This action updates a #${id} cat`;
    }

    @Delete(':id')
    @Bind(Param('id')) remove(id) {
        return `This action removes a #${id} cat`;
    }
}
```

<!-- tabs:end -->

> [!NOTE]
> **Nest CLI** предоставляет генератор, который за вас создаёт весь **шаблонный код**, чтобы помочь вам не делать всего самостоятельно, что значительно упрощает жизнь разработчикам.
> Подробнее об этом [здесь]().

## Подготовка к запуску

Вот мы и завершили создание нашего контроллера, но **Nest** всё ещё не знает, что в нашем коде существует
некий `CatsController` и поэтому не сделает его экземпляр.

Контроллеры всегда привязаны к модулю, именно поэтому мы добавляем массив `controllers` в декораторе `@Module()`. Так
как мы не создавали никаких модулей, кроме корневого `AppModule`, мы используем его, чтобы зарегистрировать в
нём `CatsController`.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
    controllers: [CatsController],
})
export class AppModule {}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
    controllers: [CatsController],
})
export class AppModule {}
```

<!-- tabs:end -->

Мы прикрепили метаданные к модулю, используя декоратор `@Module`, и теперь **Nest** лего может определить, какие
контроллеры необходимо использовать.

## Специфичный для базовой платформы подход

До сих пор мы обсуждали стандартный способ управления ответами в **Nest**. Второй способ —
использовать [объект ответа](https://expressjs.com/en/api.html#res) базовой платформы. Чтобы внедрить объект ответа, нам
нужно использовать декоратор `@Res()`. Чтобы показать вам различия, давайте поменяем реализацию `CatsController` на
такую:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.controller.ts */

import { Controller, Get, HttpStatus, Post, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
    @Post()
    create(@Res() res: Response) {
        res.status(HttpStatus.CREATED).send();
    }

    @Get()
    findAll(@Res() res: Response) {
        res.status(HttpStatus.OK).json([]);
    }
}
```

#### **JavaScript**

```js
/* cats.controller.js */

import { Controller, Get, Post, Bind, Res, Body, HttpStatus } from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Post()
    @Bind(Res(), Body()) create(res, createCatDto) {
        res.status(HttpStatus.CREATED).send();
    }

    @Get()
    @Bind(Res()) findAll(res) {
        res.status(HttpStatus.OK).json([]);
    }
}
```

<!-- tabs:end -->

Хоть такой способ и работает, и фактически обеспечивает бóльшую гибкость в некоторых случаях, предоставляя полный
контроль над объектом ответа (управление заголовками, специфичную для базовой платформы функциональность так далее), вам
следует использовать его осторожно. В основном, такой подход куда менее чист имеет некоторые недостатки. Главный
недостаток такого подхода — ваш код становится зависимым от платформы (поскольку базовые платформы могут иметь разные
**API** для объекта ответа), а также его становится труднее тестировать (вам будет нужно подделывать объект ответа, и
так далее).

Кроме того, в примере сверху вы теряете совместимость с функциями **Nest**, которые зависят от стандартной обработки
ответов **Nest**, такими как перехватчики и декораторы `@HttpCode()` / `@Header()`. Чтобы это исправить, вы можете
активировать свойство `passthrough`, как в этом примере:

<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Controller, Get, HttpStatus, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller()
export class SomeController {
    @Get()
    findAll(@Res({ passthrough: true }) res: Response) {
        res.status(HttpStatus.OK);
        return [];
    }
}
```

#### **JavaScript**

```js
import { Controller, Get, HttpStatus, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller()
export class SomeController {
    @Get()
    @Bind(Res({ passthrough: true })) findAll(res) {
        res.status(HttpStatus.OK);
        return [];
    }
}
```

<!-- tabs:end -->

Итак, теперь вы можете взаимодействовать с родным для платформы объектом ответа (к примеру, задать cookies или
заголовки, которые будут зависеть от конкретных условий), но остальную работу предоставить фреймворку.
