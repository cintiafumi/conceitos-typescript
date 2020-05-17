# Typescript
É uma linguagem totalmente baseada em Javascript. Ela adiciona as tipagens e acessar as features mais recentes do Javascript no Node ou no browser.
Ajuda muito no ambiente de desenvolvimento para criação e manutenção do código.
IntelliSense do editor de código já consegue entender o formato da variável.

Criando um projeto backend em Typescript.
```bash
mkdir typescript
cd typescript
yarn init -y
yarn add typescript -D
```

Criar um arquivo `src/index.ts`. E já dá para usar o Javascript mais recente, incluindo `import` que ainda não é aceito no Node v.12.

Adicionar o pacote `express`
```bash
yarn add express
```

No arquivo `index.ts`, ao importar o `express` o VSCode já avisa que precisa instalar também o pacote `@types/express`
```bash
yarn add -D @types/express
```

Agora já conseguimos utilizar o `express` no arquivo `index.ts` e o Intellisense já entende.
```ts
import express from 'express'

const app = express()

app.get('/', (request, response) => {
  return response.json({ message: 'Hello World' })
})

app.listen(3333)
```

Mas como o Node não entende Typescript, então é preciso converter o `index.ts` em `index.js` antes de executá-lo.
Para isso, vamos usar o pacote `typescript` que já vem com um binário dentro de `node_modules/.bin`.
Rodar:
```bash
yarn tsc src/index.ts
```
E criou um `index.js`, mas deu um erro.

Assim como babel, temos que criar um arquivo `tsconfig.json` com algumas configurações padrão
```bash
yarn tsc --init
```

Agora não precisa mais indicar qual o arquivo `.ts` que você quer converter em `.js`. É só colocar no terminal que ele já acha todos arquivos `.ts` do projeto, sem dar erro.
```bash
yarn tsc
```

E agora podemos rodar o Node
```bash
node src/index.js
```

E podemos acessar no browser `localhost:3333`

Deleta o arquivo `src/index.js` e no `tsconfig.json` descomenta a seguinte informação e adiciona `dist`:
```json
{
  "outDir": "./dist",
}
```

E roda de novo `yarn tsc` para ser criado então os arquivos de dentro da `dist` iguais ao da `src`.

**Quando criar tipagem?**

Como instalamos o `@types/express`, temos as tipagens já do `express`.
Ao passar o mouse em cima do `'express'` e dar `Cmd + click`, abre-se um arquivo `index.d.ts` que traz a tipagem de cada variável do `express`.
Arquivos com extensão `.d.ts` significa que são arquivo de definição de tipagem.
Tudo que usa de dentro da lib, no mesmo arquivo, não precisa tipar pois o VSCode entende. Quando vamos para outro arquivo onde não está sendo chamado o `express` por exemplo, que o VSCode se perde na tipagem.

Vamos separar a função de retorno da rota get criando um arquivo `src/routes.ts`
```ts
export function helloWorld (request, response) {
  return response.json({ message: 'Hello World' })
}
```

E as bibliotecas já exportam os tipos também. Então, é só importar os tipos e usá-los. E todos os métodos já ficam acessíveis, facilitando a manutenção do código.
```ts
import { Request, Response } from 'express'

export function helloWorld (request: Request, response: Response) {
  return response.json({ message: 'Hello World' })
}
```

Dentro de `src/services` temos conjuntos isolados (funções) que executam alguma regra de negócio e retornam um resultado.
Exemplo, um `service` de criação de usuário `CreateUser.ts`.
Para criar um usuário, iremos utilizar o `name`, `email` e `password`.
```ts
export default function createUser(name = '', email: string, password: string) {
  const user = {
    name,
    email,
    password,
  }

  return user
}
```

E ao importar essa função em `src/routes.ts`, o próprio VSCode avisa que está faltando parâmetro e ainda verifica se o tipo do parâmetro está correto.
```ts
import { Request, Response } from 'express'
import createUser from './services/CreateUser'

export function helloWorld (request: Request, response: Response) {
  const user = createUser('Cintia', 'cintiafumi@gmail.com', '123456')

  return response.json({ message: 'Hello World' })
}
```

Mas seria melhor se tivesse o nome de cada parâmetro. Então, faz-se a desestruturação dos parâmetros na função, mas agora o typescript se perde porque entende que está querendo mudar o nome da variável por outra. Então, é melhor criar uma variável separada, o que é chamado de `interface`, que traz o tipo de um conjunto de informações.

```ts
interface CreateUserData {
  name?: string;
  email: string;
  password: string;
}

export default function createUser({ name = '', email, password }: CreateUserData) {
  const user = {
    name,
    email,
    password,
  }

  return user
}
```
Sendo `name?` opcional, por isso pode já deixar na função o parâmetro recebendo uma string vazia caso não receba esse parâmetro, senão virá como undefined.

E na hora que chama a função, já dá para ver cada propriedade que é esperada. E no `console.log(user.)` também já é possível ver as propriedades de `user` com `Ctrl + space`.
```ts
import { Request, Response } from 'express'
import createUser from './services/CreateUser'

export function helloWorld (request: Request, response: Response) {
  const user = createUser({
    email: 'cintiafumi@gmail.com',
    password: '123456',
  })

  console.log(user.password)

  return response.json({ message: 'Hello World' })
}
```

**Tipagem de um vetor**

Ao adicionar `techs` na criação de user,
```ts
  const user = createUser({
    email: 'cintiafumi@gmail.com',
    password: '123456',
    techs: ['Node.js', 'ReactJS', 'React Native'],
  })
```
Podemos definir a tipagem do vetor da seguinte forma:
```ts
interface CreateUserData {
  name?: string;
  email: string;
  password: string;
  techs: Array<string>
}
```
Ou
```ts
  techs: string[]
```

Mas ao adicionar um objeto dentro de techs:
```ts
  const user = createUser({
    email: 'cintiafumi@gmail.com',
    password: '123456',
    techs: [
      'Node.js',
      'ReactJS',
      'React Native',
      { title: 'Javascript', experience: 100 },
    ],
  })
  ```
Precisamos criar então a tipagem desse objeto:
```ts
interface TechObject {
  title: string;
  experience: number;
}

interface CreateUserData {
  name?: string;
  email: string;
  password: string;
  techs: Array<string | TechObject>
}
```