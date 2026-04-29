

# Documentação da API Somar

Visão geral dos endpoints, modelos de requisição, respostas e enumerações disponíveis na API.

---

## 1. Visão Geral da API

| Informação | Detalhe |
|------------|---------|
| **Base URL** | `http://localhost:8080` |
| **Formato** | JSON |
| **Autenticação** | Basic (email + senha em texto plano) |

---

## 2. Enumerações

### 2.1 UsuarioEnum

Tipo de usuário do sistema.

| Valor | Descrição |
|-------|-----------|
| `ONG` | Usuário representante de organização |
| `DOADOR` | Usuário que realiza doações |
| `ADMIN` | Administrador do sistema |

```java
public enum UsuarioEnum {
    ONG,
    DOADOR,
    ADMIN
}
```

### 2.2 CampanhaEnum

Status da campanha de arrecadação.

| Valor | Descrição |
|-------|-----------|
| `ATIVA` | Campanha em andamento |
| `FINALIZADA` | Campanha encerrada |
| `CANCELADA` | Campanha cancelada |

```java
public enum CampanhaEnum {
    ATIVA,
    FINALIZADA,
    CANCELADA
}
```

---

## 3. Modelos (Entidades)

### 3.1 Usuario

```java
@Entity
@Table(name = "usuario")
public class Usuario {
    long id;
    String nome;
    String email;
    String senha;
    UsuarioEnum tipo;
    Ong ong;  // Relacionamento ManyToOne
}
```

### 3.2 Ong

```java
@Entity
@Table(name = "ong")
public class Ong {
    long id;
    String nome;
    String descricao;
    String documento;
    String telefone;
    String cidade;
    String estado;
    Usuario usuario;  // OneToOne
    List<Campanha> campanhas;  // OneToMany
    LocalDateTime datacriacao;
}
```

### 3.3 Campanha

```java
@Entity
@Table(name = "campanha")
public class Campanha {
    Long id;
    Ong ong;  // ManyToOne (obrigatório)
    String titulo;
    String descricao;
    float meta;
    float valoratual;
    CampanhaEnum status;
    LocalDateTime datacriacao;
    LocalDateTime diafinalizado;
}
```

---

## 4. Endpoints

### 4.1 UsuarioController

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/usuario/cadastrar` | Criar novo usuário |
| `GET` | `/usuario/listar-usuarios` | Listar todos os usuários |
| `DELETE` | `/usuario/{id}` | Deletar usuário por ID |
| `POST` | `/usuario/login` | Autenticar usuário |

#### POST `/usuario/cadastrar`

**Request:**

```json
{
  "nome": "João Silva",
  "email": "joao@email.com",
  "senha": "123456",
  "tipo": "DOADOR"
}
```

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| `nome` | String | ✅ | Mín: 3, Máx: 100 caracteres |
| `email` | String | ✅ | Formato de email |
| `senha` | String | ✅ | - |
| `tipo` | UsuarioEnum | ✅ | ONG, DOADOR ou ADMIN |

**Response:** Retorna o objeto `Usuario` completo com ID gerado.

---

#### GET `/usuario/listar-usuarios`

**Response:**

```json
[
  {
    "id": 1,
    "nome": "João Silva",
    "email": "joao@email.com",
    "senha": "123456",
    "tipo": "DOADOR"
  }
]
```

---

#### DELETE `/usuario/{id}`

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `id` | Long | ID do usuário a ser deletado |

**Parâmetro via query:** `?id=1`

---

#### POST `/usuario/login`

**Request:**

```json
{
  "email": "joao@email.com",
  "senha": "123456"
}
```

**Response:** `"Usuário Logado"` ou erro 400 se credenciais inválidas.

---

### 4.2 OngController

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/ong/cadastrar` | Criar nova ONG |
| `GET` | `/ong/listar-ongs` | Listar todas as ONGs |
| `GET` | `/ong/buscar-ong-{id}` | Buscar ONG por ID |
| `DELETE` | `/ong/{id}` | Deletar ONG por ID |

#### POST `/ong/cadastrar`

**Request:**

```json
{
  "nome": "ONG Esperança",
  "descricao": "ONG que ajuda crianças",
  "documento": "12.345.678/0001-90",
  "telefone": "(11) 99999-9999",
  "cidade": "São Paulo",
  "estado": "SP",
  "codusuario": 1,
  "dataCriacao": "2024-01-01T10:00:00"
}
```

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| `nome` | String | ✅ |
| `descricao` | String | ✅ |
| `documento` | String | ✅ |
| `telefone` | String | ✅ |
| `cidade` | String | ✅ |
| `estado` | String | ✅ |
| `codusuario` | Long | ✅ |

