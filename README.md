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
