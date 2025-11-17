# Projeto barbearia



    Iniciamos criando um projeto next.js, diretamente no vscode, com o seguinte comando;

```
// Criação da pasta do projeto
mkdir projeto-barbearia

// Abrir pasta do projeto
cd projeto-barbearia
```

Setup next.js [https://nextjs.org/]()

    Iniciamos o projeto com o setup do next com o seguinte comando;

```bash
npx create-next-app@latest .
```

```bash
carlos@cs-ubuntu:~/Documentos/projeto-barbearia$ npx create-next-app@latest .
Need to install the following packages:
create-next-app@15.5.6
Ok to proceed? (y) y

✔ Would you like to use TypeScript? … No / Yes
✔ Which linter would you like to use? › ESLint
✔ Would you like to use Tailwind CSS? … No / Yes
✔ Would you like your code inside a `src/` directory? … No / Yes
✔ Would you like to use App Router? (recommended) … No / Yes
✔ Would you like to use Turbopack? (recommended) … No / Yes
✔ Would you like to customize the import alias (`@/*` by default)? … No / Yes
Creating a new Next.js app in /home/carlos/Documentos/projeto-barbearia.

Using npm.

Initializing project with template: app-tw 


Installing dependencies:
- react
- react-dom
- next

Installing devDependencies:
- typescript
- @types/node
- @types/react
- @types/react-dom
- @tailwindcss/postcss
- tailwindcss
- eslint
- eslint-config-next
- @eslint/eslintrc
```

    Abrimos com o vscode com o comando;

```bash
code .
```

    E temos nosso projeto criado;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-21%2020-52-40.png)

    Rodamos o projeto com;

```bash
npm run dev
```

## 

## Setup Prisma

    O Prisma trata-se de um ORM que utilizaremos para gerênciar o banco de dados, ele nos ajuda a lidar com as tabelas, com migrations, etc...

    Setup do Prisma, [https://www.prisma.io/docs]()

    Instalamos o Prisma com o seguinte comando;

```bash
carlos@cs-ubuntu:~/Documentos/projeto-barbearia$ npm install prisma --save-dev
```

    Então rodamos um;

```bash
npx prisma init --datasource-provider postgresql
```

    Isso irá informar ao Prisma que usaremos um postgresql como banco de dados.

    Será criado um arquivo chamado schema.prisma no projeto.

```js
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

// Looking for ways to speed up your queries, or scale easily with your serverless or edge functions?
// Try Prisma Accelerate: https://pris.ly/cli/accelerate-init

generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

    Também cria um arquivo .env com uma variável de ambiente chamada;

```bash
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
```

    Aqui colocaremos a conection string do postgresql.

    Para hospedar o postgre usaremos o supabase.



    [https://supabase.com/]()



        Com o supabase conseguimos criar um banco de dados e rostear de forma gratuita.

    Acessamos o supabase, autenticamos, criamos novo projeto.

    Senha do banco;

```bash
mMZi0eLtnOrYdCtX
```

## Banco de dados, Tabelas

    Precisamos pensar em como iremos formar as tabelas do banco de dados.

    Pensar no diagrama do banco de dados.

    Para isso usaremos o diagrans.net

[https://app.diagrams.net/]()

    A primeira tabela a ser criada será usuário com;

* ID

* Name

* Email



Depois analisamos o protótipo para decidir qual a próxima tabela, que será barbearia, contendo;

* ID

* Name

* Address

* Services

* Image URL



    Na barbearia teremos uma relação de one-to-many, onde uma barbearia pode ter vários serviços, e um serviço pertence a uma barbearia.

    Em um ambiênte de SQL puro, seria necessário a criação de uma outra tabela, chamada 'barbershop services', que seria responsável por fazer a relação entre a barbearia e os serviços.

<mark>Sempre salvar a representação URL da imagem no banco.</mark>

    Cada barbearia terá seus próprios serviços, como corte, barba, etc...

    Por isso criamos uma tabela que conterá os serviços de cada barbearia, contendo;

* ID

* Barbershop ID, pois cada serviço pertence a uma barbearia;

* Service ID, pois uma reserva terá uma barbearia e um serviço;

* User ID, o usuário que reservou o serviço.



    Teremos uma relação de one-to-many entre a tabela Services e o atributo Services da tabela barbearia.

    Reservas, onde um usuário poderá reservar em várias barbearias, e uma barbearia poderá ter várias reservas.

    Criamos a tabela booking, agendamento.

    Nessa tabela teremos;

* ID

* Barbershop ID, id da barbearia

Tabelas criadas;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-21%2021-51-20.png)

    Este foi o planejamento inicial da aplicação, que é muito importânte de ser feito pois assim evita que tenhamos retrabalho na codificação do projeto.

    Iniciaremos a criação das tabelas no Prisma, no arquivo schema.prisma.



### Tabela User

Iniciamos criando a tabela user;

```dataweave
model User {

  id String @id @default(uuid())

  name String
}
```

    Na tabela User temos o id e o name.

    Importante que dizemos ao prisma que o id trata-se de um id primario, que não pode se repetir, que será um uuid, então caso criamos um usuário sem um id, o id será um id aleatório criado pelo próprio código.

### Tabela Barbershop

    O id segue o mesmo padrão de criação;

```dataweave
model Barbershop {
  id String @id @default(uuid())
  name String
  address String
  imgUrl String
}
```

### Tabela Service

    Nesta tabela precisamos salvar o preço, para isso na variável price usamos o decimal, que é muito usado no Postgree para salvar dinheiro, pois conseguimos colocar as casas decimais. Definimos que antes da virgula teremos no máximo 10 caracteres, e depois da virgula dois caracteres.

    Já para a barbershop teremos algo específico do prisma, onde o barbershop, a chave estrangeira, não será salva no banco, isso é apenas para o prisma saber que temos uma relação entre a variável barbershop, que está dentro da tabela Service, e a tabela Barbershop

    Então dizemos que a relação entre o model Barbershop e o service é que o barbershop Id, que é um parâmetro dessa variável aponta para o id do Barbershop.

    Então o `barbershopId String` seria a chave estrangeira, e o id seria a primaryKey da Barbershop;

```dataweave
model Service {
  id String @id @default(uuid())
  name String
  price Decimal @db.Decimal(10, 2)

  // ForeignKey, chave estrangeira
  barbershopId String
                                                                      // id chave primaria da tabela Barbershop
  barbershop Barbershop @relation(fields: [barbershopId], references: [id])
}
```

    Precisamos disso para saber a relação entre as duas tabelas.

### Tabela Booking

* Aqui teremos um userId, que vai ser o usuário que fez a reserva;

* Teremos a relação entre o userId, dessa reserva com o id do usuário;

* Mesma relação com service;

* E também o Barbershop

```dataweave
model Booking {
  id String @id @default(uuid())
  userId String
  user User @relation(fields: [userId], references: [id])
  serviceId String
  service Service @relation(fields: [serviceId], references: [id])
  date DateTime
  barbershopId String
  barbershop Barbershop @relation(fields: [barbershopId], references: [id])
}
```

    Após a criação das tabelas, rodamos um `npx prisma format` e será feita algumas formatações.

    Uma delas é que como uma barbearia vai ter vários serviços, precisamos especificar isso, assim é criado um array, onde teremos que uma barbearia vai ter services, e que cada service será um model service, um array de services.

    Então;

* Um usuário vai ter vários agendamentos

```dataweave
model User {
  id       String    @id @default(uuid())
  name     String
  bookings Booking[]
}
```

* A barbearia vai ter vários agendamentos e vários serviços também;

```dataweave
model Barbershop {
  id      String    @id @default(uuid())
  name    String
  address String
  imgUrl  String
  services Service[]
  bookings Booking[]
}
```

* Um serviço vai pertencer a uma barbearia;

```dataweave
model Service {
  id           String     @id @default(uuid())
  name         String
  price        Decimal    @db.Decimal(10, 2)
  barbershopId String
  barbershop   Barbershop @relation(fields: [barbershopId], references: [id])
  description  String
  bookings      Booking[]
}
```

* E um agendamento vai pertencer a um usuário, um serviço e a uma barbearia;

```dataweave
model Booking {
  id           String     @id @default(uuid())
  userId       String
  user         User       @relation(fields: [userId], references: [id])
  serviceId    String
  service      Service    @relation(fields: [serviceId], references: [id])
  date         DateTime
  barbershopId String
  barbershop   Barbershop @relation(fields: [barbershopId], references: [id])
}
```

    Voltamos ao supabase, que já criou o projeto-barbearia.

    Clicamos em settings e database, onde teremos nossa URL.

    Procuramos por connection, url database.

    Copiamos a URL no arquivo .env e inserimos o password do banco.

`DATABASE_URL="DATABASE_URL=postgresql://postgres:mMZi0eLtnOrYdCtX@db.qfquypkpffjwglhptbzk.supabase.co:5432/postgres"`



## Migration

    Feito isso rodamos o comando que irá criar nossa primeira migration, que nada mais é que uma alteração no banco de dados.

    Uma mudança no estado do banco de dados, que atualmente está no estado vazio, migramos o banco de um estado para outro.

    Comando;

`npx prisma migrate dev --name add-initial-tables`

    Assim foi criada todas as tabelas no banco, podemos conferir rodando o;

`npx prisma studio`

    E veremos todas as tables criadas.

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-27%2019-40-05.png)

    Agora iremos fazer o primeiro commit, colocando o .env no gitgnore, apenas digitando o .env dentro do arquivo gitgnore;

    Para fazer o commit usamos o convention, um padronizador de mensagens dos commits;

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)

 

## Populando o banco de dados

    Usamos o arquivo seed para popular o banco de dados, adicionando as barbearias e etc...

    Dentro da pasta Prisma, criamos um arquivo chamado seed.ts, colamos o conteúdo do arquivo seed dentro deste arquivo;

    Pegamos o conteúdo do arquivo no git.

    Esse arquivo irá criar as barbearias, serviços de fato para que possamos trabalhar no front.

    Para executar esse arquivo vamos até;

* package.json;

* Adicionamos uma propriedade;

```json
"prisma": {
    "seed": "ts-node prisma/seed.ts"
},
```

    Para funcionar, executar, precisamos instalar o ts.node, que é uma lib que irá executar códigos typeScript;

```bash
npm install -D ts-node
```

    Para executar o seed usamos o seguinte comando;

```bash
npx prisma db seed
```

    Ao tentar executar o seed,  nos deparamos com um erro, o argumento imgUrl não existia na tabela service, no arquivo ts.config, após adicionar o argumento a tabela service, o banco foi devidamente populado.

    Assim podemos fazer o commit;

`chore: add seed script`



## Telas

    Usaremos o shadcn.ui para fazer a interface, trata-se de uma biblioteca com vários componentes para o projeto.

    Uma prática altamente indicada de ser usada nos dias de hoje, pois trata-se de uma lib de componentes para se criar os projetos, isso acelera bastante o desenvolvimento.

    Precisamos instalar, inicializar o shadcn no nosso projeto, com o seguinte código;

```bash
npx shadcn@latest init
```

    Ele cria a pasta lib, com o arquivo utils.ts e o arquivo components.json, arquivos que não precisamos nos preocupar com eles por enquanto.

    Mudamos o caminho dos argumentos, adicionando `/app/_` a cada argumento;

```dataweave
 "aliases": {
    "components": "@/app/_components",
    "utils": "@/app/_lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "registries": {} 
```

### 

## Sistema de rotas do next.js

    Movemos a pasta lib e a pasta component para dentro de app

    Isso seria uma boa prática de programação do next.js.

    Para o sistema de rotas do next, quando temos um arquivo page.ts dentro da pasta app, é tratado como uma página inicial, home page.

    Então todo arquivo page.tsx trata-se de uma homePage, isso significa a primeira página a ser carregada.

    Podemos constatar isso alterando o arquivo page.tsx, apagando seu conteúdo e colocando apenas um h2 qualquer;

```js
import Image from "next/image";

export default function Home() {

  return <h1>Home Page</h1>

}
```

E temos o resultado;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-30%2020-17-55.png)

    Se criarmos uma pasta chamada about e dentro desta página tivermos um arquivo qualquer, por exemplo, about.tsx, podemos adicinar qualquer coisa dentro dele, e isso será outra rota, testando, adicionamos  a pasta e o arquivo, e podemos acessar via navegador, a partir da rota correspondente;

    Testamos criando outra rota, about.tsx, criando uma pasta chamada About e dentro desta pasta temos o arquivo page.tsx, apenas com um h1 escrito `página about`, depois conseguimos acessar essa página no navegador, usando;

`https://localhost3000/about`

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-30%2020-26-22.png)

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-30%2020-26-37.png)

    Ou seja, toda pasta que criamos dentro de app, que possua um arquivo page.tsx dentro, é tratada como uma rota no next.js .

    Para pastas que não iremos tratar como uma rota, é aconselhavel colocar um underline antes do nome, Para ficar claro que esta pasta não será uma rota e sim uma pasta normal;

    Sendo assim, renomeamos as pastas lib e components dentro da pasta app;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-10-30%2020-31-41.png)

    Agora alteramos o arquivo .css, para ter as cores que usaremos em nosso projeto, para isso copiamos o conteúdo do arquivo globals.css, para termos as cores corretas.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  .dark {
    --background: 225 9% 9%;
    --foreground: 210 40% 98%;

    --card: 228 9% 11%;
    --card-foreground: 210 40% 98%;

    --popover: 228 9% 11%;
    --popover-foreground: 210 40% 98%;

    --primary: 252 100% 69%;
    --primary-foreground: 0 0% 100%;

    --secondary: 228 6% 16%;
    --secondary-foreground: 210 40% 98%;

    --muted: 228 6% 16%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 228 6% 16%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 84% 60%;
    --destructive-foreground: 210 40% 98%;

    --border: 228 6% 16%;
    --input: 228 6% 16%;
    --ring: 212.7 26.8% 83.9%;

    --radius: 0.5rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground antialiased h-screen flex flex-col;
  }
}
```

    Este arquivo usa um sistema de cores hsl.

    Feito isso, já alteramos a cor do nosso plano de fundo, alterando a classe na nossa página inicial;

`Neste ponto tivemos outro erro, a versão do tailwind não era compátivel com o projeto, que deveria ser a versão 3 do tailwind.`

`Após pesquisar bastante, foi solucionado instalando a versão 3.alguma coisa do tailwind, e assim o projeto rodou sem erros.`

    Precisamos alterar o arquivo layout.tsx para aplicar o tema dark ao nosso projeto, para isso adicionamos a classe neste ponto do código;

```html
    <html lang="en">
      <body className={`${inter.className} dark`}>
        {children}
      </body>
    </html>
```

    Interessante que se remover-mos o dark, o tema black é removido.

    Este foi o setup inicial do shadcn.



Commit;





# Componentes

## Header Inicial

    Para iniciar, baixamos a imagem do logo no repositório do figma;

    Exportamos como logo.png, 2x para ter uma melhor qualidade.

    Movemos o logo.png dentro da pasta public do projeto.

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-02%2011-59-58.png)

    O próxio passo é colocar o arquivo page.tsx, que é a página inicial do projeto em uma pasta separada, nomeada (home).

    Criamos a pasta dentro de app, e movemos o page.tsx para esta pasta.

    Assim quando temos uma pasta (home) com um page.tsx dentro, torna-se um rout group. 

    Se colocar-mos apenas home sem parênteses, e acessar localhost.3000, não teremos mais um routgoup, e sim um /home, mas se tivermos um routgroup com home entre parênteses, o comportamento muda, é como se o nome da pasta não interferisse na rota.

    Isso serve para organizar bem o projeto.

    Assim podemos criar toda a extrutura da página home dentro desta pasta.

    Criamos então a pasta components, dentro da pasta home, e será onde colocaremos todos os componentes da página incial da home page. 

    Também dentro da pasta components, da pasta app, serão colocados os componentes que farão parte de mais de uma tela.

    Então dentro desta pasta, _components, da pasta app, criamos o arquivo header.tsx;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-02%2012-13-14.png)

    Adicionamos uma extensão para facilitar a codificação;

`Simple React Snnipet`

    Assim, com o atalho sfc, já cria uma função para  criar o componente header.

```javascript
const Header{
    return (  );
}

export default Header
```

    Para a criação do card que irá conter o header, usamo o shadcn, selecionando card, e executando o comando de criação do componente no terminal do projeto;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-02%2012-19-23.png)

    Ao executar o código, será criado uma pasta UI dentro do _components, com o arquivo card.tsx dentro.

    O shadcn coloca o componente dentro do projeto, todo o código.

    Agora retornamos o <Card> dentro da função do arquivo header.tsx.

    Dentro do card colocamos a imagem, como uma Imagem next.js, de logo que baixamos do figma.

```js
import Image from "next/image";
import { Card } from "./ui/card";

const Header () => {
    return ( 
        <Card>
            <Image src="/Logo.png" alt="FSW Barber" height={22} width={120} />
        </Card> 
    );
}

export default Header;
```

* Adicionamos um alt;

* Adicionamos o height e o width, encontrado no figma também.



    Usamo o shad para adicinar o component button também com o comando;

`npx shadcn@latest add button`

    Alteramos o código para salvar o header dentro de cardcontent, uma maneira mais organizada de contruir o código;

    Instalamos o component buttom, do shad;

    Assim podemos ir adicionando os componentes sob demanda.

    Inserimos a imagem e o buttos dentro de cardcontent, que seria uma forma correta de usar o componente.

```js
import Image from "next/image";
import { Card, CardContent } from "./ui/card";
import { Button } from "@/app/_components/ui/button";
import { MenuIcon } from "lucide-react";

const Header = () => {
    return ( 
        <Card>
            <CardContent>
                <Image src="/Logo.png" alt="FSW Barber" height={22} width={120} />
                <Button variant="outline" size="icon">
                    <MenuIcon/>
                </Button>
            </CardContent>
        </Card> 
     );
}

export default Header;
```

* Adicionamos o card content;

* Dentro de CardContent adicionamos a imagem;

* Adicionamos o Button;

* Adiconamos o MenuIcon.

    Agora precisamos adicionar o header à página inicial, então, no arquivo page.tsx adicionamos o Header;

```js
<Header/>
```



### Estilizar o header com tailwind

    O tailwind é uma forma de estilizar os components utilizando classes.

    Por exemplo, se quisermos adicionar padding a um componente, podemos pesquisar por padding no site do tailwind, e adicionar a classe correspondente;

    Para o header usamos;

* padding p-5;

* justify-content space bettwen;

* display flex;

* flex-row;

* itens-center alinha os itens ao centro

`<CardContent className="px-5 py-8 justify-between flex flex-row"">`

* Alteramos o tamanho do MenuIcon para;

`<MenuIcon size={18}/>`

* Alteramos o tamanho do button para 
  
  `className="h-8 w-8"`

Código completo;

```js
import Image from "next/image";
import { Card, CardContent } from "./ui/card";
import { Button } from "@/app/_components/ui/button";
import { MenuIcon } from "lucide-react";

const Header = () => {
    return ( 
        <Card>
            <CardContent className="p-5 justify-between items-center flex flex-row">
                <Image src="/Logo.png" alt="FSW Barber" height={22} width={120} />
                <Button variant="outline" size="icon" className="h-8 w-8">
                    <MenuIcon size={18}/>
                </Button>
            </CardContent>
        </Card> 
     );
}

export default Header;
```

Neste ponto fizemos;

    Adicionamos um card, parte inicial do header da página inicial, usando o shadcn para criar os componentes.

    Usamos o <card> e dentro dele o <CardContent>.

    Dentro do CardContent colocamos a imagem da logo e o botão do menu lateral.

    Estilisamos usando as classes do tailwind.

    Por fim o commit;



## Barra de busca com o texto de boas vindas

    Direto na página home, no arquivo page.tsx, criamos apenas o texto que irá aparecer na tela, como um boas vindas para o usuário.

    Apenas como teste criamos um h2 para conter o nome do usuário, que posteriormente será substituído pelo usuário autênticado.

    Abaixo do nome teremos a data formatada, e para isso usaremos uma lib chamada date-fns;

    Instalamos a lib;

`npm i date-fns `

    Importamos desta date-fns o format;

`import { format } from "date-fns";`

    Então usamos o format do date-fns para formatar nossas datas, acessando a documentação podemos definir com cada ítem da data irá aparecer na tela, 

    Então formamos a data completa através desta lib, desta forma;

<p className="capitalize">{format(new Date(), "EEEE',' dd 'de 'MMMM", {locale: ptBR,} )}</p>

```js
<div className="px-5 py-5">
    <h2 className="text-xl font-bold">Ola, Carlos!</h2>
    <p className="capitalize text-sm">{format(new Date(), "EEEE',' dd 'de 'MMMM", {
        locale: ptBR, 
        })}
    </p>
</div>
```



![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-02%2015-09-17.png)

    Agora precisamos estilisar este componente com o tailwind.

No h2, no título;

```js
 <h2 className="text-xl font-bold">Ola, Carlos!</h2>
```

    Aqui colocamos;

* text-xl;

* font-bold

    E no parágrafo colocamos;

```js
<p className="capitalize text-sm">{format(new Date(), "EEEE',' dd 'de 'MMMM", {
   locale: ptBR, 
} )}</p>
```

    Neste parágrafo;

* Temos a data formatada;

* capitalize, que coloca a primeira letra maiúscula;

* text-sm;

    

        Agora colocamos isso tudo dentro de uma div e formatamos-la.

        Precisamos estilisar a div para ter o espaço necessário entre os componentes.

        Colocamos os paddings;

`className="px-5 py-5"`

    Dentro da div;

```js
<div className="px-5 py-5">
  <h2 className="text-xl font-bold">Ola, Carlos!</h2>
  <p className="capitalize text-sm">{format(new Date(), "EEEE',' dd 'de 'MMMM", {
    locale: ptBR, 
  })}
  </p>
</div>
```

    Componente informativo de boas vindas completo.



Commit;

`feat: add wecome message`



### Input

    Para o input criaremos um componente separado, ao ínves de coloca-lo direto na home page, como foi feito com a mensagem de boas vindas.

    Dentro da pasta home criamos uma pasta _components e dentro desta pasta criamos o arquivo search.tsx.

* Baixar o input do shad;

`npx shadcn@latest add input`

```js
import { Button } from "@/app/_components/ui/button";
import { Input } from "@/app/_components/ui/input";
import { SearchIcon } from "lucide-react";

const Search = () => {
    return ( 
        <div className="flex itens-center">
            <Input/>
            <Button variant="default" size="icon">
                <SearchIcon size={18}/>
            </Button>
        </div>
     );
}

export default Search;
```

    Esse componente precisa ser um clientComponent, pois será um componente que terá interatividade, por exemplo, ao clicar em um botão terá algum comportamento.

    Um componente que podemos adicionar interatividade para o usuário.

    Por isso colocamos, entre parênteses um use-client no início do código;

```js
"use client";

import { Button } from "@/app/_components/ui/button";
import { Input } from "@/app/_components/ui/input";
import { SearchIcon } from "lucide-react";

const Search = () => {
    return ( 
        <div className="flex itens-center">
            <Input/>
            <Button variant="default" size="icon">
                <SearchIcon size={18}/>
            </Button>
        </div>
     );
}

export default Search;
```

    No page.tsx adicionamos o Search.

    No page.tsx devemos importar o Search do caminho correto, ou não será carregado;

`import Search from "./_components/search";`



### Estilizando o input

    Adicionamos o placeHolder;

    Colocaremos espaçamento no componente que irá usar o Search, ao ínves de colocar o padding no própio componente Search, pois isso compromete o local onde este componente será usado.

    Preferimos deixar o componente indenpendente, que podemos por o espaçamento que quisermos dentro dele.

    Assim, colocamos o Search dentro de uma div indenpendete.

    Ou seja, não colocamos o padding no componente filho, e sim no componente pai.

```js
<div className="px-5">
  <Search />
</div>
```

* Adicionamos um gap-2, na div que contêm o componente Search.

* Retiramos o size="icon", para que o botão fique menos retangular;

* Tamanho do SearchIcon size{20};

* Na div correspondente, na page.tsx, colocamos uma margen top de mt-5;

    Componente Search finalizado;

```js
"use client";

import { Button } from "@/app/_components/ui/button";
import { Input } from "@/app/_components/ui/input";
import { SearchIcon } from "lucide-react";

const Search = () => {
    return ( 
        <div className="flex itens-center gap-2">
        <Input placeholder="Busque por uma Barbearia..." />
            <Button variant="default">
                <SearchIcon size={20}></SearchIcon>
            </Button>
        </div>
     );
};
export default Search;
```

    Trecho no page.tsx;

```js
<div className="px-5 mt-6">
    <Search />
</div>
```

Commit;

`feat: add initial search component`



## Agendamentos

    Como se trata de um componente usado em mais de uma tela, criamos na pasta;

`app/_components`

    Trata-se de um card, com duas divs;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-03%2020-37-56.png)

    O texto "Confirmado" trata-se de uma badge, que pode ser importada do shadcn;

`npx shadcn@latest add badge`

    Na badge adicionamos apenas a palavra Confirmado.

    E já renderizamos na page.tsx;

    Podemos usar um atalho para criar a div direto, usando;

`.px-5`

    Já cria a div com className px-5;

```js
<div className="px-5 mt-6">
   <BookingItem/>
</div>
```

* Na div apenas padding x de - px-5;

* E margin-top de 6;

```js
import { Badge } from "./ui/badge";
import { Card, CardContent } from "./ui/card";

const BookingItem = () => {
    return ( 
        <Card>
            <CardContent className="p-5">
                <div>
                    <Badge className="bg-[#221C3D] text-primary">Confirmado</Badge>
                </div>
            </CardContent>
        </Card>
     );
}

export default BookingItem;
```

* Padding de 5 no CardContent;

    Na Badge, que é o texto;

* Usamos o background-color do figma, usamos uma cor específica usando [];

* Cor do texto text-primary, variável de cor criada pelo shadcn, que está em globals.css

* Estava com um hover quando passava o mouse encima do texto, desativamos esse hover com esse estilo, `hover:bg-[#221C3D]`, aplicando a mesma cor de fundo.

* Usamos o w-fit para que ele use apenas o tamanho necessário e não o card todo.

    Abaixo da badge vai o serviço, colocamos Corte de Cabelo, mas posteriormente isso virá do banco de dados;

```js
 <div>
    <Badge className="bg-[#221C3D] text-primary hover:bg-[#221C3D]">Confirmado</Badge>
    <h2 className="font-bold">Corte de Cabelo</h
 </div>
```

* font-bold - Negtrito



## Foto da barbearia nome

    Para esse componente, utilizaremos o avatar component, do shadcn.

    Instalamos com esse comando;

`npx shadcn@latest add avatar`

* Criamos uma div com flex definido;

* Tag Avatar, e dentro de Avatar colocamos o AvatarImage;

* src="linkDaImagemDoSeed";

* Usamos h-6 e w-6 para definir o tamanho do avatar;

<mark>Talvez de um erro de dominio de imagem neste ponto.</mark>

    Adicionamos também um AvatarFallBack, para caso o carregamento da imagem falhe;

```js
// Compenente Avatar pronto
<Avatar className="h-6 w-6">
   <AvatarImage src="https://utfs.io/f/0ddfbd26-a424-43a0-aaf3-c3f1dc6be6d1-1kgxo7.png"/>

   <AvatarFallback>A</AvatarFallback>
</Avatar>
```

    Para o nome da barbearia usamos um h3, com as seguintes estilizações;

```html
 <h3 className="text-sm">Vintage Barber</h3>
```

* Adicionamos um text-sm ao h3

* A div que contém o elemento adicionamos;

* Items-center, gap-2;

    Precisamos aplicar um espaçamento de 10 entre os itens.

    Para isso adicionamos;

* flex;

* flex-col;

* gap-2.



## Div com a data e hora

* flex;

* flex-col, um abaixo do outro;

* itens-center, itens centralizados verticalmente;

* justify-center, itens cetralizados horizontalmente.



```js
<div className="flex flex-col items-center justify-center">

</div>
```

    Dentro da div, parágrafos para os itens;

```js
<div className="flex flex-col items-center justify-center">
    <p className="text-sm">Fevereiro</p>
    <p className="text-2xl">06</p>
    <p className="text-sm">09:45</p>
</div>
```

    Agora o card que contém tudo isso precisa ser;

```js
<CardContent className="p-5 flex justify-between">
```

    Adicionamos a borda da div de data;

```js
<div className="flex flex-col items-center justify-center px-3 border-l border-solid border-secondary">
    <p className="text-sm">Fevereiro</p>
    <p className="text-2xl">06</p>
    <p className="text-sm">09:45</p>
</div>
```

### Bordas

* border-l => Borda na esquerda;

* border-solid - linha sólida;

* border-secondary - cor

`Talvez teremos um erro na borda, por conta do tamanho do texto, pois cada mês terá um tamanho de texto, e isso influenciára no tamanho do card onde está o texto com data e hora.`

    Para que a borda fique colada no card, tanto na parte de cima com na de baixo, precisamos alterar os paddings dos elementos CardContent e da div que contém o elemento Badge.

* py-0 => no CardContent;

* py-5 => na div da Badge.

    Adiconamos um h2 com a palavra agendamentos, no componente page.tsx com as

seguintes estilizações;

```js
 <h2 className="text-sm uppercase text-gray-400 font-bold">agendamentos</h2>
```

* text-xs => corresponde a 12px.

Commit;

`feat: add initial booking component`



## Itens da barbearia

    Agora faremos as listas das barbearias, onde teremos um componente que será o item da barbearia, assim como foi com o bookingItem.

    Trata-se de um componente que só será exibido na página inicial, página home, sendo assim criaremos esse componente da pasta (home)  _components. 

    Este componente terá;

* A imagem da barbearia;

* O nome da Barbearia;

* Endereço;

* Botão de reservar;

*  Também será um card.



## Props

    Aqui usaremos props, receberemos como prop o BarberShop, a Barbearia, isso porque queremos exibir um ítem de barbearia para cada barbearia no banco de dados.

    Então recebemos como prop o Barbershop;

```js
import { Card, CardContent } from "@/app/_components/ui/card";

                        // Prop barbershop
const BarbershopItem = ({barbershop}) => {
    return ( 
        <Card>
            <CardContent className="p-0">

            </CardContent>
        </Card>
     );
}

export default BarbershopItem;
```

    Tipamos esses props criando uma interface, que irá receber, retornar o barbershop, antes importamos o Barbershop do prisma client;

```js
import {Barbershop} from "@prisma/client";
```

    Esse barbershop importado será a interface;

```js
import { Card, CardContent } from "@/app/_components/ui/card";
import {Barbershop} from "@prisma/client";

interface BarbershopItem{
    barbershop: Barbershop;
}
```

<mark>Código com a interface e o import.</mark>

    Adicionamos as props ao componente;

```js
const BarbershopItem = ({barbershop}: BarbershopItemProps) => {
```

    E a partir daqui a barbershop passa ter todo o auto complete, pois já recebe do banco as informações. O prisma se responsabiliza com isso.

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-10%2019-48-45.png)

    Testamos criando um título e colocando barbershop.name.

    Para que funcione vamos até a home page, que trata-se de um server component, ou seja, umá página que é renderizada do lado do servidor, consequentemente, desse modo, podemos chamar o banco na própria home page;

    Podemos chamar o prisma dentro da home page e pegar as barbearias.

    Antes precisamos criar um arquivo novo, chamado prisma.ts. que irá prevenir que o auto reload do next abra uma nova conexão com o banco sempre que aconteca esse reload, criamos na pasta lib.

    Através desse arquivo, ele garante que tenhamos apenas um prisma client inicializado sempre.

    Todo `new PrismaClient()` é uma nova conexão com o banco.

    Isso garante que a aplicação funcione de forma correta em desenvolvimento.

    No arquivo prisma.ts exportamos a variável db, que será nosso banco de dados.

    Para receber o banco, alteramos o seguinte;

```js
import { db } from "../_lib/prisma"

export default async function Home() {
  const barbershops = await db.barbershop.findMany({})
}
```

* Importamos o db de _lib/prisma

* Alteramos a função Home() para que seja uma async function, que irá receber o banco na variável barbershops, assim conseguimos trazer todas as informações das barbearias.

    Agora criamos a div que irá conter o componente na page.ts, que terá um h2, com o titulo RECOMENDADOS.

    Adicionamos um padding a esse título;

```js
<h2 className="px-5 text-xs mb-3 uppercase text-gray-400 font-bold">recomendados</h2>
```

    Na div adicionamos um margin top de 6;

```js
      <div className="mt-6">
```

    Dentro dessa div criamos outra div que irá renderizar os itens da barbearia.

```js
<div>
   {barbershops.map((barbershop) => (
      <BarbershopItem key={barbershop.id} barbershop={barbershop} />
   ))}
</div>
```

    Nessa div;

* Aqui recebe os dados das barbearias, e é renderizado no componente barbershopItem, onde temos essa div usando os dados do banco para renderizar o componente;

```js
const BarbershopItem = ({barbershop}: BarbershopItemProps) => {
    return ( 
        <Card>
            <CardContent className="p-0">
                <h1>{barbershop.name}</h1>
            </CardContent>
        </Card>
     );
}
```

* Pegamos os itens das barbearias e percorremos com .map;

* BarbershopItem recebe o id da barbearia;

* Renderiza os nomes das barbearias;

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-10%2020-39-48.png)

## 

## Estilizando o componente

    Iniciamos renderizando a imagem, que deverá ocupar 100% da largura do card, para isso estilizamos da seguinte forma;

```js
 <Image src={barbershop.imageUrl} height={0} width={0} sizes="100vm" className="h-[159px] w-full" alt="Imagem barbearia"
/>
```

* src={barbershop.imageUrl} de onde vem a imagem.

* height={0} width={0} Para ocupar 100% do card;

* sizes="100vm" Também por causa do tamanho da imagem;

* className="h-[159px] w-full" Aqui definimos um heigth fixo de 159px e um width ocupando 100%;

* alt="Imagem barbearia" Texto alternátivo da imagem.

    Ao renderizar tivemos um erro por causa do domínio da imagem, não carregava, foi necessário alterar o arquivo next.config.ts para resolver, adicionamos;

```js
const nextConfig: NextConfig = {
  images: {
    remotePatterns: [{
      hostname: "utfs.io",
    }]
  }
};
```

    Reiniciado o servidor, carregou normalmente.

![](/home/carlos/Imagens/Capturas%20de%20tela/Captura%20de%20tela%20de%202025-11-10%2021-00-30.png)

    Agora iremos ajustar as imagens para ficarem com melhor qualidade.

    No componente card iremos estilizar de forma a dimencionar corretamente a imagem;

```js
<Card className="min-w-[167px] max-w-[167] rounded-2xl">
    <CardContent className="p-1">
        <Image src={barbershop.imageUrl} height={0} width={0} 
                    sizes="100vm" 
                    className="h-[159px] w-full rounded-2xl" 
                    alt={barbershop.name} 
    />
    </CardContent>
</Card>
```

    Assim finalizamos o cardContent que contém a imagem, onde;

    Card principal;

* min-w-[167px] max-w-[167] - Define um width minimo e máximo para a imagem;

* rounded-2xl - Arredondar as bordas;

    CardContent;

```js
 <CardContent className="p-0">
```

* className="p-0" - Apenas um padding-0 aqui tivemos um erro, então colocamos a img dentro de uma div;

    Image;

```js
<div className="px-1">
    <Image 
    src={barbershop.imageUrl}
    height={0} width={0}
    sizes="100vm" 
    className="h-[159px] w-full rounded-2xl" 
    alt={barbershop.name} 
    />
</div>
```

    Optmos por colocar a img dentro de uma div para que ficasse mais organizado a aplicação do padding;

```js
<div className="px-1">
```

A partir daqui é a img;

* src={barbershop.imageUrl} - Imagem é carregada do banco de dados;

* sizes="100vm" - Tamanho da view;

* h-[159px] w-full - Heidth máximo;

* rounded-2xl" - Arredondar bordas

* alt={barbershop.name} - Texto alternátivo, que pode ser usado em leitores de tela.

    Logo abaixo colocamos um h2 com o nome da barbearia, que também será carregado do banco;

```js
 <h2 className="font-bold mt-2 overflow-hidden text-ellipsis text-nowrap">{barbershop.name}</h2>
```

* Com font bold;

* mt-2 - Margin-top de 2, 8px.

* overflow-hidden - Não vai aplicar scroll, ou seja, não vai aumentar o tamanho do componente para colocar o texto

* text-ellipsis - Se não couber o texto, será adicionado o ellipsis, que são os três pontos;

* text-nowrap - Não vai quebrar a linha, mesmo tendo mais de uma linha.

    Abaixo criamos um parágrafo para por o endereço;

```js
<p className="text-sm text-gray-400 overflow-hidden text-ellipsis text-nowrap">{barbershop.address}</p>
```

* Text-sm - Estilo do texto;

* text-gray-400 - fonte;

* overflow-hidden - Não vai aplicar scroll, ou seja, não vai aumentar o tamanho do componente para colocar o texto;

* text-ellipsis - Se não couber o texto, será adicionado o ellipsis, que são os três pontos;

* text-nowrap - Não vai quebrar a linha, mesmo tendo mais de uma linha.

    Voltamos a page.ts para estilizar o card, para ter as imagens uma ao lado da outra e também para configurar o overflow, para ter a rolagem no eixo x das imagens;

```js
<div className="flex gap-4 overflow-x-auto [&::-webkit-scrollbar]:hidden">
        {barbershops.map((barbershop) => (
         <BarbershopItem key={barbershop.id} barbershop={barbershop} />
        ))}
</div>
```

* flex - Para ficar um ao lado do outro;

* gap-4 - Espaçamento;

* overflow-x-auto [&::-webkit-scrollbar]:hidden - Para esconder a barra de rolagem.

    Adicionamos um button, reservar;

```js
<Button variant={"secondary"}>Reservar</Button>
```

* variant={"secondary"} - Variável de cor, cinza.

* className="w-full" - Width full, para ocupar todo o espaço;

* mt-3 - Margin-top de 3, 12px.

    Toda a parte do h2, p e Buttos foram postos em uma div para poder aplicar padding;

```js
<div className="px-3 pb-3">
    <h2 className="font-bold mt-2 overflow-hidden text-ellipsis text-nowrap">{barbershop.name}</h2>
    <p className="text-sm text-gray-400 overflow-hidden text-ellipsis text-nowrap">{barbershop.address}</p>

    <Button className="w-full mt-3" variant={"secondary"}>
    Reservar
    </Button>
</div>    
```

    Aplicado os estilos, e espaçamentos necessários.

    A imagem não estava com uma resolução boa, então alteramos o modo como trabalhamos com ela.

* Na div colocamos uma posicion relative;

* Apagamos o height, width e sizes, e substituimos por fill;

* fill - Propriedade que irá preecher a div;

    Neste ponto a imagem some pois ela precisa ter um tamanho;

* O tamanho será w-full - width full

    Inserimos um espaçamento na div que está em volta dos barbershops itens, pois estava colada na margem, assim foi necessário colocar um padding horizontal nela.

    Então na page.tsx adicionamos um px-5 para ter o espaçamento.



## Avaliações

    A estrelinha precisamos posicinar de forma absolute no componente, para conter esse componente foi necessário criar uma div, para que pudessemos usar o position absolute de uma forma mais segura;

    A div que contém a imagem foi configurada como position relative, para que pudessemos usar position absolute no icone das avaliações, as estrelas.

```js
<div className="w-full h-[159px] relative">
    <div className="absolute top-2 left-2  z-50">
        <Badge variant="secondary" className="opacity-90 flex gap-1 items-center top-3 left-3">
            <StarIcon size={12} className="fill-primary text-primary"/>
            <span className="text-xs ">5,0</span>
        </Badge>
    </div>
        <Image 
            alt={barbershop.name}
            src={barbershop.imageUrl} 
            style={{
                objectFit: "cover"
            }}
            fill 
            className="rounded-2xl" 
    />
</div>
```

    Então, na div que contém as images e a badge, a div "pai" de todas;

* w-full - Largura máxima, todo o componente;

* h-[159px] - Altura de 159px;

* relative - Posição para ser possível usar absolute na imagem do ícone.

    Para a div que contém o componente Badge;

* absolute - Para que consigamos alterar o posicionamento dentro do componente;

* top-2;

* left-2;

* z-50 - Traz para frente do componente pai.

   Para a badge, aqui temos como se fosse um componente;

- variant="secondary"

- opacity-90

- flex

- gap-1

- items-center

- top-3

- left-3

```js
<Badge variant="secondary" className="opacity-90 flex gap-1 items-center top-3 left-3">
    <StarIcon size={12} className="fill-primary text-primary"/>
    <span className="text-xs ">5,0</span>
</Badge>
```

    Star Icon;

* size={12}

* fill-primary - usado para preecher o svg, a imagem do icone da estrela;

* text-primary - usado no traçado do svg

* Posicion absolute;

* Top zero - top-0;

* left-0 - zero esquerda;

* z-50 - Para ela ficar acima da imagem

* Dentro da badge colocamos um starIcon com size de 12px;

* Dentro da badge colocamos um spam para o texto, a nota no caso;

    Texto dentro do componente, no caso `5,0`

* text-xs.



Commit

`feat: add initial barbershop item`



## Populares

    Para criar esse componente apenas copiamos e colamos o componente criado anteriormente, a alteramos no título;

```js
<div className="mt-6">
    <h2 className="px-5 text-xs mb-3 uppercase text-gray-400 font-bold">Populares</h2>

    <div className="flex px-5 gap-4 overflow-x-auto [&::-webkit-scrollbar]:hidden">
        {barbershops.map((barbershop) => (
        <BarbershopItem key={barbershop.id} barbershop={barbershop} />
        ))}
    </div>
</div>
```

    Isso será alterado para pegar-mos apenas barbearias aleatórias.



## Footer

    Como o footer usaremos em todas as páginas criaremos na pasta components;

* w-full

* bg-secondary - um cinza mais escuro;

* py-6;

* px-5.

    Para o texto dentro do footer;

* text-gray-400 - cor

* text-xs - Tamanho do texto

* font-bold - Negrito

* opacity-75 - Opacidade

    Como trata-se de um componente que usaremos em todas as páginas sem exceção, colocaremos no layout do next, e não na page.tsx;

    Então colocamos na página layout;

```js
{
  return (
    <html lang="en">
      <body className={`${inter.className} dark`}>
        {children}
        < Footer />
      </body>
    </html>
  );
}
```

Colocamos uma margem no elemento anterior, na page.tsx, que no caso é o último card, Populares.

```js
<div className="mt-6 mb-[4.5rem]">
    <h2 className="px-5 text-xs mb-3 uppercase text-gray-400 font-bold">Populares</h2>

    <div className="flex px-5 gap-4 overflow-x-auto [&::-webkit-scrollbar]:hidden">
    {barbershops.map((barbershop) => (
    <BarbershopItem key={barbershop.id} barbershop={barbershop} />
    ))}
    </div>
</div>
```

* mb-[4.5rem];

Commit

`feat: add footer component`
