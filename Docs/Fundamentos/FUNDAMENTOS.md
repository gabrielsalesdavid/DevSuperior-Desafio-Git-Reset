# Fundamentos das Linguagens do Repositório

Este documento aborda os fundamentos essenciais das linguagens e frameworks utilizados neste repositório.

---

## 1. JavaScript/TypeScript

### O que é?
JavaScript é uma linguagem de programação interpretada, inicialmente criada para navegadores. TypeScript é um superset do JavaScript que adiciona tipagem estática.

### Conceitos Fundamentais

#### Variáveis e Tipos
```javascript
// JavaScript
let nome = "Gabriel"; // Pode ser reatribuído
const idade = 30;     // Não pode ser reatribuído
var x = 10;           // Escopo de função (obsoleto)

// TypeScript
let nome: string = "Gabriel";
let idade: number = 30;
let ativo: boolean = true;
```

#### Funções
```javascript
// Função tradicional
function saudar(nome) {
  return `Olá, ${nome}!`;
}

// Arrow function
const saudar = (nome) => `Olá, ${nome}!`;

// TypeScript com tipos
function saudar(nome: string): string {
  return `Olá, ${nome}!`;
}
```

#### Objetos e Arrays
```javascript
// Objeto
const usuario = {
  nome: "Gabriel",
  idade: 30,
  email: "gabriel@email.com"
};

// Array
const numeros = [1, 2, 3, 4, 5];
const usuarios = [
  { nome: "Ana", idade: 25 },
  { nome: "Bruno", idade: 28 }
];

// Destructuring
const { nome, idade } = usuario;
const [primeiro, segundo] = numeros;
```

#### Operações Assíncronas
```javascript
// Callbacks
function buscarDados(callback) {
  setTimeout(() => {
    callback("Dados carregados");
  }, 1000);
}

// Promises
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Sucesso!");
  }, 1000);
});

// Async/Await
async function buscarDados() {
  try {
    const resposta = await fetch('https://api.exemplo.com/dados');
    const dados = await resposta.json();
    return dados;
  } catch (erro) {
    console.error('Erro:', erro);
  }
}
```

---

## 2. Angular (TypeScript)

### O que é?
Angular é um framework TypeScript para construir aplicações web de página única (SPA). Mantido pelo Google, oferece uma estrutura completa e opinativa.

### Conceitos Fundamentais

#### Componentes
Um componente é a unidade básica do Angular.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>Olá {{ nome }}</h1>`,
  styles: [`h1 { color: blue; }`]
})
export class HelloComponent {
  nome: string = "Mundo";
}
```

#### Módulos
Módulos organizam componentes, serviços e outras funcionalidades.

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HelloComponent } from './hello.component';

@NgModule({
  declarations: [HelloComponent],
  imports: [BrowserModule],
  providers: [],
  bootstrap: [HelloComponent]
})
export class AppModule { }
```

#### Data Binding
```typescript
// Interpolação
<h1>{{ titulo }}</h1>

// Property binding
<img [src]="urlImagem">

// Event binding
<button (click)="fazerAlgo()">Clique</button>

// Two-way binding
<input [(ngModel)]="nome">
```

#### Serviços e Injeção de Dependência
```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class UsuarioService {
  constructor(private http: HttpClient) { }
  
  obterUsuarios() {
    return this.http.get('/api/usuarios');
  }
}
```

---

## 3. React (JavaScript/TypeScript)

### O que é?
React é uma biblioteca JavaScript para construir interfaces de usuário com componentes reutilizáveis. Mantida pelo Facebook, é baseada em estado e renderização declarativa.

### Conceitos Fundamentais

#### Componentes Funcionais
```javascript
// Componente básico
function Saudacao() {
  return <h1>Olá, Mundo!</h1>;
}

// Com props
function Usuario({ nome, idade }) {
  return <div>Nome: {nome}, Idade: {idade}</div>;
}
```

#### Hooks
```javascript
import { useState, useEffect } from 'react';

function Contador() {
  const [contador, setContador] = useState(0);
  
  useEffect(() => {
    console.log('Componente montado');
  }, []);
  
  return (
    <div>
      <p>Contador: {contador}</p>
      <button onClick={() => setContador(contador + 1)}>Incrementar</button>
    </div>
  );
}
```

#### JSX
JSX é uma sintaxe que parece HTML mas é JavaScript.

```javascript
const usuario = {
  nome: "Gabriel",
  idade: 30
};

const elemento = (
  <div>
    <h1>Olá, {usuario.nome}!</h1>
    <p>Idade: {usuario.idade}</p>
  </div>
);
```

#### Props e State
```javascript
// Props: dados passados de pai para filho
function Produto({ nome, preco }) {
  return <div>{nome} - R$ {preco}</div>;
}

