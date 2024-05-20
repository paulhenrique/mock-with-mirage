---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Arquitetura Front-end orientada a Dados
# info: 
# apply any unocss classes to the current slide
# class: text-center
# https://sli.dev/custom/highlighters.html
# highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
colorSchema: dark
aspectRatio: 16/9
themeConfig:
  primary: '#5d8392'
plantUmlServer: 'https://www.plantuml.com/plantuml'

layout: cover
---

# Arquitetura Front-end orientada a Dados


---

<div class="flex gap-5 flex-col items-start">


<div class=" p-5 flex-1">

<div class="bg-[#1f1f1fcc] p-2 px-5 rounded flex gap-5 flex-row items-start">
<div class="flex flex-row gap-2 items-center justify-between">
  <span><img src="/images/paulo.jpeg" class="w-20 rounded-full" /></span>
  <div class="flex flex-col">
    Paulo Cândido
    <span class="text-sm">
      @paulhenrique
    </span>
  </div>
  <img src="/images/qr_code.png" class="w-15">
</div>
</div>

```json
{ 
  "name": "Paulo Cândido",
  "professional": [
    "Senior Frontend Develiper no CPQD",
    "Organizador do Front in Campinas",
    "Membro da Liga voluntária do MWPT"
  ],
  "technologies": [
    "React", "Vue", "Typescript", "3D Maps", "Progressive Web Apps" 
  ],
  "likes": [
    "Animes", "Séries", "The Office", "Musician"
  ]
}
```
</div>

</div>

<!-- <div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/paulhenrique" target="_blank" alt="GitHub" title="Abrir perfil no github"
    class="text-xl slidev-icon-btn !border-none !hover:opacity-90">
    <carbon-logo-github />
  </a>
</div>   -->

---
hide: true
---

# Agenda
- A API nunca está pronta quando começamos o frontend;
- Mock, o salvador da pátria;
- Miragem, uma forma de resolver o problema;
- Arquitetura voltada a mutação de dados;
  - Isolando serviços; 
  - Isolando chamada de API;
  - Mock com types;

---
layout: cover
---

# São dois caminhos

Ou API existe e precisamos construir uma interface para interagir com ela, ou (o que é mais comum) a API e Interface serão construídas ao mesmo tempo. 

## É sobre isso, e tá tudo bem!

---
layout: cover
---

# Mock, o salvador da pátria

<div v-click>

## Vamos pensar em uma aplicação de consulta de usuários.

</div>

---
layout: intro
---

# Usuário 
- ## <span v-mark.line.red="1">Buscar</span> usuários
- ## <span>Cadastrar</span> usuários
- ## <span>Atualizar</span> usuários
- ## <span>Excluir</span> usuários 

---
transition: slide-up
level: 2
---

# O comum
Criar um serviço que retorna os usuários

<br/>

```ts{|2,7-9}
export const ENDPOINTS = {
  USER: "/api/users"
}

const UserService = {
  findAll: (): User[] => {
    const response = await fetch(ENDPOINTS.USER);
    const data = await response.json();
    return data;
  },
}
```

---
transition: slide-up
level: 3
---

# Criando um *"mock"*
Cria um JSON para o mock, o que é <span v-mark.line.red="1">bastante interessante</span>

```json 
{
  "users": [
    {
      "id": 1,
      "username": "anakin",
      "email": "anakin@example.com"
    },
    {
      "id": 2,
      "username": "padme",
      "email": "padme@example.com"
    },
     {
      "id": 2,
      "username": "kenobi ",
      "email": "kenobi  @example.com"
    }
  ]
}
```

---
transition: slide-up
level: 3
---

# Ainda no comum
Os devs colocam um retorno direto <span v-mark.line.red="0">substituindo a chamada do serviço</span>

```ts{all|3|11-13}
import UsersJson from './mock/users.json';

const MOCK_ACTIVE = true; // define se o mock será aplicado ou não

export const ENDPOINTS = {
  USER: "/api/users"
}

const UserService = {
  findAll: (): User[] => {
    if(MOCK_ACTIVE) {
      return UsersJson.users;
    }
    const response = await fetch(ENDPOINTS.USER);
    const data = await response.json();
    return data;
  },
}
```

---
transition: slide-up
level: 3
layout: intro
---
# Como escalar isso? 
<span v-click>

## Simples, não escala

</span>

---
layout: two-cols
transition: slide-up
---

<div class="pr-4">

```ts{|2}
import UsersJson from './mock/users.json';

const MOCK_ACTIVE = true;

export const ENDPOINTS = {
  USER: "/api/users"
}

const UserService = {
  findAll: (): User[] => {
    if(MOCK_ACTIVE) {
      return UsersJson.users;
    }
    const response = await fetch(ENDPOINTS.USER);
    const data = await response.json();
    return data;
  },
}
```
antes
</div>

