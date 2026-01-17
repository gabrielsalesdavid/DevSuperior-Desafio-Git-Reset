# Conceitos Avançados das Linguagens do Repositório

Este documento aborda conceitos mais avançados e padrões de desenvolvimento para as linguagens e frameworks utilizados.

---

## 1. JavaScript/TypeScript - Conceitos Avançados

### Closures e Escopo Léxico
Closures permitem que funções acessem variáveis do escopo externo mesmo após a função ter retornado.

```typescript
function criar_contador() {
  let contador = 0;
  
  return {
    incrementar: () => ++contador,
    decrementar: () => --contador,
    obter: () => contador
  };
}

const contador = criar_contador();
console.log(contador.incrementar()); // 1
console.log(contador.incrementar()); // 2
console.log(contador.obter());       // 2
```

### Generics (TypeScript)
Permitem criar código reutilizável que funciona com múltiplos tipos.

```typescript
// Função genérica
function identidade<T>(arg: T): T {
  return arg;
}

// Classe genérica
class Repositorio<T> {
  private itens: T[] = [];
  
  adicionar(item: T): void {
    this.itens.push(item);
  }
  
  obter(indice: number): T {
    return this.itens[indice];
  }
  
  listar(): T[] {
    return this.itens;
  }
}

interface Usuario {
  id: number;
  nome: string;
}

const repoUsuarios = new Repositorio<Usuario>();
repoUsuarios.adicionar({ id: 1, nome: "Gabriel" });
```

### Decorators (TypeScript)
Modificam o comportamento de classes, métodos ou propriedades.

```typescript
// Decorator de classe
function Logger(constructor: Function) {
  console.log(`Criando instância de ${constructor.name}`);
}

// Decorator de método
function ValidarEmail(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const metodoOriginal = descriptor.value;
  
  descriptor.value = function(email: string) {
    if (!email.includes('@')) {
      throw new Error('Email inválido');
    }
    return metodoOriginal.call(this, email);
  };
  
  return descriptor;
}

@Logger
class Usuario {
  @ValidarEmail
  atribuirEmail(email: string) {
    this.email = email;
  }
}
```

### Padrões de Módulo e Encapsulamento
```typescript
// Private, Protected, Public
class ContaBancaria {
  public titular: string;
  protected saldo: number = 0;
  private senha: string;
  
  depositar(valor: number): void {
    this.saldo += valor;
  }
  
  private validarSenha(senha: string): boolean {
    return senha === this.senha;
  }
}

// Encapsulamento com getters e setters
class Temperatura {
  private _celsius: number = 0;
  
  get celsius(): number {
    return this._celsius;
  }
  
  set celsius(valor: number) {
    if (valor < -273.15) {
      throw new Error('Temperatura abaixo do zero absoluto');
    }
    this._celsius = valor;
  }
  
  get fahrenheit(): number {
    return (this._celsius * 9/5) + 32;
  }
}
```

### Type Guards e Narrowing
```typescript
type Animal = 'cachorro' | 'gato' | 'passaro';
type Veiculo = 'carro' | 'bicicleta';
type Objeto = Animal | Veiculo;

function processar(obj: Objeto): string {
  if (obj === 'cachorro' || obj === 'gato' || obj === 'passaro') {
    return `Animal encontrado: ${obj}`;
  } else {
    return `Veículo encontrado: ${obj}`;
  }
}

// Type predicate
function eh_string(valor: unknown): valor is string {
  return typeof valor === 'string';
}

function processar_valor(valor: unknown): void {
  if (eh_string(valor)) {
    console.log(valor.toUpperCase());
  }
}
```

---

## 2. Angular - Conceitos Avançados

### RxJS e Observables
Angular utiliza RxJS para gerenciar streams de dados assíncronos.

