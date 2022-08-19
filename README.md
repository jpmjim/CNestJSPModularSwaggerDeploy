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