::right::
<div v-click >

```ts
import ApiServices from 'services/ApiServices';
export const ENDPOINTS = {
  USER: "/api/users"
}

const UserService = {
  findAll: (): User[] => {
    return ApiServices.get(ENDPOINTS.USER);
  },
}
```
depois

</div>

---

# Isolando os serviços de API
O nascimento do API Services

```ts {|2-5}
const apiServices = {
  find: async (endpoint: string) => {
    const response = await fetch(endpoint);
    const data = await response.json();
    return data;
  },
  create: async <T>(endpoint: string, data: T) => {
    const response = await fetch(endpoint, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    });
    const dataResponse = await response.json();
    return dataResponse;
  },
  delete: async <T>(endpoint: string, data: T) => {
    const response = await fetch(endpoint, {
      method: "DELETE",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    });
    const dataResponse = await response.json();
    return dataResponse;
  },
};

export default apiServices;
```
---
layout: intro
---

# Onde colocar os mocks agora?
Uma dúvida cruel, por que não os isolar?


---
layout: intro
---

# Uma miragem

<span v-click>

## O Mirage JS

</span>

---
transition: slide-up
---

<div v-click>

## Manipulação de Rotas
<br>

```ts
this.get("/users", { users: ["Anakin", "Padme", "Kenobi"] })
```

</div>
<br>
<div v-click>

## APIs em diferentes hosts
<br>

```ts
routes() {
  this.urlPrefix = 'http://localhost:3000';
...        
```
</div>
<br>
<div v-click>

## Manipulação de retorno com função
<br>

```ts
this.get("/users", (schema, request) => {
  return ["Anakin", "Padme", "Kenobi"]
})
```

</div>

---

## Manipulando operações HTTP 

```ts
this.get('/users', () => { ... });
this.post('/users', () => { ... });
this.patch('/users/:id', () => { ... });
this.put('/users/:id', () => { ... });
this.del('/users/:id', () => { ... });
this.options('/users', () => { ... });
```
<br>

<div v-click>

## Timing 
```ts
this.get(
  "/users",
  () => {
    return ["Anakin", "Padme", "Kenobi"]
  },
  { timing: 4000 }
)
```

</div>

---
transition: slide-up
layout: intro
---

# Acessando a camada de dados 

---
transition: slide-up
---


## Retornando dados do Schema 
```ts
this.get("/users", (schema) => {
  return schema.users.all()
})
```
<br>

<div v-click>

## Buscando com parâmetros 
```ts
this.get("/users/:id", (schema, request) => {
  let id = request.params.id

  return schema.users.find(id)
})
```
</div>

<div v-click>
<br>

## Requisições com body
```ts
this.post("/movies", (schema, request) => {
  let attrs = JSON.parse(request.requestBody)

  return schema.movies.create({ attrs })
})
```
</div>

---
layout: intro 
transition: slide-up
---

# Precisamos disso tudo apenas para um mock?
<span v-click>

## A resposta é que: <span v-mark.line.red="1">Depende</span>

</span>

---

## Simples

```ts
import UsersJson from './mock/users.json';
import { createServer } from "miragejs"

createServer({
  routes() {
    this.namespace = 'api';
    this.get("/users", UsersJson.users);
  }
})
```
<div v-click class="flex gap-5 pt-5">

<div class="min-w-100">
Nada muda no nosso serviço de usuários

```ts
import ApiServices from 'services/ApiServices';
export const ENDPOINTS = {
  USER: "/api/users"
}

const UserService = {
  findAll: (): User[] => {
    return ApiServices.get(ENDPOINTS.USER);
  },
}
```
</div>

<div v-click class="min-w-100">
Nem nos serviços da API

```ts
const apiServices = {
  find: async (endpoint: string) => {
    const response = await fetch(endpoint);
    const data = await response.json();
    return data;
  },
  ...
};
```

</div>

</div>

---
layout: intro
---

# Isso escala? 
Sim, e de forma bem mais Simples

---
layout: intro
---

# Por enquanto é só!
## Muito obrigado!

<div class="bg-[#1f1f1fcc] p-2 px-5 rounded flex gap-5 flex-row items-start position-fixed left-15 bottom-15">
<div class="flex flex-row gap-2 items-center">
  <span><img src="/images/paulo.jpeg" class="w-20 rounded-full" /></span>
  <div class="flex flex-col">
  Paulo Cândido
  <span class="text-sm">
    @paulhenrique
  </span>
  </div>
  <img src="/images/qr_code.png" class="w-15">
</div>
</div>