```typescript
import { Observable, Subject, BehaviorSubject } from 'rxjs';
import { map, filter, debounceTime, distinctUntilChanged } from 'rxjs/operators';

@Injectable()
export class UsuarioService {
  private usuarios$ = new BehaviorSubject<Usuario[]>([]);
  private busca$ = new Subject<string>();
  
  constructor(private http: HttpClient) {
    this.busca$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(termo => termo.length > 0),
      map(termo => this.http.get<Usuario[]>(`/api/usuarios?q=${termo}`))
    ).subscribe(resultado => this.usuarios$.next(resultado));
  }
  
  buscar(termo: string): void {
    this.busca$.next(termo);
  }
  
  obterUsuarios(): Observable<Usuario[]> {
    return this.usuarios$.asObservable();
  }
}
```

### Change Detection Strategy
Angular oferece estratégias otimizadas de detecção de mudanças.

```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

@Component({
  selector: 'app-usuario-card',
  template: `
    <div class="card">
      <h3>{{ usuario.nome }}</h3>
      <p>{{ usuario.email }}</p>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UsuarioCardComponent {
  @Input() usuario: Usuario;
}
```

### Interceptadores HTTP
Interceptam requisições e respostas HTTP globalmente.

```typescript
import { Injectable } from '@angular/core';
import {
  HttpInterceptor,
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpErrorResponse
} from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErroInterceptador implements HttpInterceptor {
  constructor(private roteador: Router) { }
  
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((erro: HttpErrorResponse) => {
        if (erro.status === 401) {
          this.roteador.navigate(['/login']);
        }
        return throwError(() => erro);
      })
    );
  }
}

// Registrar no módulo
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: ErroInterceptador,
    multi: true
  }
]
```

### Lazy Loading
Carrega módulos sob demanda para melhorar performance inicial.

```typescript
const rotas: Routes = [
  {
    path: 'usuarios',
    loadChildren: () => import('./usuarios/usuarios.module')
      .then(m => m.UsuariosModule)
  },
  {
    path: 'produtos',
    loadChildren: () => import('./produtos/produtos.module')
      .then(m => m.ProdutosModule)
  }
];
```

### State Management com NgRx
Gerencia estado da aplicação de forma previsível.

```typescript
// Action
export const carregarUsuarios = createAction(
  '[Usuário] Carregar Usuários'
);

export const usuariosCarregados = createAction(
  '[Usuário] Usuários Carregados',
  props<{ usuarios: Usuario[] }>()
);

// Reducer
const usuarioReducer = createReducer(
  estadoInicial,
  on(carregarUsuarios, (state) => ({ ...state, carregando: true })),
  on(usuariosCarregados, (state, { usuarios }) => ({
    ...state,
    usuarios,
    carregando: false
  }))
);

// Effect
@Injectable()
export class UsuarioEffect {
  carregarUsuarios$ = createEffect(() =>
    this.actions$.pipe(
      ofType(carregarUsuarios),
      mergeMap(() =>
        this.usuarioService.obter().pipe(
          map(usuarios => usuariosCarregados({ usuarios }))
        )
      )
    )
  );
}
```

---

## 3. React - Conceitos Avançados

### Context API e Padrão Provider
Evita prop drilling ao compartilhar estado globalmente.

```javascript
import React, { createContext, useContext, useState } from 'react';

const UsuarioContext = createContext();

export function UsuarioProvider({ children }) {
  const [usuario, setUsuario] = useState(null);
  const [carregando, setCarregando] = useState(false);
  
  const login = async (email, senha) => {
    setCarregando(true);
    const resposta = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, senha })
    });
    const dados = await resposta.json();
    setUsuario(dados);
    setCarregando(false);
  };
  
  return (
    <UsuarioContext.Provider value={{ usuario, carregando, login }}>
      {children}
    </UsuarioContext.Provider>
  );
}

export function useUsuario() {
  const contexto = useContext(UsuarioContext);
  if (!contexto) {
    throw new Error('useUsuario deve ser usado dentro de UsuarioProvider');
  }
  return contexto;
}
```

### Custom Hooks
Extraem lógica reutilizável em hooks personalizados.

```javascript
function useFetch(url) {
  const [dados, setDados] = useState(null);
  const [carregando, setCarregando] = useState(true);
  const [erro, setErro] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(dados => {
        setDados(dados);
        setCarregando(false);
      })
      .catch(err => {
        setErro(err);
        setCarregando(false);
      });
  }, [url]);
  
  return { dados, carregando, erro };
}

