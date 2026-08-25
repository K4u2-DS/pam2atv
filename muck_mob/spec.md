# Especificação Funcional e Técnica (`spec.md`)

## 1. Visão Geral do Sistema

O **muck_mob** é o aplicativo mobile do **Sistema de Gestão de Treinamentos**, desenvolvido em **React Native (Expo SDK 57)** + **TypeScript (strict)** + **React Native Paper**.

O objetivo é permitir que colaboradores consultem treinamentos, acompanhem o status de capacitações, visualizem detalhes de instrutores/participantes e acessem certificados de conclusão.

**Stack tecnológica:**

| Camada | Tecnologia |
|--------|------------|
| Framework | Expo SDK 57 |
| UI | React Native Paper |
| Navegação | React Navigation (native-stack + bottom-tabs) |
| HTTP | Axios |
| Storage | expo-secure-store (JWT) |
| Linguagem | TypeScript (strict mode) |

---

## 2. Escopo do Aplicativo Mobile

### Inclusos na Versão Atual
- Autenticação de usuários via API REST (JWT).
- Dashboard com visão geral dos treinamentos e métricas.
- Listagem e detalhamento completo de treinamentos.
- Consulta e visualização de certificados.
- Perfil do usuário logado.

### Fora de Escopo nesta Etapa
- Cadastro ou edição administrativa de treinamentos/usuários.
- Emissão de assinaturas digitais ou auditoria de sistema.
- Push notifications.
- Modo offline.

---

## 3. Arquitetura Feature-Based — 4 Pilares

```
src/
├── app/           # Navegação e providers globais
│   ├── navigation/
│   │   ├── AppNavigator.tsx
│   │   ├── AuthNavigator.tsx
│   │   └── types.ts
│   └── providers/
│       ├── AppProviders.tsx
│       └── AuthProvider.tsx
├── core/          # Infraestrutura transversal
│   ├── api/
│   │   ├── api.ts              # Instância axios
│   │   ├── apiTypes.ts         # Tipos genéricos de resposta
│   │   └── interceptors.ts     # Request/Response interceptors
│   ├── config/
│   │   └── env.ts              # Base URL, variáveis de ambiente
│   ├── errors/
│   │   └── ApiError.ts         # Classe de erro customizada
│   └── storage/
│       └── storage.ts          # Wrapper de armazenamento seguro
├── shared/        # Componentes e utilitários reutilizáveis
│   ├── components/
│   ├── hooks/
│   └── utils/
└── features/      # Módulos de negócio
    ├── auth/
    ├── dashboard/
    ├── treinamentos/  # Inclui certificados
    └── perfil/
```

### Fluxo de Dados Obrigatório

```
Screen → Hook → Service → API (axios)
```

- **Screen**: Renderiza UI (React Native Paper), delega lógica ao Hook.
- **Hook**: Gerencia estado (`useState`/`useEffect`), orquestra chamadas, expõe dados e callbacks.
- **Service**: Contém lógica de chamada HTTP, retorna dados tipados.
- **API (core)**: Instância axios com interceptors (token, tratamento 401).

---

## 4. Mapeamento de Features vs. Endpoints

| Feature | Tela(s) | Endpoint(s) HTTP | Responsabilidade |
|---------|---------|-------------------|------------------|
| **auth** | `LoginScreen` | `POST /api/login` | Login, logout, validação de token |
| **dashboard** | `DashboardScreen` | `GET /api/dashboard` | Métricas rápidas, próximos treinamentos |
| **treinamentos** | `TreinamentosScreen` | `GET /api/treinamentos` | Listagem com filtros |
| | `TreinamentoDetalhesScreen` | `GET /api/treinamentos/{id}/completo` | Detalhe completo (instrutores, participantes, evidências) |
| | `CertificadosScreen` | `GET /api/certificados` | Lista de certificados do usuário |
| | `CertificadoFuncionariosScreen` | `GET /api/certificados/{id}` | Detalhe do certificado |
| **perfil** | `PerfilScreen` | `GET /api/usuarios/me` | Dados do usuário logado |

> **Nota:** Recursos relacionais (`treinamentoInstrutores`, `treinamentoParticipantes`, `evidencias`) são encapsulados no endpoint `/completo` dentro da feature `treinamentos`.

---

## 5. Contratos de API — Tipos TypeScript

### 5.1. Tipos Genéricos (`src/core/api/apiTypes.ts`)

```typescript
export interface ApiResponse<T> {
  data: T;
  message?: string;
  status: number;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  perPage: number;
}
```

### 5.2. Auth (`src/features/auth/types/authTypes.ts`)

```typescript
export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  token: string;
  user: User;
}

export interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  funcionario?: Funcionario;
}

export interface Funcionario {
  id: number;
  nome: string;
  cargo: string;
  setor: string;
}
```

### 5.3. Treinamentos (`src/features/treinamentos/types/treinamentoTypes.ts`)