**Response:** Retorna o objeto `Ong` completo.

---

#### GET `/ong/listar-ongs`

**Response:**

```json
[
  {
    "id": 1,
    "nome": "ONG Esperança",
    "descricao": "ONG que ajuda crianças",
    "documento": "12.345.678/0001-90",
    "telefone": "(11) 99999-9999",
    "cidade": "São Paulo",
    "estado": "SP",
    "usuario": { ... },
    "campanhas": [ ... ],
    "datacriacao": "2024-01-01T10:00:00"
  }
]
```

---

#### GET `/ong/buscar-ong-{id}`

| Parâmetro | Tipo |
|-----------|------|
| `id` | Long |

**Parâmetro via query:** `?id=1`

---

#### DELETE `/ong/{id}`

| Parâmetro | Tipo |
|-----------|------|
| `id` | Long |

**Parâmetro via query:** `?id=1`

---

### 4.3 CampanhaController

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/campanha/cadastrar` | Criar nova campanha |
| `GET` | `/campanha/listar-campanhas` | Listar todas as campanhas |
| `DELETE` | `/campanha/{id}` | Deletar campanha por ID |

#### POST `/campanha/cadastrar`

**Request:**

```json
{
  "codong": 1,
  "titulo": "Campanha de Natal",
  "descricao": "Arrecadação de presentes",
  "meta": 10000.00,
  "valoratual": 0.00,
  "diafinalizado": "2024-12-25T00:00:00",
  "status": "ATIVA"
}
```

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| `codong` | Long | ✅ |
| `titulo` | String | ✅ |
| `descricao` | String | ✅ |
| `meta` | Float | ✅ |
| `valoratual` | Float | ✅ |
| `diafinalizado` | LocalDateTime | ✅ |
| `status` | CampanhaEnum | ✅ |

**Response:** Retorna o objeto `Campanha` completo.

---

#### GET `/campanha/listar-campanhas`

**Response:**

```json
[
  {
    "id": 1,
    "titulo": "Campanha de Natal",
    "descricao": "Arrecadação de presentes",
    "ong": { ... },
    "meta": 10000.00,
    "valoratual": 2500.00,
    "datacriacao": "2024-11-01T10:00:00",
    "status": "ATIVA"
  }
]
```

---

#### DELETE `/campanha/{id}`

| Parâmetro | Tipo |
|-----------|------|
| `id` | Long |

**Parâmetro via query:** `?id=1`

---

## 5. Estrutura de Pastas

```
src/main/java/br/com/somar/backend_somar/
├── Controller/
│   ├── UsuarioController.java
│   ├── OngController.java
│   └── CampanhaController.java
├── Service/
│   ├── UsuarioService.java
│   ├── OngService.java
│   └── CampanhaService.java
├── Repository/
│   ├── UsuarioRepository.java
│   ├── OngRepository.java
│   └── CampanhaRepository.java
├── Model/
│   ├── Usuario.java
│   ├── Ong.java
│   └── Campanha.java
├── DTO/
│   ├── Requests/
│   │   ├── UsuarioCreateRequestDTO.java
│   │   ├── OngCreateRequestDTO.java
│   │   └── CampanhaCreateRequestDTO.java
│   └── Responses/
│       ├── UsuarioCreateResponseDTO.java
│       ├── OngCreateResponseDTO.java
│       └── CampanhaCreateResponseDTO.java
├── Enums/
│   ├── UsuarioEnum.java
│   └── CampanhaEnum.java
└── Mapping/
    ├── UsuarioMapper.java
    ├── OngMapper.java
    └── CampanhaMapper.java
```

---

## 6. Observações para o Frontend

### 6.1 Campos Obrigatórios por Endpoint

**Usuario:**
- `nome` (3-100 caracteres)
- `email`
- `senha`
- `tipo` (ONG | DOADOR | ADMIN)

**Ong:**
- `nome`
- `descricao`
- `documento`
- `telefone`
- `cidade`
- `estado`
- `codusuario` (ID de um usuário existente)

**Campanha:**
- `codong` (ID de uma ONG existente)
- `titulo`
- `descricao`
- `meta`
- `valoratual`
- `diafinalizado`
- `status` (ATIVA | FINALIZADA | CANCELADA)

### 6.2 Validações Importantes

1. **Criar ONG antes:** O `codusuario` deve ser de um usuário existente.
2. **Criar Campanha antes:** O `codong` deve ser de uma ONG existente.
3. **Login:** Retorna string simples, não gera token JWT.

---

> ⚠️ **Recomendação:** Implementar autenticação com JWT para maior segurança em produção.