// Uso
function ListaUsuarios() {
  const { dados: usuarios, carregando } = useFetch('/api/usuarios');
  
  if (carregando) return <p>Carregando...</p>;
  
  return (
    <ul>
      {usuarios.map(u => <li key={u.id}>{u.nome}</li>)}
    </ul>
  );
}
```

### Memoização e Otimização
Evita re-renderizações desnecessárias.

```javascript
import React, { memo, useMemo, useCallback } from 'react';

// Memoizar componente
const UsuarioCard = memo(({ usuario, aoClicarem }) => {
  return (
    <div onClick={aoClicarem}>
      {usuario.nome}
    </div>
  );
});

function Lista({ usuarios }) {
  // useMemo: memoizar valor
  const usuariosAtivos = useMemo(() => {
    return usuarios.filter(u => u.ativo);
  }, [usuarios]);
  
  // useCallback: memoizar função
  const aoClicarem = useCallback((usuarioId) => {
    console.log(`Clicou em ${usuarioId}`);
  }, []);
  
  return (
    <div>
      {usuariosAtivos.map(u => (
        <UsuarioCard 
          key={u.id} 
          usuario={u} 
          aoClicarem={() => aoClicarem(u.id)}
        />
      ))}
    </div>
  );
}
```

### Error Boundaries
Captura erros em árvore de componentes.

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { temErro: false };
  }
  
  static getDerivedStateFromError(error) {
    return { temErro: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Erro capturado:', error, errorInfo);
  }
  
  render() {
    if (this.state.temErro) {
      return <h1>Algo deu errado!</h1>;
    }
    
    return this.props.children;
  }
}

// Uso
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

### Gerenciamento de Estado Complexo
Utilizando useReducer para lógica complexa.

```javascript
const estadoInicial = {
  usuarios: [],
  carregando: false,
  erro: null
};

function reducer(estado, acao) {
  switch (acao.tipo) {
    case 'CARREGAR_INICIO':
      return { ...estado, carregando: true };
    case 'CARREGAR_SUCESSO':
      return {
        ...estado,
        usuarios: acao.payload,
        carregando: false
      };
    case 'CARREGAR_ERRO':
      return {
        ...estado,
        erro: acao.payload,
        carregando: false
      };
    default:
      return estado;
  }
}

function UsuariosList() {
  const [estado, dispatch] = useReducer(reducer, estadoInicial);
  
  useEffect(() => {
    dispatch({ tipo: 'CARREGAR_INICIO' });
    fetch('/api/usuarios')
      .then(res => res.json())
      .then(dados => dispatch({
        tipo: 'CARREGAR_SUCESSO',
        payload: dados
      }))
      .catch(err => dispatch({
        tipo: 'CARREGAR_ERRO',
        payload: err.message
      }));
  }, []);
  
  return (
    <div>
      {estado.carregando && <p>Carregando...</p>}
      {estado.erro && <p>Erro: {estado.erro}</p>}
      {estado.usuarios.map(u => <div key={u.id}>{u.nome}</div>)}
    </div>
  );
}
```

---

## 4. Vue - Conceitos Avançados

### Composables (Vue 3)
Reutilizar lógica entre componentes.

```javascript
import { ref, computed, watch } from 'vue';

export function useTarefa() {
  const tarefas = ref([]);
  const novoTitulo = ref('');
  
  const tarefasCompletas = computed(() => {
    return tarefas.value.filter(t => t.completa);
  });
  
  const adicionarTarefa = () => {
    tarefas.value.push({
      id: Date.now(),
      titulo: novoTitulo.value,
      completa: false
    });
    novoTitulo.value = '';
  };
  
  const removerTarefa = (id) => {
    tarefas.value = tarefas.value.filter(t => t.id !== id);
  };
  
  watch(tarefas, (novasTarefas) => {
    localStorage.setItem('tarefas', JSON.stringify(novasTarefas));
  }, { deep: true });
  
  return {
    tarefas,
    novoTitulo,
    tarefasCompletas,
    adicionarTarefa,
    removerTarefa
  };
}
```

### Reactive vs Ref
Diferenças entre os dois padrões de reatividade.

```javascript
import { ref, reactive } from 'vue';

