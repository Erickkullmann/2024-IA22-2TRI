### 2024 - IA22 - 2TRI

# Iniciando um projeto Node.js com TypeScript

### abra seu arquivo do Github e entre no codeSpace,em seguida abra seu `Terminal` (Ctrl+Shift+U).

- Depois de abrir o trminal copie e cole este codigo a seguir

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## agore configure o arquivo `tsconfig.json`

Altere a linha  ```"outDir": "./",``` para ```"outDir": "./dist",``` e insira a linha ```"rootDir": "./src",```, logo abaixo. O seu arquivo de configuração do TypeScript deve se parecer com o exemplo abaixo.


```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

## Configurando o `package.json`

Adicione o seguinte script ao seu `package.json`

```json
"scripts": {
  "dev": "npx nodemon src/app.ts"
}
```

## Criando arquivo inicial para seu servidor

Adicione o seguinte código ao arquivo `src/app.ts`

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inicializando o servidor

```bash
npm run dev
```
Caso tudo funcione corretamente, a mensagem `Server running on port 3333` aparecerá no terminal.

## Testando o servidor

Abra seu navegador e acesse `http://localhost:3333`, se tudo tiver certo verá a mensagem `Hello World`.

# Configurando o Banco de Dados

### Crie um arquivo chamado *`database.ts`* dentro da pasta `src` e insira o seguinte código:

```typescript
import { Database, open } from 'sqlite';
import sqlite3 from 'sqlite3';


let dbInstance: Database | null = null;


export async function connect() {
  
  if (dbInstance) return dbInstance;

  
  const db = await open({
    filename: './src/database.sqlite',
    driver: sqlite3.Database
  });


  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);

  
  dbInstance = db;
  return db;
}
```


# Adicionando o banco de dados ao servidor

### no arquivo _`App.ts`_ insira o seguinte código:

``` ts
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;

  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);

  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

Crie um novo arquivo chamado `Teste.http` dentro desse arquivo insira o código:

### insira um *_`Insira um nome e email`_*

``` ts
POST http://localhost:3333/users HTTP/1.1
Content-Type: application/json



{
  "name": " John Doe ",
  "email": " "
}

### Se tudo ocorrer bem, você verá a resposta com o usuário inserido.

``` ts
{
  "id": 1,
  "name": " John Doe",
  "email": "
}
```

# Listando os usuários
## Adicione a rota `/users` ao *`app.ts`*

``` ts
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
}
);

```

# Editando um usuário
## Adicione a rota `/users/:id` ao *`app.ts`*.

``` ts 
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});

```

# Deletando um usuário
## Adicione a rota `/users/:id` ao *`app.ts`*

``` ts

app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});

```
# crie um novo arquivo chamado *`test.http`*
## copie e cole o codigo a seguir o codigo a seguir:

``` ts
POST http://localhost:3333/users HTTP/1.1
content-type: application/json

{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}

####

PUT http://localhost:3333/users/2 HTTP/1.1
content-type: application/json

{
  "name": "John Doe Updated",
  "email": "johndoe@mail.com"
}

####

DELETE http://localhost:3333/users/1 HTTP/1.1

```

### crie uma nova pasta chamada  *`public`*

### agora dentro da posta crie o *`index.html`* e insira o código:

``` ts
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" name="name" placeholder="Nome">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>

  <script>
    // 
    const form = document.querySelector('form')

    form.addEventListener('submit', async (event) => {
      event.preventDefault()

      const name = form.name.value
      const email = form.email.value

      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })

      form.reset()
      fetchData()
    })

    // 
    const tbody = document.querySelector('tbody')

    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()

      tbody.innerHTML = ''

      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `

        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')

        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })

        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)

          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          fetchData()
        })

        tbody.appendChild(tr)
      })
    }

    fetchData()
  </script>
</body>

</html>

```
# na pasta *Src* em *`app.ts`*insira o seguite codigo
### "app.use(express.static(__dirname + '/../public'))"
 ## se tiver tudo certo ficará assim: 
``` ts 
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors())
app.use(express.json())
app.use(express.static(__dirname + '/../public'))

 ```
 # agora por ultimo salve(Ctrl+S) e comite o código no terminal copie e cole no terminal o codigo abaixo
 
 ``` ts
gadd --all; git it commit -m ...; git push 

 ```