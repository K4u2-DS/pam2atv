# Especificação Funcional e Técnica (`spec.md`)

## 1. Visão Geral do Sistema

O **muck_mob** é o aplicativo mobile do **Sistema de Gestão de Treinamentos**, desenvolvido em **React Native + Expo**. 

O objetivo do aplicativo é permitir que os colaboradores/usuários consultem treinamentos, acompanhem o status de suas capacitações, visualizem detalhes de instrutores e participantes, e acessem seus certificados de conclusão.

---

## 2. Escopo do Aplicativo Mobile

### Inclusos na Versão Atual
* Autenticação de usuários via API REST.
* Dashboard com visão geral dos treinamentos e métricas iniciais.
* Listagem e detalhamento completo de treinamentos (instrutores, responsáveis, participantes e evidências).
* Consulta e visualização de certificados obtidos.
* Perfil do usuário logado.

### Fora de Escopo nesta Etapa
* Cadastro ou edição administrativa de novos treinamentos/usuários.
* Emissão de novas assinaturas digitais ou auditoria de sistema.

---

## 3. Mapeamento de Recursos da API vs. Features do Mobile

Para manter o app escalável e focado na experiência do usuário (UI/UX), os endpoints da API foram agrupados por **Features de Negócio**:

| Feature Mobile | Endpoint(s) da API Utilizado(s) | Responsabilidade da Feature |
|---|---|---|
| **`auth`** | `POST /api/login` | Login, logout e validação de token de sessão. |
| **`dashboard`** | `GET /api/dashboard` (ou agregações) | Exibição de métricas rápidas e próximos treinamentos. |
| **`treinamentos`** | `GET /api/treinamentos`<br>`GET /api/treinamentos/{id}`<br>`GET /api/treinamentos/{id}/completo` | Listagem e detalhes completos do treinamento. |
| **`certificados`** | `GET /api/certificados`<br>`GET /api/certificados/{id}` | Consulta e exibição de certificados do usuário. |
| **`perfil`** | `GET /api/usuarios/me` (ou equivalente) | Dados da conta logada e informações do funcionário. |

> **Nota de Arquitetura:** Recursos relacionais da API (`treinamentoInstrutores`, `treinamentoParticipantes`, `evidencias`) não possuem Features isoladas. Eles são encapsulados dentro do fluxo da Feature **`treinamentos`** (utilizando chamadas como `/completo`).

---

## 4. Requisitos Funcionais por Feature

### 4.1. Feature `auth`
* **RF01 - Login:** O usuário deve informar credenciais (e-mail/CPF e senha) para se autenticar.
* **RF02 - Persistência:** O token JWT retornado deve ser armazenado com segurança no dispositivo.
* **RF03 - Proteção de Rota:** Caso o token seja inválido ou expirado, o app deve redirecionar automaticamente para a tela de Login.

### 4.2. Feature `dashboard`
* **RF04 - Resumo de Indicadores:** Apresentar a quantidade de treinamentos pendentes e concluídos.
* **RF05 - Acesso Rápido:** Permitir atalho direto para os treinamentos em andamento.

### 4.3. Feature `treinamentos`
* **RF06 - Listagem:** Exibir a lista de treinamentos disponíveis e seus status (Em andamento, Concluído, Pendente).
* **RF07 - Filtro/Busca:** Permitir filtrar treinamentos por nome ou status.
* **RF08 - Detalhe Completo:** Na tela de detalhes (`/completo`), exibir:
  * Informações básicas (título, carga horária, datas).
  * Lista de instrutores e responsáveis.
  * Lista de participantes inscritos.
  * Evidências anexadas.

### 4.4. Feature `certificados`
* **RF09 - Consulta de Certificados:** Listar os certificados emitidos para o colaborador.
* **RF10 - Detalhes do Certificado:** Exibir código de validação, data de emissão e detalhes da carga horária cumprida.

### 4.5. Feature `perfil`
* **RF11 - Dados do Usuário:** Exibir nome, cargo, e-mail e informações do funcionário associado.
* **RF12 - Logout:** Permitir encerrar a sessão limpando os dados salvos localmente.

---

## 5. Requisitos Não-Funcionais

* **RNF01 - UI Framework:** Todas as telas devem utilizar componentes visuais da biblioteca **React Native Paper**.
* **RNF02 - Desacoplamento:** Nenhuma tela pode fazer requisições diretas via `axios` ou `fetch`. O fluxo obrigatório é `Screen -> Hook -> Service -> API`.
* **RNF03 - Responsividade:** A interface deve adaptar-se adequadamente a diferentes tamanhos de tela (Android e iOS).
* **RNF04 - Tratamento de Erros:** Exibir mensagens de erro amigáveis ao usuário (ex: falha de conexão, credenciais inválidas) sem quebrar o aplicativo.

Regras de Negócio e Telas: Consulte sempre o arquivo spec.md para validar requisitos, endpoints e critérios de aceite e após alterações bem feitas, salve às lá.

Histórico e Decisões: Consulte o memory.md para manter a consistência com decisões arquiteturais anteriores.

Ponto de Entrada: Respeite as configurações do App.tsx (Providers do React Native Paper, Navigation Container, etc.).