// ref: para valores primitivos
const contador = ref(0);
contador.value++; // Acessa com .value em JS

// reactive: para objetos
const usuario = reactive({
  nome: 'Gabriel',
  idade: 30,
  emails: ['gabriel@email.com']
});
usuario.nome = 'Carlos'; // Acesso direto

// Desestruturação com ref
import { toRefs } from 'vue';

const usuario = reactive({ nome: 'Gabriel', idade: 30 });
const { nome, idade } = toRefs(usuario);
// Agora nome e idade são refs
```

### Teleport e Suspense
Casos de uso avançados.

```javascript
// Teleport: renderizar componente em outro lugar do DOM
<template>
  <div>
    <button @click="mostrarModal = true">Abrir Modal</button>
    
    <Teleport to="body" v-if="mostrarModal">
      <div class="modal">
        <h2>Modal</h2>
        <button @click="mostrarModal = false">Fechar</button>
      </div>
    </Teleport>
  </div>
</template>

// Suspense: aguardar dados assíncronos
<template>
  <Suspense>
    <template #default>
      <ListaUsuarios />
    </template>
    <template #fallback>
      <p>Carregando usuários...</p>
    </template>
  </Suspense>
</template>
```

### Store (Pinia)
Gerenciamento de estado centralizado.

```javascript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useUsuarioStore = defineStore('usuario', () => {
  const usuarios = ref([]);
  const filtro = ref('');
  
  const usuariosFiltrados = computed(() => {
    return usuarios.value.filter(u =>
      u.nome.toLowerCase().includes(filtro.value.toLowerCase())
    );
  });
  
  async function carregar() {
    const resposta = await fetch('/api/usuarios');
    usuarios.value = await resposta.json();
  }
  
  function adicionar(usuario) {
    usuarios.value.push(usuario);
  }
  
  return {
    usuarios,
    filtro,
    usuariosFiltrados,
    carregar,
    adicionar
  };
});

// Uso em componente
import { useUsuarioStore } from '@/stores/usuario';

export default {
  setup() {
    const store = useUsuarioStore();
    return { store };
  }
}
```

---

## 5. Java Spring - Conceitos Avançados

### AOP (Aspect-Oriented Programming)
Separar preocupações transversais como logging e segurança.

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {
  
  private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);
  
  @Before("execution(* com.exemplo.service.*.*(..))")
  public void logBefore() {
    logger.info("Executando método de serviço");
  }
  
  @After("execution(* com.exemplo.service.*.*(..))")
  public void logAfter() {
    logger.info("Método executado");
  }
}
```

### Segurança com Spring Security
Autenticação e autorização.

```java
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@EnableWebSecurity
public class ConfigSeguranca {
  
  @Bean
  public SecurityFilterChain filtroSeguranca(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
        .antMatchers("/", "/home").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        .and()
      .formLogin()
        .loginPage("/login")
        .permitAll()
        .and()
      .logout()
        .permitAll();
    
    return http.build();
  }
  
  @Bean
  public BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }
}
```

### Validação com Bean Validation
Validar dados de entrada.

```java
import javax.validation.constraints.*;

public class Usuario {
  
  @NotBlank(message = "Nome é obrigatório")
  @Size(min = 3, max = 100)
  private String nome;
  
  @Email(message = "Email inválido")
  private String email;
  
  @Min(value = 18, message = "Deve ter pelo menos 18 anos")
  private Integer idade;
  
  @Pattern(regexp = "^[0-9]{3}\\.[0-9]{3}\\.[0-9]{3}-[0-9]{2}$")
  private String cpf;
}

@RestController
@RequestMapping("/usuarios")
public class UsuarioController {
  
  @PostMapping
  public ResponseEntity<Usuario> criar(@Valid @RequestBody Usuario usuario) {
    return ResponseEntity.ok(usuarioService.criar(usuario));
  }
}
```

### JPA e Relacionamentos
Mapeamento Objeto-Relacional avançado.