// State: dados internos do componente
function Carrinho() {
  const [itens, setItens] = useState([]);
  
  const adicionarItem = (item) => {
    setItens([...itens, item]);
  };
  
  return <div>Itens: {itens.length}</div>;
}
```

---

## 4. Vue (JavaScript)

### O que é?
Vue é um framework JavaScript progressivo para construir interfaces de usuário. É conhecido pela sua curva de aprendizado suave e reatividade elegante.

### Conceitos Fundamentais

#### Componentes Vue 3
```javascript
<template>
  <div>
    <h1>{{ mensagem }}</h1>
    <button @click="incrementar">Contador: {{ contador }}</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const mensagem = ref('Olá, Vue!');
const contador = ref(0);

const incrementar = () => {
  contador.value++;
};
</script>

<style scoped>
h1 {
  color: green;
}
</style>
```

#### Data Binding
```javascript
// Interpolação
{{ mensagem }}

// Binding de atributos
:src="urlImagem"

// Binding de eventos
@click="fazerAlgo"

// Two-way binding
v-model="nome"
```

#### Computed e Watch
```javascript
import { ref, computed, watch } from 'vue';

const nome = ref('Gabriel');
const idade = ref(30);

// Computed: propriedade derivada
const apresentacao = computed(() => {
  return `${nome.value} tem ${idade.value} anos`;
});

// Watch: observar mudanças
watch(nome, (novoValor, valorAntigo) => {
  console.log(`Nome mudou de ${valorAntigo} para ${novoValor}`);
});
```

#### Diretivas
```javascript
// v-if: renderização condicional
<div v-if="usuario">
  <p>{{ usuario.nome }}</p>
</div>

// v-for: renderização de listas
<ul>
  <li v-for="item in itens" :key="item.id">
    {{ item.nome }}
  </li>
</ul>

// v-show: exibição condicional com CSS
<p v-show="mostrar">Texto visível</p>
```

---

## 5. Java Spring

### O que é?
Spring é um framework Java para criar aplicações robustas e escaláveis. Spring Boot simplifica a configuração e permite criar aplicações rapidamente.

### Conceitos Fundamentais

#### Estrutura Básica de um Projeto Spring
```
projeto-spring/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/exemplo/
│   │   │       ├── Application.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       └── model/
│   │   └── resources/
│   │       └── application.properties
│   └── test/
└── pom.xml
```

#### Anotações Básicas
```java
// Classe principal
@SpringBootApplication
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}

// Controlador
@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {
  
  @GetMapping
  public List<Usuario> listar() {
    return usuarioService.listar();
  }
  
  @PostMapping
  public Usuario criar(@RequestBody Usuario usuario) {
    return usuarioService.criar(usuario);
  }
}
```

#### Serviços e Injeção de Dependência
```java
@Service
public class UsuarioService {
  
  @Autowired
  private UsuarioRepository repository;
  
  public List<Usuario> listar() {
    return repository.findAll();
  }
  
  public Usuario criar(Usuario usuario) {
    return repository.save(usuario);
  }
}
```

#### Entidades e Repositórios
```java
@Entity
@Table(name = "usuarios")
public class Usuario {
  
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  @Column(nullable = false)
  private String nome;
  
  private String email;
  
  // Getters e Setters
}

@Repository
public interface UsuarioRepository extends JpaRepository<Usuario, Long> {
  Usuario findByEmail(String email);
  List<Usuario> findByNomeContaining(String nome);
}
```

#### Requisições HTTP
```java
// GET
@GetMapping("/{id}")
public Usuario obter(@PathVariable Long id) {
  return usuarioService.obter(id);
}

// POST
@PostMapping
public ResponseEntity<Usuario> criar(@RequestBody Usuario usuario) {
  Usuario criado = usuarioService.criar(usuario);
  return ResponseEntity.ok(criado);
}

// PUT
@PutMapping("/{id}")
public Usuario atualizar(@PathVariable Long id, @RequestBody Usuario usuario) {
  return usuarioService.atualizar(id, usuario);
}

// DELETE
@DeleteMapping("/{id}")
public void deletar(@PathVariable Long id) {
  usuarioService.deletar(id);
}
```

---

## Resumo Comparativo

| Aspecto | JavaScript | TypeScript | Angular | React | Vue | Java Spring |
|---------|-----------|-----------|---------|-------|-----|-------------|
| **Tipo** | Linguagem interpretada | Superset TypeScript | Framework web | Biblioteca web | Framework web | Framework backend |
| **Tipagem** | Dinâmica | Estática | Estática | Dinâmica | Dinâmica | Estática |
| **Curva Aprendizado** | Média | Média-Alta | Alta | Média | Baixa | Média-Alta |
| **Comunidade** | Muito Grande | Grande | Grande | Muito Grande | Grande | Muito Grande |
| **Performance** | Boa | - | Ótima | Ótima | Ótima | Excelente |

---

## Recursos de Aprendizado Recomendados

- **JavaScript/TypeScript**: MDN Web Docs, TypeScript Official Handbook
- **Angular**: Angular Official Documentation, Angular University
- **React**: React Official Documentation, Scrimba React Course
- **Vue**: Vue Official Documentation, Vue School
- **Java Spring**: Spring Official Documentation, Spring in Action book