```typescript
export type StatusTreinamento = 'em_andamento' | 'concluido' | 'pendente';

export interface Treinamento {
  id: number;
  titulo: string;
  descricao: string;
  cargaHoraria: number;
  dataInicio: string;
  dataFim: string;
  status: StatusTreinamento;
}

export interface TreinamentoCompleto extends Treinamento {
  instrutores: Instrutor[];
  responsaveis: Responsavel[];
  participantes: Participante[];
  evidencias: Evidencia[];
}

export interface Instrutor {
  id: number;
  nome: string;
  especialidade: string;
}

export interface Responsavel {
  id: number;
  nome: string;
  cargo: string;
}

export interface Participante {
  id: number;
  nome: string;
  status: 'concluido' | 'pendente' | 'em_andamento';
}

export interface Evidencia {
  id: number;
  descricao: string;
  url: string;
  tipo: string;
}
```

### 5.4. Certificados (`src/features/treinamentos/types/certificadoTypes.ts`)

```typescript
export interface Certificado {
  id: number;
  codigoValidacao: string;
  dataEmissao: string;
  cargaHoraria: number;
  treinamento: {
    id: number;
    titulo: string;
  };
}
```

### 5.5. Dashboard (`src/features/dashboard/types/dashboardTypes.ts`)

```typescript
export interface DashboardData {
  treinamentosPendentes: number;
  treinamentosConcluidos: number;
  treinamentosEmAndamento: number;
  proximosTreinamentos: Treinamento[];
}
```

### 5.6. Perfil (`src/features/perfil/types/perfilTypes.ts`)

```typescript
export interface PerfilData {
  id: number;
  nome: string;
  email: string;
  cargo: string;
  funcionario: Funcionario;
}
```

---

## 6. Requisitos Funcionais por Feature

### 6.1. Feature `auth`

| RF | Descrição | Critério de Aceite |
|----|-----------|---------------------|
| **RF01** | Login com credenciais | O usuário informa e-mail/CPF + senha. O app envia `POST /api/login`. Em sucesso, navega para Dashboard. Em falha, exibe mensagem de erro. |
| **RF02** | Persistência do token | O JWT retornado é armazenado via `expo-secure-store`. O token é anexado automaticamente a todas as requisições via interceptor. |
| **RF03** | Proteção de rota | Se o token não existir ou for inválido (401), o app redireciona para `LoginScreen` automaticamente. |

**Tela `LoginScreen`:**
- Componentes Paper: `TextInput` (e-mail, senha), `Button` (entrar), `ActivityIndicator` (loading).
- Validação client-side: campos obrigatórios antes do envio.

### 6.2. Feature `dashboard`

| RF | Descrição | Critério de Aceite |
|----|-----------|---------------------|
| **RF04** | Resumo de indicadores | Exibe cards com quantidade de treinamentos pendentes, concluídos e em andamento. |
| **RF05** | Acesso rápido | Botão/atalho para navegar diretamente para a lista de treinamentos em andamento. |

**Tela `DashboardScreen`:**
- Componentes Paper: `Card` (indicadores), `Button` (atalho), `ActivityIndicator`.
- Layout: grid de cards com ícones e contadores.

### 6.3. Feature `treinamentos`

| RF | Descrição | Critério de Aceite |
|----|-----------|---------------------|
| **RF06** | Listagem | Exibe lista de treinamentos com título, status e carga horária. Pull-to-refresh habilitado. |
| **RF07** | Filtro/Busca | Campo de busca por nome + filtro por status (em andamento, concluído, pendente). |
| **RF08** | Detalhe completo | Tela com abas ou seções: dados gerais, instrutores, participantes, evidências. Dados vindos de `/completo`. |
| **RF09** | Lista de certificados | Lista os certificados do usuário com código, data e carga horária. |
| **RF10** | Detalhe do certificado | Exibe código de validação, data de emissão, carga horária e treinamento associado. |

**Telas:**
- `TreinamentosScreen`: `FlatList` com `TreinamentoCard`, `Searchbar` (Paper), `Chip` para filtros.
- `TreinamentoDetalhesScreen`: `Card` com seções, `List.Section` para instrutores/participantes.
- `CertificadosScreen`: `FlatList` com cards de certificado.
- `CertificadoFuncionariosScreen`: Detalhe com `Card` e informações de validação.

### 6.4. Feature `perfil`

| RF | Descrição | Critério de Aceite |
|----|-----------|---------------------|
| **RF11** | Dados do usuário | Exibe nome, cargo, e-mail e dados do funcionário. |
| **RF12** | Logout | Botão de logout limpa token do storage e navega para `LoginScreen`. |

**Tela `PerfilScreen`:**
- Componentes Paper: `Card` (dados do usuário), `Button` (logout), `List.Item` (informações).
- Ícone de usuário no topo.

---

## 7. Requisitos Não-Funcionais