```java
@Entity
@Table(name = "usuarios")
public class Usuario {
  
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  private String nome;
  
  // Um para muitos
  @OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL)
  private List<Telefone> telefones;
  
  // Muitos para um
  @ManyToOne
  @JoinColumn(name = "id_endereco")
  private Endereco endereco;
}

@Entity
@Table(name = "telefones")
public class Telefone {
  
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  private String numero;
  
  @ManyToOne
  @JoinColumn(name = "id_usuario")
  private Usuario usuario;
}
```

### Transações e Propagação
Controle de transações em múltiplos métodos.

```java
@Service
public class UsuarioService {
  
  @Autowired
  private UsuarioRepository usuarioRepository;
  
  @Autowired
  private HistoricoRepository historicoRepository;
  
  @Transactional
  public void criar(Usuario usuario) {
    usuarioRepository.save(usuario);
    
    Historico historico = new Historico();
    historico.setAcao("USUARIO_CRIADO");
    historico.setData(LocalDateTime.now());
    historicoRepository.save(historico);
  }
  
  @Transactional(readOnly = true)
  public List<Usuario> listar() {
    return usuarioRepository.findAll();
  }
}
```

### Caching
Melhorar performance com cache.

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.CacheEvict;

@Service
@EnableCaching
public class UsuarioService {
  
  @Cacheable(value = "usuarios", key = "#id")
  public Usuario obter(Long id) {
    return usuarioRepository.findById(id).orElse(null);
  }
  
  @CacheEvict(value = "usuarios", key = "#usuario.id")
  public Usuario atualizar(Usuario usuario) {
    return usuarioRepository.save(usuario);
  }
  
  @CacheEvict(value = "usuarios", allEntries = true)
  public void limparCache() {
  }
}
```

---

## Padrões de Projeto Comuns

### Factory Pattern
```typescript
// TypeScript
abstract class Veiculo {
  abstract dirigir(): void;
}

class Carro extends Veiculo {
  dirigir() {
    console.log("Carro dirigindo com 4 rodas");
  }
}

class Moto extends Veiculo {
  dirigir() {
    console.log("Moto dirigindo com 2 rodas");
  }
}

class FabricaVeiculo {
  criar(tipo: string): Veiculo {
    switch(tipo) {
      case 'carro': return new Carro();
      case 'moto': return new Moto();
      default: throw new Error('Tipo desconhecido');
    }
  }
}
```

### Observer Pattern
```javascript
// JavaScript
class EventEmitter {
  constructor() {
    this.eventos = {};
  }
  
  on(evento, callback) {
    if (!this.eventos[evento]) {
      this.eventos[evento] = [];
    }
    this.eventos[evento].push(callback);
  }
  
  emit(evento, dados) {
    if (this.eventos[evento]) {
      this.eventos[evento].forEach(callback => callback(dados));
    }
  }
}

const emitter = new EventEmitter();
emitter.on('usuario-criado', (usuario) => {
  console.log('Novo usuário:', usuario);
});

emitter.emit('usuario-criado', { id: 1, nome: 'Gabriel' });
```

### Singleton Pattern
```java
// Java Spring
@Component
public class ConfiguradorGlobal {
  private static ConfiguradorGlobal instancia;
  
  private ConfiguradorGlobal() {
  }
  
  public static synchronized ConfiguradorGlobal obter() {
    if (instancia == null) {
      instancia = new ConfiguradorGlobal();
    }
    return instancia;
  }
}

// Spring gerencia automaticamente como Singleton
```

---

## Recursos Avançados

- **TypeScript**: Advanced Types, Decorators
- **Angular**: RxJS Documentation, Angular Performance Guide
- **React**: React Suspense, React 18 Concurrency
- **Vue**: Vue 3 Composition API, Pinia Documentation
- **Java Spring**: Spring Boot Documentation, Spring in Action (6th Edition)

---

## Conclusão

O domínio desses conceitos avançados permite criar aplicações mais robustas, eficientes e mantíveis. Cada tecnologia tem seus próprios padrões e melhores práticas que devem ser estudadas e aplicadas conforme a situação.

