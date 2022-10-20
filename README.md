# A novel schema-first approach to building forms
## Intro
Creating forms is an inseparable part of the building process of your application. Delivering them in React can be very troublesome, depending on the complexity of your application and business requirements.
A typical approach to building forms that a developer follows could be called code-first, where you define your form using React components. One may ask if this is the only way to create forms. What if we wanted to write the code once and control how the form looks or how it’s validated using some data format?
We want to present the relatively new schema-first form design. The core difference is that you use a schema to control how your form should be structured. In this article, we will show the difference between these two approaches and how one could benefit from using the schema in the form-building process.

## The code-first approach and its limitations
The process usually involves manually creating form input components and binding them with the React state. Many libraries have been created to mitigate the need for boilerplate code to be written. One of the most popular ones that support the code-first approach is Formik, where you assemble the form using form pieces.
The code-first is more prone to mix definitions of the form layout, reacting to actions, validation, and other layers that build up a form. In general, it’s a lot easier to mix implementation details with the domain logic of the form. This can not only affect the readability of the code but also slow down the development process or even force us to refactor early. 
In the code-first approach, you have to start with the implementation, leading to less thought-through solutions, whereas the schema-first approach forces you to start by analyzing the problem. Compared to reading code, the schema is a lot easier to understand, which means other developers will have an easy time working with your schema definition.

## Using uniforms to build schema-first forms in React
Building a solution that will parse schemas and render forms could be too complex to start with. However, there is already a library called uniforms that handles this for you.
All that we need is the schema,
```ts
const loginSchema: JSONSchema = {
 title: "Login form",
 type: "object",
 properties: {
   username: {
     type: "string",
     minLength: 1
   },
   password: {
     type: "string",
     minLength: 8,
     uniforms: {
       type: "password"
     }
   }
 },
 required: ["username", "password"]
};
```

and uniforms will handle the rest for you

```ts
import { AutoForm } from "uniforms"
import { createBridge } from "./createBridge" // explained later
 
function AutomaticForm() {
 return <AutoForm schema={createBridge(loginSchema)} />;
}
```
producing the following view

## Schema parsing and data layer
The uniforms package is capable of parsing multiple schema types such as JSON Schema and automatically renders appropriate fields as React components. It’s a very important characteristic of using schema - separating the data layer from the form layout.
To understand the schema, uniforms use the bridge concept. It allows reading any schema as long as you provide an implementation. Fortunately, uniforms come with many bridges that operate on popular schemas that you can choose from.
An example is JSONSchemaBridge which requires you to pass a valid JSON schema and a validator.
```ts
function createBridge(schema: JSONSchema) {
 const validator = createValidator(schema);
 return new JSONSchemaBridge(schema, validator);
}
```

## Validation
We know that there is an infinite number of ways in which you might want your data to be validated. Having said that, we allow you to write your own validators. At the same time, we suggest using tools that integrate with the schema itself and provide validation based on the restrictions described in the schema. Our favorite tool for JSON Schema validation is Ajv. It allows you to plug & play the validation of the schema you write without writing too much boilerplate code.
An example validator can look like the following:
```ts
function createValidator(schema: JSONSchema) {
 const ajv = new Ajv({ keywords: ["uniforms"], allErrors: true });
 const validate = ajv.compile(schema);
 const validator = (model: Record<string, unknown>) => {
   validate(model);
   const errors = validate.errors;
   return errors?.length ? { details: errors } : null;
 };
 return validator;
}
```
Don't be afraid if this code looks intimidating to you. You will need to set this up in your app only once. You can find more about creating validators in the documentation.
The above validation is strictly tied to the schema definition. It’s also possible to add more validation by passing `onValidate` callback to the `<AutoForm />` component. It’s especially useful if you want to combine asynchronous validation with some API after you’ve confirmed that the form data meets the schema requirements.
Layout
By default, uniforms will render the fields one by one in a sequential fashion. The library has a predefined set of components (called fields) handling various data types. This is great for prototyping but might be insufficient for tailoring the layout for your needs. To overcome this, uniforms allow you to manually place form fields in the React tree under the uniform’s form context and connect them to the form state. There are also uniforms-compatible theme packages that use popular design libraries such as MUI, Bootstrap, Ant Design, and many more.
```ts
function CustomLayoutForm() {
 return (
   <AutoForm schema={createBridge(login)}>
     <AutoField name="username" />
     <AutoField name="password" />
     <SubmitField />
     <ErrorsField />
   </AutoForm>
 );
}
```
## Custom fields
At some point, your field will have to cover some custom logic, and uniforms allow you to create your own fields that consume uniforms' context.
```ts
function DisplayFormContext() {
 const { formRef, schema, ...form } = useForm();
 return <pre>{JSON.stringify(form, null, 2)}</pre>;
}
```

It’s also possible to consume just a single data field using `useField(name)`.
You can also define which component should be used for a particular field on the schema level, completely moving the control from the code to the schema.
## Metaprogramming
What we love about uniforms at Vazco is the fact that we can write applications that can display any form represented by schemas. This allows you to create a software where you can create forms without writing a single line of code. That enables the so-called citizen development, where non-developers can build forms too. Of course, writing the schema with complex validation and encoding the form layout are completely different tasks. This led us to create a drag & drop tool for building form schemas and layouts. Feel free to check out Form Builder and offload schema building to your customers or help developers prototype.
Understanding business needs is essential and should be done at the early design stages. Prototyping helps you verify ideas but requires fast feedback and the ability to adjust things quickly. Luckily, uniforms solve this problem with automatic form generation. Provide the schema, and the form will be rendered for you.

## Conclusion
There is no single way of building forms. However, you should always choose the tool best suited for the problem you’re facing. Using a schema-first approach helps split implementation from design. This results in overall faster development speed in the long run because of a better understanding of the problem. It makes maintenance easier too. Choosing a schema-first approach with uniforms can benefit you in applications heavily focused on forms. Whether you’re using it for fast feedback or rendering many forms, it will serve its purpose. Designing a schema in the first place leads to more resilient solutions and helps to maintain them, making everyone happy.

The code examples can be found here.
