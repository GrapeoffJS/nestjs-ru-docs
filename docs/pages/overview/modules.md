# Модули

Модуль — это класс, помеченный декоратором `@Module()`. Данный декоратор предоставляет метаданные, которые **Nest**
использует для составления структуры приложения.

![](https://docs.nestjs.com/assets/Modules_1.png)

Каждое приложения имеет как минимум один модуль, **корневой модуль**. Корневой модуль является отправной точкой для
постройки графа приложения — внутренней структуры данных, которую **Nest** использует для разрешения отношений и
зависимостей между модулями и поставщиками. Хоть маленькие приложения теоретически могут иметь лишь один модуль, это не
является типичным случаем. Мы хотим подчеркнуть, что модули настоятельно рекомендуются как эффективный способ
организации ваших компонентов. Таким образом, для большинства приложений итоговая архитектура будет использовать
несколько модулей, где каждый инкапсулирует тесно связанный с ним набор ответственностей.

Декоратор `@Module()` принимает в себя объект, чьи свойства описывают модуль:

[//]: # (@formatter:off)

| Свойство      | Описание                                                                                                                                                                                                                             |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `providers`   | Поставщики, экземпляры которых будут созданы системой внедрения **Nest**, после чего они могут быть использованы в любом месте в контексте данного модуля.                                                                           |
| `controllers` | Набор контроллеров, определённых в текущем модуле, экземпляры которых необходимо создать.                                                                                                                                            |
| `imports`     | Список импортированных модулей, которые экспортируют поставщиков, которые необходимы в данном модуле.                                                                                                                                |
| `exports`     | Множество поставщиков, которые предоставляются данным модулем и должны быть доступны в других модулях, которые импортируют данный модуль. Вы можете использовать либо самого поставщика, либо только его токен (свойство `provide`). |

[//]: # (@formatter:on)

Модуль **инкапсулирует** в себе поставщиков по умолчанию. Это означает, что нельзя внедрить поставщика если он не
является частью данного модуля или не экспортирован из импортируемых модулей. Таким образом, экспортированные из модуля
поставщики могут быть рассмотрены как публичный **API** этого модуля.

## Функциональные модули

`CatsController` и `CatsService` принадлежат одной и той же доменной области приложения. Поскольку они тесно связаны
между собой, есть смысл поместить их в функциональный модуль. Функциональный модуль попросту объединяет код, относящийся
к конкретной задаче, сохраняя структурированность кода и устанавливая чёткие границы. Это помогает нам управлять
сложностью и разрабатывать согласно принципам
[SOLID](https://inlnk.ru/l0E0eX), особенно если размер приложения и/или команды разработчиков растёт.

Чтобы продемонстрировать это, мы создадим `CatsModule`.

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats/cats.module.ts */

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class CatsModule {}
```

#### **JavaScript**

```js
/* cats/cats.module.js */

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class CatsModule {}
```

<!-- tabs:end -->

> [!NOTE]
> Чтобы создать модуль используя **CLI**, запустите данную команду: `$ nest g module cats`

Сверху мы определили `CatsModule` в файле `cats.module.ts`, а также перенесли всё, относящееся к этому модулю в
директорию `cats`. Последнее, что нам нужно сделать это импортировать данный модуль в корневой модуль (`AppModule`,
который находится в файле `app.module.ts`).

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* app.module.ts */

import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule {}
```

#### **JavaScript**

```js
/* app.module.js */

import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
    imports: [CatsModule],
})
export class AppModule {}
```

<!-- tabs:end -->

Теперь наша структура директорий выглядит так:

```
src/
├── cats/
│   ├── dto/
│   │   └── create-cat.dto.ts
│   ├── interfaces/
│   │   └── cat-interface.ts
│   ├── cats.controller.ts
│   ├── cats.module.ts
│   └── cats.service.ts
├── app.module.ts
└── main.ts
```

## Общие модули

В **Nest**, модули являются одиночками (singleton) по умолчанию и, следовательно, вы легко можете использовать один и
тот же экземпляр любого провайдера внутри нескольких модулей.

![](https://docs.nestjs.com/assets/Shared_Module_1.png)

Каждый модуль автоматически является общим. После создания он может быть переиспользован в любым другим модулем. Давайте
представим, что мы хотим использовать экземпляр `CatsService` в нескольких других модулях. Чтобы провернуть это, вначале
мы должны экспортировать `CatsService` при помощи добавления его массив `exports`, как это показано выше:

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.module.ts */

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
    exports: [CatsService]
})
export class CatsModule {}
```

#### **JavaScript**

```js
/* cats.module.js */

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
    exports: [CatsService]
})
export class CatsModule {}
```

<!-- tabs:end -->

Теперь каждый модуль, который импортирует в себя `CatsModule` имеет доступ к `CatsService` и будет использовать один и
тот же экземпляр вместе модулями, которые также импортируют `CatsModule`.

## Переэкспорт модулей

Как вы уже увидели выше, модули могут экспортировать собственных поставщиков. В дополнение к этому, они могут
переэкспортировать модули, которые импортируют. В примере ниже, `CommonModule` импортирован и экспортирован
из `CoreModule`, что делает его доступным другим модулям, которые импортируют его.

```typescript
@Module({
    imports: [CommonModule],
    exports: [CommonModule],
})
export class CoreModule {}
```

## Внедрение зависимости

Модуль также может внедрять поставщиков (например, в целях конфигурации).

<!-- tabs:start -->

#### **TypeScript**

```typescript
/* cats.module.ts */

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
export class CatsModule {
    constructor(private catsService: CatsService) {}
}

```

#### **JavaScript**

```js
/* cats.module.js */

import { Module, Dependencies } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
    controllers: [CatsController],
    providers: [CatsService],
})
@Dependencies(CatsService)
export class CatsModule {
    constructor(catsService) {
        this.catsService = catsService;
    }
}
```

<!-- tabs:end -->

Однако, сами классы модулей не могут быть внедрены как поставщики из-за [круговой зависимости]().

## Глобальные модули

Если вам везде нужно внедрять один и тот же набор модулей, это может стать утомительным. В отличие от **Nest**,
поставщики [Angular](https://angular.io/) зарегистрированы в глобальной области видимости. Как только они определены,
они становятся доступными везде. Однако **Nest** инкапсулирует поставщиков внутри области видимости модуля. Вы не можете
использовать поставщиков модуля в другом месте без предварительного импорта инкапсулирующего модуля.

Когда вы хотите определить список поставщиков, которые должны быть доступны везде (вроде помощников, подключений к базе
данных и т.д.), сделайте модуль **глобальным** с помощью декоратора `@Global()`

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
    controllers: [CatsController],
    providers: [CatsService],
    exports: [CatsService],
})
export class CatsModule {}
```

