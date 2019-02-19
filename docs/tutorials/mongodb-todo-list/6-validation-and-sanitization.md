# Validation and Sanitization

Currently inputs received by the server are not checked. Everyone could send anything when requesting `POST /api/todos`. That's why client inputs cannot be trusted.

You will use the `ValidateBody` and `ValidateParams` hooks to validate and sanitize incoming data.

A *hook* is a decorator that is attached to a route handler (a controller method). It is executed before the method and is therefore particularly suitable for validation or access control.

The `ValidateBody` and `ValidateParams` check respectively the `body` and `params` properties of the request object. They take a schema as unique argument.

> FoalTS uses [Ajv](https://github.com/epoberezkin/ajv), a fast JSON Schema validator, to define its schemas.

Let's add validation and sanitization to your application. In fact, you have already defined the todo schema in the `create-todo` script earlier.

```typescript
import {
  ...
  ValidateBody, ValidateParams
} from '@foal/core';

export class ApiController {

  ...

  @Post('/todos')
  @ValidateBody({
    // The body request should be an object once parsed by the framework.
    // Every additional properties that are not defined in the "properties"
    // object should be removed.
    additionalProperties: false,
    properties: {
      // The "text" property of ctx.request.body should be a string if it exists.
      text: { type: 'string' }
    },
    // The property "text" is required.
    required: [ 'text' ],
    type: 'object',
  })
  async postTodo(ctx: Context) {
    const todo = new Todo();
    todo.text = ctx.request.body.text;

    await todo.save();

    return new HttpResponseCreated(todo);
  }

  @Delete('/todos/:id')
  @ValidateParams({
    properties: {
      id: { type: 'string' }
    },
    type: 'object',
  })
  async deleteTodo(ctx: Context) {
    const todo = await Todo.findById(ctx.request.params.id);
    if (!todo) {
      return new HttpResponseNotFound();
    }
    await Todo.findByIdAndDelete(ctx.request.params.id);
    return new HttpResponseNoContent();
  }

}

```