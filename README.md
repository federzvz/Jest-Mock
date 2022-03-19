# Jest-Mock

## Mocking a Funciones y Módulos

Hay tres tipos principales de mocks de módulos y funciones en Jest:

  - jest.fn: simula una función
  - jest.mock: simula un módulo
  - jest.spyOn: espiar una función
Cada uno de estos creará, de alguna manera, la función mock. Para explicar cómo cada uno de estos lo hace vamos a crear un proyecto con React, create react app creara un proyecto base donde Jest ya vendrá incluido

npx

`npx create-react-app my-app`

npm

`npm init react-app my-app`

yarn

`yarn create react-app my-app`

probaremos app.js y sin llamar a las funciones reales de math.js o espiarlas para asegurarse de que se llamen como se esperaba. Este ejemplo es trivial, pero imagina que math.js es un cálculo complejo o requiere algún IO que quieres evitar

### math.js
```js
export const add      = (a, b) => a + b;
export const subtract = (a, b) => b - a;
export const multiply = (a, b) => a * b;
export const divide   = (a, b) => b / a;
```

### app.js
```js
import * as math from './math.js';
export const doAdd      = (a, b) => math.add(a, b);
export const doSubtract = (a, b) => math.subtract(a, b);
export const doMultiply = (a, b) => math.multiply(a, b);
export const doDivide   = (a, b) => math.divide(a, b);
```

### Mock de una función con Jest.fn

La estrategia más básica para un mock es reasignar una función a la función del mock. Luego, en cualquier lugar que se usen las funciones reasignadas, se llamará al mock en lugar de a la función original:

### app.test.js

```js
import * as app from "./app";
import * as math from "./math";

math.add = jest.fn();
math.subtract = jest.fn();

test("calls math.add", () => {
  app.doAdd(1, 2);
  expect(math.add).toHaveBeenCalledWith(1, 2);
});

test("calls math.subtract", () => {
  app.doSubtract(1, 2);
  expect(math.subtract).toHaveBeenCalledWith(1, 2);
});
```

### Mock de un modulo con Jest.mock
Una estrategia común es la de desarrollar un mock para simular un modulo completo de forma automática, cuando usemos la función jest.mock('./math.sj') convirtiendo simulando todas sus funciones.

```js
export const add      = jest.fn();
export const subtract = jest.fn();
export const multiply = jest.fn();
export const divide   = jest.fn();
```

### mock_jest_mock.test.js

```js
import * as app from "./app";
import * as math from "./math";
// Set all module functions to jest.fn
jest.mock("./math.js");
test("calls math.add", () => {
  app.doAdd(1, 2);
  expect(math.add).toHaveBeenCalledWith(1, 2);
});
test("calls math.subtract", () => {
  app.doSubtract(1, 2);
  expect(math.subtract).toHaveBeenCalledWith(1, 2);
});
```
> al realizar el mock del modulo entero podemos simular cada una de sus funciones.

### Spy de una función con Jest.spyOn

También podemos supervisar el llamado de una función, pero manteniendo la implementación original. En otras ocasiones, es posible que desee simular la implementación, pero restaurar el original más adelante.

Aquí simplemente "espíamos" las llamadas a la función de math, pero dejamos la implementación original en su lugar:

### mock_jest_spyOn.test.js
```js
import * as app from "./app";
import * as math from "./math";
test("calls math.add", () => {
  const addMock = jest.spyOn(math, "add");
  // calls the original implementation
  expect(app.doAdd(1, 2)).toEqual(3);
  // and the spy stores the calls to add
  expect(addMock).toHaveBeenCalledWith(1, 2);
});
```

En el ejemplo anterior podemos ver un escenario donde comprobamos un efecto de invocar la función sin reemplazar la función con un mock realmente.

En otros casos podemos simular la función para mockear su funcionamiento, pero restaurar la implementación original en algún momento. 
```js
import * as app from "./app";
import * as math from "./math";
test("calls math.add", () => {
  const addMock = jest.spyOn(math, "add");
  // override the implementation
  addMock.mockImplementation(() => "mock");
  expect(app.doAdd(1, 2)).toEqual("mock");
  // restore the original implementation
  addMock.mockRestore();
  expect(app.doAdd(1, 2)).toEqual(3);
});
```

Describe el funcionamiento del siguiente test. ¿Qué ocurre?

### mock_jest_spyOn_sugar.js
```js
import * as app from "./app";
import * as math from "./math";
test("calls math.add", () => {
  // store the original implementation
  const originalAdd = math.add;
  // mock add with the original implementation
  math.add = jest.fn(originalAdd);
  // spy the calls to add
  expect(app.doAdd(1, 2)).toEqual(3);
  expect(math.add).toHaveBeenCalledWith(1, 2);
  // override the implementation
  math.add.mockImplementation(() => "mock");
  expect(app.doAdd(1, 2)).toEqual("mock");
  expect(math.add).toHaveBeenCalledWith(1, 2);
  // restore the original implementation
  math.add = originalAdd;
  expect(app.doAdd(1, 2)).toEqual(3);
});
```

## Simulando APIs

Realicemos un ultimo test. Vamos a simular el consumo de un API que usa axios para obtener un conjunto de datos. La clase que probaremos sera:
```js
import axios from 'axios';
class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}
export default Users;
```
>Ahora, para probar la clase sin realizar el llamado real al API, usaremos la funcion de mock para simular esta dependencia.

### users.test.js

```js
import axios from 'axios';
import Users from './users';
jest.mock('axios');
test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);
  return Users.all().then(data => expect(data).toEqual(users));
});
```

Aunque no vas a usar la dependencia, si necesitas agregarla a tu proyecto. 

npm

`npm install axios`