| RNF | Descrição | Detalhamento |
|-----|-----------|--------------|
| **RNF01** | UI Framework | Todas as telas usam componentes **React Native Paper**. Nenhum componente nativo estilizado manualmente. |
| **RNF02** | Desacoplamento | Fluxo obrigatório: `Screen → Hook → Service → API`. Sem `axios`/`fetch` direto nas telas. |
| **RNF03** | Responsividade | Layout adaptivo para Android e iOS. Usar `flex`, `%`, e `Dimensions` para responsividade. |
| **RNF04** | Tratamento de erros | Mensagens amigáveis via `Snackbar` ou `Dialog` (Paper). Logs no console para debug. Sem crashes. |
| **RNF05** | Tipagem | TypeScript strict. Todos os parâmetros, retornos e props tipados. Sem `any`. |
| **RNF06** | Navegação | Fluxo condicional: token válido → AppNavigator (tabs); token inválido → AuthNavigator (stack). |

---

## 8. Navegação

### 8.1. Fluxo Condicional

```
┌──────────────────────────┐
│     AppProviders         │
│  (Paper Theme + Auth)    │
└────────────┬─────────────┘
             │
     ┌───────┴───────┐
     │ Token válido?  │
     └───┬───────┬───┘
      SIM│       │NÃO
         v       v
  ┌──────────┐ ┌──────────┐
  │   App    │ │   Auth   │
  │Navigator │ │Navigator │
  └────┬─────┘ └────┬─────┘
       │            │
       v            v
  ┌──────────┐ ┌──────────┐
  │ Bottom   │ │  Login   │
  │  Tabs    │ │  Screen  │
  └──────────┘ └──────────┘
```

### 8.2. Rotas

| Navigator | Tab/Screen | Rota | Feature |
|-----------|------------|------|---------|
| `AuthNavigator` | LoginScreen | `login` | auth |
| `AppNavigator` | DashboardScreen | `dashboard` | dashboard |
| `AppNavigator` | TreinamentosScreen | `treinamentos` | treinamentos |
| `AppNavigator` | TreinamentoDetalhesScreen | `treinamento-detalhes` | treinamentos |
| `AppNavigator` | CertificadosScreen | `certificados` | treinamentos |
| `AppNavigator` | CertificadoFuncionariosScreen | `certificado-detalhes` | treinamentos |
| `AppNavigator` | PerfilScreen | `perfil` | perfil |

### 8.3. Tipos de Navegação (`src/app/navigation/types.ts`)

```typescript
export type RootStackParamList = {
  login: undefined;
  dashboard: undefined;
  treinamentos: undefined;
  'treinamento-detalhes': { treinamentoId: number };
  certificados: undefined;
  'certificado-detalhes': { certificadoId: number };
  perfil: undefined;
};
```

---

## 9. Configuração do Tema (React Native Paper)

```typescript
// src/app/providers/AppProviders.tsx
import { PaperProvider, MD3LightTheme } from 'react-native-paper';

const theme = {
  ...MD3LightTheme,
  colors: {
    ...MD3LightTheme.colors,
    primary: '#1976D2',
    secondary: '#FFC107',
  },
};
```

---

## 10. Tratamento de Erros

### 10.1. Classe `ApiError` (`src/core/errors/ApiError.ts`)

```typescript
export class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public data?: unknown,
  ) {
    super(message);
    this.name = 'ApiError';
  }
}
```

### 10.2. Interceptor de Resposta

- **401**: Limpar token, navegar para `login`.
- **403**: Exibir "Acesso negado".
- **404**: Exibir "Recurso não encontrado".
- **500+**: Exibir "Erro interno. Tente novamente."

### 10.3. UX de Erros

- **Loading**: `ActivityIndicator` centralizado na tela.
- **Erro de rede**: `Snackbar` com botão "Tentar novamente".
- **Erro de credenciais**: Mensagem inline no campo de senha.
- **Lista vazia**: `EmptyState` com ícone e texto explicativo.

---

## 11. Ordem de Implementação Sugerida

| Fase | O que implementar | Arquivos-chave |
|------|-------------------|----------------|
| 1 | Dependências + core | `package.json`, `api.ts`, `interceptors.ts`, `storage.ts`, `env.ts`, `ApiError.ts` |
| 2 | Auth + Navegação | `AuthProvider.tsx`, `AuthNavigator.tsx`, `AppNavigator.tsx`, `types.ts`, `AppProviders.tsx` |
| 3 | Feature auth | `LoginScreen.tsx`, `useAuth.ts`, `authService.ts` |
| 4 | Feature dashboard | `DashboardScreen.tsx`, `useDashboard.ts`, `dashboardService.ts` |
| 5 | Feature treinamentos | Telas, hooks, services, components |
| 6 | Feature perfil | `PerfilScreen.tsx`, `usePerfil.ts`, `perfilService.ts` |
| 7 | App.tsx wiring | Providers + NavigationContainer |
| 8 | Documentação | `docs/arquitetura.md` |

---

## 12. Referências

- `ai.md` — Prompt de arquitetura e convenções de código.
- `memory.md` — Decisões arquiteturais tomadas (manter consistência).
- `docs/arquitetura.md` — Documentação oficial de arquitetura.
- Expo SDK 57: https://docs.expo.dev/versions/v57.0.0/