Декоратор `@Global()` делает этот модуль видимым глобальным. Глобальные модули должны быть зарегистрированы лишь
**один раз**, в основном в корневом или основном (ориг. core) модуле. В примере выше, поставщик `CatsService` будет
доступен повсеместно и модули, которые будут его внедрять, не должны импортировать `CatsModule`.

> [!NOTE]
> Делать всё глобальным — не самое хорошее архитектурное решение. Глобальные модули созданы для уменьшения количества необходимого шаблонного кода. Массив `imports` в основном является предпочтительным путём сделать API модуля доступным для потребителей.

## Динамические модули

Система модулей **Nest** включает в себя мощную концепцию под названием **динамические модули**. Она позволяет вам легко
создавать настраиваемые модули, которые могут конфигурировать поставщиков динамически. Динамические в нужной степени
объяснены [здесь](). В этой главе мы дадим лишь краткий обзор, на чём и закончим введение в модули.

Следующий код является примером определения динамического модуля для `DatabaseModule`:

<!-- tabs:start -->

#### **TypeScript**

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
    providers: [Connection],
})
export class DatabaseModule {
    static forRoot(entities = [], options?): DynamicModule {
        const providers = createDatabaseProviders(options, entities);
        return {
            module: DatabaseModule,
            providers: providers,
            exports: providers,
        };
    }
}
```

#### **JavaScript**

```js
import { Module } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
    providers: [Connection],
})
export class DatabaseModule {
    static forRoot(entities = [], options) {
        const providers = createDatabaseProviders(options, entities);
        return {
            module: DatabaseModule,
            providers: providers,
            exports: providers,
        };
    }
}
```

<!-- tabs:end -->

> [!NOTE] Метод `forRoot()` может возвращать динамический модуль как синхронно, так и асинхронно (т.е. при помощи `Promise`).

Данный модуль создаёт поставщика `Connection` по умолчанию (в метаданных декоратора `@Module()`), но также в зависимости
от объектов сущностей и настроек, переданных в метод `forRoot()`, предоставляет набор поставщиков, например,
репозитории. Заметьте, что свойства, возвращаемые динамическим модулем, расширяют (а не переопределяют) метаданные
базового модуля, которые определены в декораторе `@Module()`. Таким образом из модуля экспортируются как статически
объявленный поставщик, так и динамически сгенерированные провайдеры репозитория.

Если вы хотите зарегистрировать модуль в глобальной области видимости, установите свойство `global` в значение `true`.

```
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> [!WARNING] Как уже упомянуто выше, **не стоит делать глобальным всё подряд**.

`DatabaseModule` может быть импортирован и настроен таким образом:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
    imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

Если вы хотите переэкспортировать динамический модуль, вы можете опустить вызов метода `forRoot()` в массиве `exports`:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
    imports: [DatabaseModule.forRoot([User])],
    exports: [DatabaseModule],
})
export class AppModule {}
```

Глава о [Динамических модулях]() отлично покрывает данную тему, а также
предоставляет [рабочий пример](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules).

> [!NOTE]
> Научитесь создавать отлично настраиваемые динамические модули с использованием `ConfigurableModuleBuilder` в [данной главе]().
