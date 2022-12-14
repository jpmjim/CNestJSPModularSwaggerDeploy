# CNestJSPModularSwaggerDeploy
Curso de NestJS: Programación Modular, Documentación con Swagger y Deploy

## Configuración de la aplicación
  Temas que verás dentro del curso NestJS

  En este curso aprenderás sobre:

  - Modularización de un proyecto NestJS.
  - Servicios e inyección de dependencias.
  - Manejo de ambientes y variables de entorno.
  - Documentación del proyecto con Swagger.
  - Despliegue de una aplicación en Heroku.

## Encapsular lógica en módulos
  Las aplicaciones profesionales que se desarrollan con NestJS se realizan de forma modularizada para dividir el código fuente de forma lógica y que el proyecto sea más escalable y comprensible.

  ### Cómo hacer la modularización de un proyecto en NestJS
  Para modularizar una aplicación, el CLI de NestJS trae consigo la posibilidad de autogenerar módulos con el comando <code>nest generate module "module-name"</code> o en su forma corta <code>nest g mo "module-name"</code>.

  Los módulos son simples clases que utilizan el decorador @Module() para importar todo lo que construyan al mismo.
  ```typescript
  import { Module } from '@nestjs/common';

  @Module({
    imports: [],             // Importación de otros módulos
    controllers: [],         // Importación de controladores
    providers: [],           // Importación de servicios
  })
  export class PruebaModule {}
  ```
  De esta manera, un módulo agrupará un conjunto de controladores y servicios, además de importar otros módulos.

  A partir de aquí, tu aplicación podría tener un módulo para usuarios, otro para productos, otro para comentarios, etc. Crea tantos módulos como tu aplicación necesite.

  **Ejemplo de modularización de una aplicación**

  En las siguientes imágenes te mostramos como debería quedar organizada la aplicación:
  ![](https://i.imgur.com/ozsw0eb.png)

  Donde los módulos deberían quedar así:
  ```typescript
  // src/products/products.module.ts
  import { Module } from '@nestjs/common';

  import { ProductsController } from './controllers/products.controller';
  import { BrandsController } from './controllers/brands.controller';
  import { CategoriesController } from './controllers/categories.controller';
  import { ProductsService } from './services/products.service';
  import { BrandsService } from './services/brands.service';
  import { CategoriesService } from './services/categories.service';

  @Module({
    controllers: [ProductsController, CategoriesController, BrandsController],
    providers: [ProductsService, BrandsService, CategoriesService],
    exports: [ProductsService],
  })
  export class ProductsModule {}
  ```
  ```typescript
  // src/users/users.module.ts
  import { Module } from '@nestjs/common';

  import { CustomerController } from './controllers/customers.controller';
  import { CustomersService } from './services/customers.service';
  import { UsersController } from './controllers/users.controller';
  import { UsersService } from './services/users.service';

  import { ProductsModule } from '../products/products.module';

  @Module({
    imports: [ProductsModule],
    controllers: [CustomerController, UsersController],
    providers: [CustomersService, UsersService],
  })
  export class UsersModule {}
  ```
  ```typescript
  // src/app.module.ts
  import { Module } from '@nestjs/common';
  import { AppController } from './app.controller';
  import { AppService } from './app.service';
  import { UsersModule } from './users/users.module';
  import { ProductsModule } from './products/products.module';

  @Module({
    imports: [UsersModule, ProductsModule],
    controllers: [AppController],
    providers: [AppService],
  })
  export class AppModule {}
  ```

## Overview del proyecto: PlatziStore
  A lo largo de este curso, continuarás trabajando con el proyecto iniciado en el [Curso de Backend con NestJS](https://platzi.com/cursos/nestjs/). Te recomendamos tomar ese curso antes de continuar con este. Allí, se desarrolló una API Rest para el manejo de un catálogo de productos y en este curso llevaremos esa aplicación un paso más allá.

  Prepárate para desarrollar tu primera API con NestJS de forma profesional.

  Recuerda en la rama [2-step](https://github.com/platzi/nestjs-modular/tree/2-step) esta la solución en donde estan los controllers, servicios y dtos.
  
  [Organización para Insomnia](https://static.platzi.com/media/public/uploads/insomnia_2021-03-09_d2d933fc-d36a-44b3-a47d-fdce06e83f15.json)


## Interacción entre módulos
  Dentro de un módulo, puedes tener la necesidad de utilizar un servicio que pertenece a otro módulo. Importar estos servicios en otros módulos requiere de un paso adicional.

  ### Importaciones de servicios compartidos
  Si tienes un **Módulo A** que posee un *Servicio A* y un segundo **Módulo B** requiere hacer uso de este, debes exportar el servicio para que otro módulo pueda utilizarlo.
  ```typescript
  // Módulo A
  import { ServiceA } from './service-A.service';

  @Module({
    providers: [ServiceA],
    exports: [ServiceA]
  })
  export class ModuleA {}
  ```
  ```typescript
  // Módulo B
  import { ServiceA } from './module-A/service-A.service';

  @Module({
    providers: [ServiceA]
  })
  export class ModuleB {}
  ```
  Debes indicar en la propiedad <code>exports</code> del decorador <code>@Module()</code> que un módulo es exportable para que otro módulo pueda importarlo en sus <code>providers</code>.

  De esta manera, evitas errores de compilación de tu aplicación que ocurren cuando importas servicios de otros módulos que no están siendo exportados correctamente.

  ***Ejemplo de interacción entre módulos***

  A continuación, podrás ver el código que necesitas para hacer que los módulos interactúen entre sí.
  ```typescript
  // src/users/entities/order.entity.ts
  import { User } from './user.entity';
  import { Product } from './../../products/entities/product.entity';

  export class Order { //  // 👈 new entity
    date: Date;
    user: User;
    products: Product[];
  }
  ```
  ```typescript
  // src/users/controllers/users.controller.ts
  @Get(':id/orders') //  👈 new endpoint
  getOrders(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.getOrderByUser(id);
  }
  ```
  ```typescript
  // src/users/services/users.service.ts
  ...
  import { Order } from '../entities/order.entity';
  import { ProductsService } from './../../products/services/products.service';

  @Injectable()
  export class UsersService {
    constructor(private productsService: ProductsService) {}
    ...

    getOrderByUser(id: number): Order { // 👈 new method
      const user = this.findOne(id);
      return {
        date: new Date(),
        user,
        products: this.productsService.findAll(),
      };
    }
  }
  ```
  ```typescript
  // src/products/products.module.ts
  import { Module } from '@nestjs/common';
  ....
  @Module({
    controllers: [ProductsController, CategoriesController, BrandsController],
    providers: [ProductsService, BrandsService, CategoriesService],
    exports: [ProductsService], // 👈 Export ProductsService
  })
  export class ProductsModule {}
  ```
  ```typescript
  // src/users/users.module.ts
  import { Module } from '@nestjs/common';
  ...
  import { ProductsModule } from '../products/products.module';

  @Module({
    imports: [ProductsModule], // 👈 Import ProductsModule
    controllers: [CustomerController, UsersController],
    providers: [CustomersService, UsersService],
  })
  export class UsersModule {}
  ```

## Entendiendo la inyección de dependencias
  Es muy sencillo crear un servicio en NestJS, inyectarlo en un componente y utilizar su lógica. A pesar de esto, siempre es recomendable entender cómo lo está haciendo y qué sucede por detrás en tu aplicación.

  ### Patrones de diseño en NestJS

  NestJS utiliza varios Patrones de Diseño para permitir que esto funcione. Te presentamos dos para tener en cuenta:

  ***Inyección de dependencias***

  Imagínate que tienes un Servicio A que utiliza el Servicio B y este a su vez utiliza el Servicio C. Si tuvieses que instanciar el Servicio A, primero deberías instanciar el C para poder instanciar el B y luego sí hacerlo con el A. Se vuelve confuso y poco escalable si en algún momento también tienes que instanciar el Servicio D o E.

  La inyección de dependencias llega para solucionar esto, resolver las dependencias de una clase por nosotros. Cuando instanciamos en el constructor el Servicio A, NestJS por detrás genera automáticamente la instancia del servicio B y C sin que nosotros nos tengamos que preocupar por estos.

  ***Singleton***

  La inyección de dependencias no es el único patrón de diseño que NestJS utiliza con sus servicios. También hace uso del patrón Singleton para crear una instancia única de cada servicio. Así es como, si tienes un servicio que se utiliza en N cantidad de componentes (u otros servicios) todos estos estarán utilizando la misma instancia del servicio, compartiendo el valor de sus variables y todo su estado.

  [Singleton en TypeScript](https://refactoring.guru/es/design-patterns/singleton/typescript/example)

  ### Precauciones utilizando servicios
  Un servicio puede ser importado en muchos componentes u otros servicios a la vez. Puedes inyectar la cantidad de servicio que quieras en un componente, siempre de una forma controlada y coherente.
  ![](https://static.platzi.com/media/user_upload/Circular%20dependency-0c7642ea-5281-4561-b20c-1bd97bfee9ba.jpg)
  Solo debes tener cuidado con las dependencias circulares. Cuando un servicio importa a otro y este al anterior. NestJS no sabrá cuál viene primero y tendrás un error al momento de compilar tu aplicación.

## useValue y useClass
  NestJS posee diferentes formas de inyectar servicios en un módulo según la necesidad. Exploremos algunas de ellas, sus diferencias y cuándo utilizarlas.

  ### Cómo hacer la inyección con “useClass”
  Cuando realizas un import de un servicio en un módulo:
  ```typescript
  import { AppService } from './app.service';

  @Module({
    providers: [AppService],
  })
  export class AppModule {}
  ```
  Internamente, NestJS realiza lo siguiente:
  ```typescript
  import { AppService } from './app.service';

  @Module({
    providers: [
      {
        provide: AppService,
        useClass: AppService
      }
    ]
  })
  export class AppModule {}
  ```
  Ambas sintaxis son equivalentes, **useClass** es el tipo de inyección por defecto. Básicamente, indica que un servicio debe utilizar X clase para funcionar. Si el día de mañana, por algún motivo en tu aplicación, el servicio AppService queda obsoleto y tienes que reemplazarlo por uno nuevo, puedes realizar lo siguiente:
  ```typescript
  import { AppService2 } from './app.service';

  @Module({
    providers: [
      {
        provide: AppService,
        useClass: AppService2
      }
    ]
  })
  export class AppModule {}
  ```
  De este modo, no tienes necesidad de cambiar el nombre **AppService** en todos los controladores donde se utiliza, este será reemplazado por la nueva versión del servicio.

  ### Cómo hacer la inyección con “useValue”
  Además de clases, puedes inyectar valores como un string o un número. **useValue** suele utilizarse para inyectar globalmente en tu aplicación la llave secreta de una API o alguna otra variable de entorno que tu app necesita.

  Para esto, simplemente inyecta el valor de una constante en el providers.
  ```typescript
  const API_KEY = '1324567890';

  @Module({
    providers: [
      {
        provide: 'API_KEY',
        useValue: API_KEY
      }
    ],
  })
  export class AppModule {}
  ```
  Importa este valor en los controladores u otros servicios donde se necesite de la siguiente manera:
  ```typescript
  import { Controller, Inject } from '@nestjs/common';

  @Controller()
  export class AppController {

    constructor(@Inject('API_KEY') private apiKey: string) {}
  }
  ```
  Ahora tienes a disposición el valor de este dato en tu controlador para utilizarlo en lo que necesites.

  **Cuadro de códigos para inyección de servicios**

  ```typescript
  // src/app.module.ts
  ...

  const API_KEY = '12345634';
  const API_KEY_PROD = 'PROD1212121SA';

  @Module({
    imports: [UsersModule, ProductsModule],
    controllers: [AppController],
    providers: [
      AppService,
      {
        provide: 'API_KEY',
        useValue: process.env.NODE_ENV === 'prod' ? API_KEY_PROD : API_KEY,
      },
    ],
  })
  export class AppModule {}
  ```

  ```typescript
  // src/app.service.ts
  import { Injectable, Inject } from '@nestjs/common';

  @Injectable()
  export class AppService {
    constructor(@Inject('API_KEY') private apiKey: string) {} // 👈 Inject API_KEY
    getHello(): string {
      return `Hello World! ${this.apiKey}`;
    }
  }
  ```

  ```typescript
  // src/app.controller.ts

  @Controller()
  export class AppController {

    @Get()
    getHello(): string { // 👈 new enpoint
      return this.appService.getHello();
    }

  }
  ```
  Corremos en la terminal el comando:
  ```bash
  NODE_ENV=prod npm run start:dev
  ```

## useFactory
  NestJS permite inyecciones de servicios o datos que necesiten de alguna petición HTTP o algún proceso **asíncrono**.(

  - [HTTP module](https://docs.nestjs.com/techniques/http-module#http-module)
  - **npm i --save @nestjs/axios** para la ultima version.

  ### Inyecciones Asíncronas
  El tipo de inyección <code>useFactory</code> permite que realices un proceso asíncrono para inyectar un servicio o datos provenientes de una API.

  A partir de NestJS v8, el servicio HttpService importado desde <code>nestjs/common</code>@ fue deprecado. Instala la dependencia <code>nestjs/axios</code>@ e impórtalo desde ahí. No deberás realizar ningún otro cambio en tu código. También debes asegurarte de importar el módulo HttpModule desde la misma dependencia.
  ```typescript
  import { HttpService } from '@nestjs/axios';

  @Module({
    providers: [
      {
        provide: 'DATA',
        useFactory: async (http: HttpService) => {
          return await http.get('<URL_REQUEST>').toPromise()
        },
        inject: [HttpService]
      }
    ],
  })
  export class AppModule {}
  ```
  La propiedad **inject** permite que inyectes (valga la redundancia) dentro de esta función asíncrona del **useFactory** otros servicios que este pueda necesitar. En el ejemplo anterior, se está haciendo una llamada a un request para obtener datos.

  Importa estos datos en el controlador que lo necesite de la siguiente manera.
  ```typescript
  import { Controller, Inject } from '@nestjs/common';

  @Controller()
  export class AppController {

    constructor(@Inject('DATA') private data: any[]) {}
  }
  ```
  Así podrás hacer uso de estos datos que fueron cargados de forma asíncrona.

  Ten en cuenta que, al realizar una solicitud asíncrona, el controlador dependerá de la finalización de este proceso para estar disponible, pudiendo retrasar el inicio de tu aplicación. Esta funcionalidad suele utilizarse para conexiones de base de datos o procesos asíncronos similares.

  **Cuadro de código para inyección de servicios useFactory**

  ```typescript
  // src/app.module.ts
  import { Module, HttpModule, HttpService } from '@nestjs/common';  // 👈 imports

  @Module({
    imports: [HttpModule, UsersModule, ProductsModule],
    controllers: [AppController],
    providers: [
      imports: [HttpModule, UsersModule, ProductsModule], // 👈 add HttpModule
      ...,
      {
        provide: 'TASKS',
        useFactory: async (http: HttpService) => { // 👈 implement useFactory
          const tasks = await http
            .get('https://jsonplaceholder.typicode.com/todos')
            .toPromise();
          return tasks.data;
        },
        inject: [HttpService],
      },
    ],
  })
  export class AppModule {}
  ```

  ```typescript
  // src/app.service.ts
  import { Injectable, Inject } from '@nestjs/common';

  @Injectable()
  export class AppService {
    constructor(
      @Inject('API_KEY') private apiKey: string,
      @Inject('TASKS') private tasks: any[], // 👈 inject TASKS
    ) {}
    getHello(): string {
      console.log(this.tasks); // 👈 print TASKS
      return `Hello World! ${this.apiKey}`;
    }
  }
  ```

## Global Module
  Al desarrollar una aplicación con NestJS, existen necesidades de importar módulos cruzados o de importar un mismo servicio en varios módulos. Lo anterior, hace que la cantidad de imports en cada módulo crezca y se vuelva complicado de escalar.

  ### Cómo usar el módulo global
  NestJS otorga la posibilidad de crear módulos globales que se importarán automáticamente en todos los otros módulos de la aplicación, sin necesidad de importarlos explícitamente.
  ```typescript
  import { Module, Global } from '@nestjs/common';

  @Global()
  @Module({
    // ...
  })
  export class MyCustomModule {}
  ```
  Todos los servicios que importes en este módulo, estarán disponibles para su utilización en cualquier otro módulo.

  Es importante no abusar de esta característica y no tener más de un módulo global para controlar las importaciones. Pueden ocurrir **errores de dependencias circulares** que suceden cuando el **Módulo A** importa al **Módulo B** y este a su vez importa al **Módulo A**. El decorador <code>@Global()</code> te ayudará a resolver estos problemas.

  **Cuadro de código para uso de global module**

  ```typescript
  // src/database/database.module.ts
  import { Module, Global } from '@nestjs/common';

  const API_KEY = '12345634';
  const API_KEY_PROD = 'PROD1212121SA';

  @Global()
  @Module({
    providers: [
      {
        provide: 'API_KEY',
        useValue: process.env.NODE_ENV === 'prod' ? API_KEY_PROD : API_KEY,
      },
    ],
    exports: ['API_KEY'],
  })
  export class DatabaseModule {}
  ```

  ```typescript
  // src/app.module.ts
  ...
  import { DatabaseModule } from './database/database.module';

  @Module({
    imports: [
      HttpModule,
      UsersModule,
      ProductsModule,
      DatabaseModule // 👈 Use DatabaseModule like global Module
    ], 
    ...
  })
  export class AppModule {}
  ```

  ```typescript
  // src/users/services/users.service.ts
  import { Injectable, NotFoundException, Inject } from '@nestjs/common';
  ..

  @Injectable()
  export class UsersService {
    constructor(
      private productsService: ProductsService,
      @Inject('API_KEY') private apiKey: string, // 👈 Inject API_KEY
    ) {}

  }
  ```

## Módulo de configuración
  A medida que tu aplicación crezca, puedes llegar a necesitar decenas de variables de entorno. Variables que cambian de valor dependiendo si estás en un entorno de desarrollo, de pruebas o de producción.

  ### Variables de entorno en NestJS
  El manejo de variables de entorno en NestJS se realiza de una forma muy sencilla. Instala la dependencia <code>npm i @nestjs/config</code> e importa el módulo **ConfigModule** en el módulo principal de tu aplicación. 
  ```typescript
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: '.env',
        isGlobal: true
      }),
    ],
  })
  export class AppModule {}
  ```
  El archivo que almacena las variables de entorno suele llamarse <code>.env</code>. Créalo en la raíz de tu proyecto con las variables que necesitas.
  ```bash
  API_KEY=1324567890
  API_SECRET=ABCDEFGHI
  ```
  De esta manera, las variables de entorno estarán disponibles en tu aplicación y utilizando el objeto global de **NodeJS** llamado <code>process</code> puedes acceder a estos valores de la siguiente manera:
  ```bash
  process.env.API_KEY
  process.env.API_SECRET
  ```

  ### Consejos sobre las variables de entorno
  Es muy importante **NO VERSIONAR** el archivo <code>.env</code> en el repositorio de tu proyecto. No guardes las claves secretas de tu aplicación en **GIT**.

  Para asegurar esto, agrega el archivo <code>.env</code> a la configuración del archivo <code>.gitignore</code> para que no sea reconocido por Git y este no lo guarde en el repositorio.

  Lo que puedes hacer es crear un archivo llamado <code>.env.example</code> que contendrá un modelo de las variables de entorno que tu aplicación necesita, pero no sus valores.
  ```bash
  API_KEY=
  API_SECRET=
  ```
  De este modo, cuidas tu aplicación y guardas un archivo para que cualquier desarrollador que tome el proyecto, sepa qué variables necesita configurar para el funcionamiento de la misma.

  **Cuadro de código para usar el módulo de configuración**

  ```bash
  npm i --save @nestjs/config
  ```

  ```bash
  // .gitignore
  *.env
  ```

  ```bash
  // .env
  DATABASE_NAME=my_db
  API_KEY='1234'
  ```
  ```typescript
  // src/app.module.ts
  ...
  import { ConfigModule } from '@nestjs/config';

  @Module({
    imports: [
      ConfigModule.forRoot({ // 👈 Implement ConfigModule
        envFilePath: '.env',
        isGlobal: true,
      }),
      ...
    ],
  })
  export class AppModule {}
  ```

  ```typescript
  // src/users/services/users.service.ts
  import { ConfigService } from '@nestjs/config';
  ...
  @Injectable()
  export class UsersService {
    constructor(
      private productsService: ProductsService,
      private configService: ConfigService, // 👈 inject ConfigService
    ) {}
    ...

    findAll() {
      const apiKey = this.configService.get('API_KEY'); // 👈 get API_KEY
      const dbName = this.configService.get('DATABASE_NAME');  // 👈 get DATABASE_NAME
      console.log(apiKey, dbName);
      return this.users;
    }

    ...
  }
  ```

## Configuración por ambientes
  **Una aplicación profesional suele tener más de un ambiente**. Ambiente local, ambiente de desarrollo, ambiente de pruebas, producción, entre otros, dependiendo la necesidad del equipo y de la organización. Veamos cómo puedes administrar N cantidad de ambientes en NestJS.

  ### Configuración dinámica del entorno
  Configuremos la aplicación para intercambiar fácilmente entre diversos ambientes, cada uno con su propia configuración.

  **1. Archivo principal para manejo de ambientes**

  Crea un archivo llamado <code>enviroments.ts</code> (o el nombre que prefieras) que contendrá un objeto con una propiedad por cada ambiente que tenga tu aplicación.
  ```typescript
  // src/enviroments.ts
  export const enviroments = {
    dev: '.env',
    test: '.test.env',
    prod: '.prod.env',
  };
  ```
  **2. Configuración por ambiente**

  Crea un archivo <code>.env</code> por cada ambiente que necesites. Recuerda que todos los archivos finalizados en <code>.env</code> no deben guardarse en GIT.
  ```typescript
  // .test.env
  DATABASE_NAME=my_db_test
  API_KEY=12345
  ```
  ```typescript
  // .prod.env
  DATABASE_NAME=my_db_prod
  API_KEY=67890
  ```
  **3. Importando variables de entorno**

  Importa en el módulo principal de tu aplicación el archivo principal para manejo de ambientes y, a través de una única variable de entorno llamada **NODE_ENV**, elegirás qué configuración usar.

  **NODE_ENV** es una variable de entorno propia de NodeJS y del framework Express que se encuentra preseteada en tu aplicación.
  ```typescript
  import { enviroments } from './enviroments'; // 👈

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: enviroments[process.env.NODE_ENV] || '.env', // 👈
        isGlobal: true,
      }),
    ],
  })
  export class AppModule {}
  ```
  **4. Inicio de la aplicación**

  Finalmente, para iniciar tu aplicación basta con el comando <code>NODE_ENV=test npm run start:dev</code> o <code>NODE_ENV=prod npm run start:dev</code> para configurar la variable de entorno principal **NODE_ENV** y escoger qué configuración utilizar.

  **5. Utilizando las variables de entorno**

  Puedes utilizar las variables de entorno en tu aplicación de dos maneras. Con el objeto global de NodeJS llamado <code>process</code>:
  ```typescript
  process.env.DATABASE_NAME
  process.env.API_KEY
  ```
  O puedes usar estas variables a través del servicio ConfigService proveniente de <code>@nestjs/config</code>.
  ```typescript
  import { ConfigService } from '@nestjs/config';

  @Injectable()
  export class AppService {

    constructor(private config: ConfigService) {}
    
    getEnvs(): string {
      const apiKey = this.config.get<string>('API_KEY');
      const name = this.config.get('DATABASE_NAME');
      return `Envs: ${apiKey} ${name}`;
    }
  }
  ```
  De este modo, configura de la mejor manera que necesites para tu aplicación el manejo de múltiples ambientes, cada uno con su propia configuración.

  **Cuadro de código para la configuración de ambientes**
  ```typescript
  // .stag.env
  DATABASE_NAME=my_db_stag
  API_KEY=333
  ```
  ```typescript
  // .prod.env
  DATABASE_NAME=my_db_prod
  API_KEY=999
  ```
  ```typescript
  // src/enviroments.ts
  export const enviroments = {
    dev: '.env',
    stag: '.stag.env',
    prod: '.prod.env',
  };
  ```
  ```typescript
  // src/app.module.ts
  ...
  import { enviroments } from './enviroments'; // 👈

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: enviroments[process.env.NODE_ENV] || '.env', // 👈
        isGlobal: true,
      }),
      ...
    ],
    ...
  })
  export class AppModule {}
  ```
  ```typescript
  // src/app.service.ts
  import { ConfigService } from '@nestjs/config'; // 👈

  @Injectable()
  export class AppService {
    constructor(
      @Inject('TASKS') private tasks: any[],
      private config: ConfigService,  // 👈
    ) {}
    getHello(): string {
      const apiKey = this.config.get<string>('API_KEY');  // 👈
      const name = this.config.get('DATABASE_NAME');  // 👈
      return `Hello World! ${apiKey} ${name}`;
    }
  }
  ```
  Rin with NODE_ENV // 👈
  ```bash
  NODE_ENV=prod npm run start:dev
  NODE_ENV=stag npm run start:dev
  ```

## Tipado en config
  A medida que tu aplicación acumule más y más variables de entorno, puede volverse inmanejable y es propenso a tener errores el no recordar sus nombres o escribirlos mal. A continuación verás como tipar variables.

  ### Cómo hacer el tipado de variables de entorno
  Seguriza tu lista de variables de entorno de manera que evites errores que son difíciles de visualizar. Veamos cómo puedes tipar tus variables.

  **1. Archivo de tipado de variables**

  Crea un archivo al que denominaremos <code>config.ts</code> que contendrá el tipado de tu aplicación con ayuda de la dependencia <code>@nestjs/config</code>.
  ```typescript
  // src/config.ts
  import { registerAs } from "@nestjs/config";

  export default registerAs('config', () => {
    return {
      database: {
        name: process.env.DATABASE_NAME,
        port: process.env.DATABASE_PORT,
      },
      apiKey: process.env.API_KEY,
    }
  })
  ```
  Importa **registerAs** desde <code>@nestjs/config</code> que servirá para crear el tipado de datos. Crea un objeto con la estructura de datos que necesita tu aplicación. Este objeto contiene los valores de las variables de entorno tomados con el objeto global de NodeJS, <code>process</code>.

  **2. Importación del tipado de datos**

  Importa el nuevo archivo de configuración en el módulo de tu proyecto de la siguiente manera para que este sea reconocido.
  ```typescript
  import { ConfigModule } from '@nestjs/config';
  import config from './config';

  @Global()
  @Module({
    imports: [
      HttpModule,
      ConfigModule.forRoot({
        envFilePath: '.env',
        load: [config],
        isGlobal: true
      }),
    ],
  })
  export class AppModule {}
  ```

  **3. Tipado de variables de entorno**
  Es momento de utilizar este objeto que genera una interfaz entre nuestra aplicación y las variables de entorno para no confundir el nombre de cada variable.
  ```typescript
  import { Controller, Inject } from '@nestjs/common';
  import { ConfigType } from '@nestjs/config';
  import config from './config';

  @Controller()
  export class AppController {

    constructor(
      @Inject(config.KEY) private configService: ConfigType<typeof config>
    ) {}
    
    getEnvs(): string {
      const apiKey = this.configService.apiKey;
      const name = this.configService.database.name;
      return `Envs: ${apiKey} ${name}`;
    }
  }
  ```
  Observa la configuración necesaria para inyectar y tipar tus variables de entorno. Ahora ya no tendrás que preocuparte por posibles errores al invocar a una de estas variables y evitar dolores de cabeza debugueando estos errores.

  **Cuadro de código para tipado en config**
  ```typescript
  // .env
  DATABASE_NAME=my_db_prod
  API_KEY=999
  DATABASE_PORT=8091 // 👈
  ```
  ```typescript
  // .stag.env
  DATABASE_NAME=my_db_stag
  API_KEY=333
  DATABASE_PORT=8091 // 👈
  ```
  ```typescript
  // .prod.env
  DATABASE_NAME=my_db_prod
  API_KEY=999
  DATABASE_PORT=8091 // 👈
  ```
  ```typescript
  // src/config.ts // 👈 new file
  import { registerAs } from '@nestjs/config';

  export default registerAs('config', () => { // 👈 export default
    return { 
      database: {
        name: process.env.DATABASE_NAME,
        port: process.env.DATABASE_PORT,
      },
      apiKey: process.env.API_KEY,
    };
  });
  ```
  ```typescript
  // src/app.module.ts
  import config from './config'; // 👈

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: enviroments[process.env.NODE_ENV] || '.env',
        load: [config], // 👈
        isGlobal: true,
      }),
      ...
    ],
    ...
  })
  export class AppModule {}
  ```
  ```typescript
  // src/app.service.ts
  import { ConfigType } from '@nestjs/config'; // 👈 Import ConfigType 
  import config from './config'; // 👈 config file

  @Injectable()
  export class AppService {
    constructor(
      @Inject('TASKS') private tasks: any[],
      @Inject(config.KEY) private configService: ConfigType<typeof config>, // 👈
    ) {}
    getHello(): string {
      const apiKey = this.configService.apiKey; // 👈
      const name = this.configService.database.name; // 👈
      return `Hello World! ${apiKey} ${name}`;
    }
  }
  ```

## Validación de esquemas en .envs con Joi
  Las variables de entorno son sensibles, pueden ser requeridas o no, pueden ser un string o un number. **Validemos tus variables de entorno para evitar errores** u omisiones de las mismas.

  ### Validando variables de entorno
  Instala la dependencia [Joi](https://www.npmjs.com/package/joi) con el comando <code>npm instal joi --save</code>. La misma nos dará las herramientas para realizar validaciones de nuestras variables de entorno.

  Importa **Joi** en el módulo de tu aplicación y a través de la propiedad validationSchema del objeto que recibe el ConfigModule crea el tipado y las validaciones de tus variables de entorno.
  ```typescript
  import { ConfigModule } from '@nestjs/config';
  import * as Joi from 'joi';

  import config from './config';

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: '.env',
        load: [config],
        isGlobal: true,
        validationSchema: Joi.object({
          API_KEY: Joi.string().required(),
          DATABASE_NAME: Joi.string().required(),
          DATABASE_PORT: Joi.number().required(),
        })
      }),
    ],
    ],
  })
  export class AppModule {}
  ```
  Lo que hace **Joi** es asegurar que, en el archivo <code>.env</code>, existan las variables de entorno indicadas dependiendo si son obligatorias o no, además de validar el tipo para no ingresar un string donde debería ir un number.

  En equipos de trabajo profesional, quienes suelen desplegar las aplicaciones en los entornos es el equipo **DevOps** y ellos **no necesariamente saben qué variables de entorno necesita tu aplicación** y de qué tipo son. Gracias a esta configuración, **tu app emitirá mensajes de error** claros por consola cuando alguna variable no sea correcta.

  **Cuadro de código para variables de entorno**
  ```bash
  npm install --save joi
  ```
  ```typescript
  // src/app.module.ts
  import * as Joi from 'joi';  // 👈

  @Module({
    imports: [
      ConfigModule.forRoot({
        envFilePath: enviroments[process.env.NODE_ENV] || '.env',
        load: [config],
        isGlobal: true,
        validationSchema: Joi.object({ // 👈
          API_KEY: Joi.number().required(),
          DATABASE_NAME: Joi.string().required(),
          DATABASE_PORT: Joi.number().required(),
        }),
      }),
      ...
    ],
    ...
  })
  export class AppModule {}
  ```

## Integrando Swagger y PartialType con Open API
  **Una API profesional debe estar documentada**. Cuando hablamos de documentación, nos suena a una tarea tediosa que nadie quiere realizar. Afortunadamente, NestJS permite automatizar fácilmente la creación de la misma.

  - [OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0)
  - [NestJS Swagger](https://docs.nestjs.com/openapi/introduction)
  - [Using the CLI plugin](https://docs.nestjs.com/openapi/cli-plugin#using-the-cli-plugin)

  ### Cómo hacer la documentación API Rest
  [Swagger](https://swagger.io/) es un reconocido set de herramientas para la documentación de API Rest. Instala las dependencias necesarias con el comando <code>npm install --save @nestjs/swagger swagger-ui-express</code> y configura el archivo <code>ain.ts</code>m con el siguiente código.
  ```typescript
  // main.ts
  import { NestFactory } from '@nestjs/core';
  import { AppModule } from './app.module';
  import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
      
    // Configuración Swagger en NestJS
    const config = new DocumentBuilder()
      .setTitle('Platzi API')
      .setDescription('Documentación Platzi API')
      .setVersion('1.0')
      .build();
    const document = SwaggerModule.createDocument(app, config);
    
    // URL API
    SwaggerModule.setup('docs', app, document);

    await app.listen(process.env.PORT || 3000);
  }
  bootstrap();
  ```
  Setea el título, descripción y versión de tu documentación. Lo más importante es la URL para acceder a la misma.

  Levanta el servidor con <code>npm run start:dev</code> y accede a <code>localhost:3000/docs</code> para visualizar la documentación autogenerada que mapea automáticamente todos los endpoints de tu aplicación.

  ### Tipado de la documentación
  La documentación autogenerada por Swagger es bastante simple, puedes volverla más completa tipando los datos de entrada y salida de cada endpoint gracias a los DTO.

  Busca el archivo <code>nest-cli.json</code> en la raíz de tu proyecto y agrega el siguiente plugin:
  ```json
  {
    "$schema": "https://json.schemastore.org/nest-cli",
    "collection": "@nestjs/schematics",
    "sourceRoot": "src",
    "compileOptions": {
      "plugins": ["@nestjs/swagger"]
    }
  }
  ```
  A continuación, prepara tus DTO de la siguiente manera:
  ```typescript
  import { IsNotEmpty, IsString, IsNumber } from 'class-validator';
  import { ApiProperty, PartialType, OmitType } from '@nestjs/swagger';

  export class CreateProductDTO {

    @ApiProperty()
    @IsNotEmpty()
    @IsString()
    readonly name: string;

    @ApiProperty()
    @IsNotEmpty()
    @IsString()
    readonly description: string;

    @ApiProperty()
    @IsNotEmpty()
    @IsNumber()
    readonly price: number;
  }

  export class UpdateAuthorDto extends PartialType(
    OmitType(CreateProductDTO, ['name']),
  ) {}
  ```
  Lo más relevante aquí es importar **PartialType y OmitType** desde <code>@nestjs/swagger</code> en lugar de importarlo desde @nestjs/mapped-types. Coloca también el decorador <code>@ApiProperty()</code> en cada una de las propiedades que el DTO necesita.
  ![](https://static.platzi.com/media/user_upload/Screenshot%20from%202022-06-17%2014-08-51%281%29-436e5207-765c-4d51-94b4-b3f72d1b8c93.jpg)

  De esta manera, observarás en la documentación que indica el tipo de dato que requiere cada uno de tus endpoints.

  ### Cuadro de código para uso de swagger
  ```bash
  npm install --save @nestjs/swagger swagger-ui-express
  ```
  ```typescript
  // src/main.ts 
  import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

  async function bootstrap() {
    ...
    const config = new DocumentBuilder()
      .setTitle('API')
      .setDescription('PLATZI STORE')
      .setVersion('1.0')
      .build();
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('docs', app, document);
    ...
    await app.listen(3000);
  }
  bootstrap();
  ```
  ```json
  # nest-cli.json
  {
    "collection": "@nestjs/schematics",
    "sourceRoot": "src",
    "compilerOptions": {
      "plugins": ["@nestjs/swagger/plugin"]
    }
  }
  ```
  ```typescript
  // src/products/dtos/brand.dtos.ts
  import { PartialType } from '@nestjs/swagger';
  ```
  ```typescript
  // src/products/dtos/category.dtos.ts
  import { PartialType } from '@nestjs/swagger';
  ```
  ```typescript
  // src/products/dtos/products.dtos.ts
  import { PartialType } from '@nestjs/swagger';
  ```
  ```typescript
  // src/users/dtos/customer.dto.ts
  import { PartialType } from '@nestjs/swagger';
  ```
  ```typescript
  // src/users/dtos/user.dto.ts
  import { PartialType } from '@nestjs/swagger';
  ```

## Extendiendo la documentación
  La documentación automática que genera NestJS y Swagger es muy fácil de implementar y otorga una buena base. **La documentación de tu aplicación puede ser aún más completa y detallada**, si así lo quieres con algo de trabajo de tu parte.

  ### Cómo hacer la documentación personalizada
  Veamos varios decoradores que te servirán para ampliar la documentación de tu API.

  **Descripción de las propiedades**

  En tus DTO, puedes dar detalle sobre qué se espera recibir en cada propiedad de tus endpoints gracias al decorador <code>@ApiProperty()</code>
  ```typescript
  import { IsNotEmpty, IsString, IsNumber } from 'class-validator';
  import { ApiProperty, PartialType, OmitType } from '@nestjs/swagger';

  export class CreateProductDTO {

    @ApiProperty({ description: 'Nombre del producto' })
    @IsNotEmpty()
    @IsString()
    readonly name: string;

    @ApiProperty({ description: 'Descripción del producto' })
    @IsNotEmpty()
    @IsString()
    readonly description: string;

    @ApiProperty({ description: 'Precio del producto' })
    @IsNotEmpty()
    @IsNumber()
    readonly price: number;
  }
  ```
  **Descripción de los controladores**

  Puedes agrupar los endpoints en la documentación por controlador con el decorador <code>@ApiTags()</code> y describir, endpoint por endpoint, la funcionalidad de cada uno con el decorador <code>@ApiOperation()</code>.
  ```typescript
  import { ApiTags, ApiOperation } from '@nestjs/swagger';

  @ApiTags('Productos')
  @Controller()
  export class AppController {

    @ApiOperation({ summary: 'Obtener lista de productos.' })
    @Get('products')
    getProducts() {
      // ...
    }
  }
  ```
  Para obtener un resultado en la documentación de tu API como el siguiente:

  ![](https://static.platzi.com/media/user_upload/Screenshot%20from%202022-06-17%2015-42-27-5241b1e3-815e-483c-895c-f7387b19d55d.jpg)

  De este modo, la documentación de tu aplicación es súper profesional y está lista para ser recibida por el equipo front-end o por clientes externos que consumirán el servicio.

  **Cuadro de código para documentación personalizada**

  ```typescript
  // src/products/dtos/products.dtos.ts
  import { PartialType, ApiProperty } from '@nestjs/swagger';

  import {
    IsString,
    IsNumber,
    IsUrl,
    IsNotEmpty,
    IsPositive,
  } from 'class-validator';
  import { PartialType, ApiProperty } from '@nestjs/swagger'; // 👈

  export class CreateProductDto {
    @IsString()
    @IsNotEmpty()
    @ApiProperty({ description: `product's name` }) // 👈
    readonly name: string;

    @IsString()
    @IsNotEmpty()
    @ApiProperty() // 👈
    readonly description: string;

    @IsNumber()
    @IsNotEmpty()
    @IsPositive()
    @ApiProperty() // 👈
    readonly price: number;

    @IsNumber()
    @IsNotEmpty()
    @ApiProperty() // 👈
    readonly stock: number;

    @IsUrl()
    @IsNotEmpty()
    @ApiProperty() // 👈
    readonly image: string;
  }

  export class UpdateProductDto extends PartialType(CreateProductDto) {}
  ```
  ```typescript
  // src/products/controllers/products.controller.ts
  import { ApiTags, ApiOperation } from '@nestjs/swagger'; // 👈

  @ApiTags('products') // 👈
  @Controller('products')
  export class ProductsController {
    constructor(private productsService: ProductsService) {}

    @Get()
    @ApiOperation({ summary: 'List of products' }) // 👈
    getProducts(
      @Query('limit') limit = 100,
      @Query('offset') offset = 0,
      @Query('brand') brand: string,
    ) {
      // return {
      //   message: `products limit=> ${limit} offset=> ${offset} brand=> ${brand}`,
      // };
      return this.productsService.findAll();
    }
  }
  ```
  ```typescript
  // src/products/controllers/brands.controller.ts
  import { ApiTags } from '@nestjs/swagger';


  @ApiTags('brands') // 👈
  @Controller('brands')
  export class BrandsController {
    ...
  }
  ```

## Configuración de Heroku
  Llega el momento del despliegue de tu aplicación en un entorno productivo. Utilizaremos [Heroku](https://www.heroku.com/), uno de los proveedores de servidores Cloud más utilizado y fácil de utilizar de la industria.

  ### Configuración del proyecto al usar Heroku
  Preparar tu aplicación para Heroku requiere de algunas configuraciones sencillas.

  **Configuración de puerto y CORS**

  Heroku, por defecto, usa una variable de entorno llamada <code>PORT</code> para levantar la aplicación en un puerto aleatorio. Asegúrate de configurar esta variable dinámica en el bootstrap de tu app, además de activar CORS para no tener problemas con el mismo. Agrega las configuraciones en el archivo <code>main.ts</code>.
  ```typescript
  // main.ts
  import { NestFactory } from '@nestjs/core';
  import { AppModule } from './app.module';

  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.enableCors();
    await app.listen(process.env.PORT || 3000);
  }
  bootstrap();
  ```

  **Configuración versión de NodeJS**

  Edita el archivo package.json para especificar la versión de NodeJS que tu aplicación necesita con la siguiente configuración:
  ```bash
  "engines": {
    "node": "14.x"
  },
  ```

  **Configuración de Heroku**

  Heroku requiere de un pequeño archivo adicional en la raíz de tu proyecto llamado Procfile que contiene el comando que da inicio a tu proyecto:
  ```bash
  web: npm run start:prod
  ```
  Además, [Heroku posee su propio CLI](https://devcenter.heroku.com/articles/heroku-cli) que nos ayudará en el despliegue de cualquier aplicación. Instálalo dependiendo tu sistema operativo para estar listo para el despliegue de tu app.

  [Deploying Node.js Apps on Heroku](https://devcenter.heroku.com/articles/deploying-nodejs)

## Deploy de NestJS en Heroku
  Comandos:
  - heroku login
  - heroku create
  - heroku local web "provar de forma local"

  Teniendo tu aplicación configurada correctamente. Realiza el despliegue en Heroku instalando su CLI en primer lugar.

  ### Cómo hacer el despliegue en Heroku
  Luego de instalar el CLI, realiza un heroku login para autenticarte. Si aún no posees una cuenta en Heroku, es el momento de crearte una de forma gratuita.

  **Creando proyecto en Heroku**

  Una vez situado en tu proyecto, utiliza el comando <code>heroku create -a "nombre-proyecto"</code> para crear un nuevo proyecto remoto en tu cuenta de Heroku.

  Heroku, internamente, posee su propio servidor de GIT. Si realizas un <code>git remote -v</code>, observarás que este ha agregado a tu proyecto nuevos servidores remotos. El despliegue se hará usando los propios de Heroku.

  Con el simple comando <code>git push heroku master</code> la aplicación demorará unos pocos minutos en desplegarse. Podrás observar el progreso en la consola.

  La aplicación quedará desplegada en una URL proporcionada por Heroku similar <code>https://"nombre_proyecto".herokuapp.com/</code>a , a la cual puedes acceder para observar si tu aplicación fue desplegada con éxito.

  **Variables de entorno en Heroku**

  Si tu aplicación utiliza variables de entorno debes configurar estas. De manera muy sencilla, el siguiente comando te permite configurar cada una de tus variables de entorno: <code>heroku config:set APP_KEY=12345</code>, mientras que el comando <code>heroku config</code>heroku config te permitirá ver una lista de las variables que ya están configuradas.

  Recuerda que las variables de entorno son sensibles y debes cuidar quién tiene acceso a ellas.

  ¡Felicidades! Has desplegado tu aplicación en un entorno productivo. Ahora el mundo puede acceder a tu app.

  ### Cuadro de código para despliegue de Heroku
  ```bash
  heroku local web
  git checkout master
  git merge 14-step
  git remote -v
  git push heroku master
  heroku logs --tail
  npm run format
  ```
  También puedes usar el comando <code>heroku config:set APP_KEY=12345</code> para configurar variables de ambiente desde el CLI y esto hace que la app se reinicie sin la necesidad de enviar un push.

  Notas:
  - Cuidado con los typos 🙂
  - No dejes comments en producción 📢
