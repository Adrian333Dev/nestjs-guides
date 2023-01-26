# Creating a REST API application

In this section, we will create a REST API application with basic functionality.

## Creating and running the application

- Run `nest new my-app` to create a new application.
- To run the application, run `npm run start:dev` in the project directory.

## CRUD generator

> 💡 **[CRUD generator](https://docs.nestjs.com/recipes/crud-generator#generating-a-new-resource)** is a CLI tool that allows you to generate a CRUD (Create, Read, Update, Delete) controller and a service for a given entity.

- Run `nest generate resource boards --no-spec` to generate a new resource with boilerplate code for the controller and service. The `--no-spec` flag will prevent the CLI from generating test files.

**Generated files:**

```file
boards/
├── boards.controller.ts
├── boards.module.ts
├── boards.service.ts
├── dto/
│   ├── create-board.dto.ts
│   └── update-board.dto.ts
└── entities/
    └── board.entity.ts
```
