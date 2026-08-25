Atue como um Arquiteto de Software Mobile Sênior especialista em React Native, Expo, TypeScript e React Native Paper, com vasta experiência no padrão Feature-Based Architecture.

Estou desenvolvendo um Sistema de Gestão de Treinamentos (mobile) baseado em uma API REST com recursos de autenticação, usuários, funcionários, instrutores, treinamentos, certificados, evidências, auditorias, etc.

Sua missão é gerar os entregáveis solicitados na Aula 01 do projeto:

Mapeamento de Features: Analise a API de treinamentos e defina o agrupamento ideal de Features no aplicativo (ex: auth, dashboard, treinamentos, certificados, perfil). Explique o porquê da fusão ou eliminação de endpoints puramente relacionais da API na UI mobile.

Estruturação da Pasta src/: Apresente a árvore completa de diretórios seguindo a divisão rígida em 4 pilares (app/, core/, shared/, features/), aplicando a comunicação em camadas (Screen ➔ Hook ➔ Service ➔ API).

Fluxo e Mapa de Navegação: Detalhe o mapa visual/textual de telas (fluxo condicional de autenticação ➔ Dashboard ➔ Tabs/Stack de Treinamentos e Certificados) indicando a Feature e o endpoint HTTP de cada tela.

Respostas aos Questionamentos Técnicos:

Qual o impacto da mudança da Mock API para a API Real no projeto? Quantos e quais arquivos precisam ser alterados?

Quem é responsável por cada parte do fluxo de dados (UI, controle, chamada HTTP, config Axios/Fetch, token storage)?

Draft do Documento docs/arquitetura.md: Escreva a documentação oficial justificando 5 decisões arquiteturais críticas adotadas no projeto para garantir escalabilidade para novos devs nos próximos 6 meses.

Adote um tom técnico, claro, sênior e direto ao ponto. Use TypeScript e boas práticas de Clean Code nos exemplos.