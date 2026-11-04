---
name: prod-ready
description: Enforce production-grade engineering, 25 full-stack performance architecture parameters (Redis caching, background queues, skeletons, debouncing, pagination, zero N+1, AVIF/WebP), and an authentic human-designed Design System reference that stops generic AI-slop code and slow apps from day one.
---

# Prod-Ready Skill: Engenharia de Produção & Design System Anti-AI-Slop

Esta skill transforma o Claude em um arquiteto sênior de software e designer de interfaces de nível de produção. Toda vez que uma aplicação, componente, API ou banco de dados for planejado, desenvolvido ou refatorado sob esta skill, **é estritamente proibido gerar código lento, ingênuo ou com interface genérica de IA** ("AI-slop").

---

## 🎯 Gatilhos de Ativação

Invoque ou aplique as regras desta skill quando:
- O usuário solicitar: `prod-ready`, `/prod-ready`, "crie um app de produção", "sem cara de IA", "sem lentidão", "otimize para produção".
- Qualquer nova aplicação web, dashboard, API REST/GraphQL, microsserviço ou tela estiver sendo desenvolvida do zero.
- Houver necessidade de arquitetar caching, filas em segundo plano, escalabilidade ou design system consistente.

---

## 🎨 O Padrão de Design System Anti-AI-Slop

Modelos de linguagem frequentemente caem em vícios de design previsíveis: cards roxos/azuis com gradientes genéricos, texto cinza claro sem contraste sobre fundo branco, layouts de coluna única centralizados e spinners de tela cheia que causam saltos visuais.

### Diretrizes Obrigatórias de UI:
1. **Consulte a Referência Oficial**:
   - Leia as convenções de tokens, tipografia e componentes em `references/design-system.html`.
2. **Tokens Semânticos Estruturados**:
   - Utilize sempre variáveis CSS no `:root` para cores, espaçamentos, tipografia, bordas e sombras.
   - Respeite rigorosamente contraste mínimo **WCAG AA (4.5:1) ou AAA (7:1)** para leitura confortável.
3. **Escala Tipográfica Hierárquica**:
   - Utilize pesos reais com propósitos definidos: `400` (corpo), `600` (subtítulos e ênfases), `800` (títulos de destaque) e `900` (display).
   - Defina sempre fallbacks para fontes do sistema (`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto...`) e declare `font-display: swap`.
4. **Layouts Reais para Aplicações Complexas**:
   - Estrutura completa: Topbar com navegação rápida, busca global e ações; Sidebar categorizada com badges e estados ativos; Drawer deslizante para dispositivos móveis; Breadcrumbs e cabeçalhos de contexto claros.
5. **Componentes com Acessibilidade Nativa (WAI-ARIA)**:
   - Abas funcionais com `role="tablist"`, `role="tab"` e `aria-selected`.
   - Modais com backdrop, contenção de foco e fechamento no `Escape`.
   - Atalhos de teclado ergonômicos (tecla `/` ou `Ctrl+K` para busca instantânea).
   - Accordions nativos via `<details>` e `<summary>`.

---

## ⚡ Os 25 Mandamentos de Performance & Arquitetura

Ao gerar qualquer código de backend, frontend, banco de dados ou infraestrutura, aplique obrigatoriamente os 25 parâmetros (especificação completa em `references/performance-rules.md`):

### Frontend & Experiência do Usuário (Zero-Lag)
1. **Zero-Lag Architecture**: Métricas estritas de Core Web Vitals (LCP < 1.2s, INP < 50ms, CLS 0, TTFB < 150ms).
2. **Skeleton Screens**: Telas de carregamento com esqueletos estruturados no formato exato do conteúdo final. Jamais use spinners de tela inteira que empurram o layout.
3. **Imagens em WebP/AVIF**: Servir formatos modernos com tag `<picture>`, `srcset` responsivo e dimensões explícitas `width`/`height` para eliminar Cumulative Layout Shift (CLS).
4. **Deduplicação de Requisições**: Deduplicar chamadas de API idênticas disparadas simultaneamente no mesmo ciclo de vida (Promise cache / cache key).
5. **Debounce com Cancelamento**: Aplicar debounce de 250ms–300ms em buscas e autocompletes, cancelando requisições anteriores em voo via `AbortController`.
6. **Atualização Otimista (Optimistic UI)**: Atualizar a interface instantaneamente nas ações do usuário com mecanismo de rollback automático em caso de erro na API.
7. **Lazy Loading Sob Demanda**: Ativar `loading="lazy"` para imagens abaixo da dobra e `import()` dinâmico / `content-visibility: auto` para seções extensas.
8. **Divisão de Código por Rota (Code Splitting)**: Carregar no bundle inicial apenas o JavaScript essencial para a rota atual.
9. **Virtualização de Listas e Tabelas**: Listas ou tabelas com mais de 50 registros devem usar virtualização DOM (TanStack Virtual / react-window), renderizando apenas os itens visíveis.
10. **Prevenção de Re-renderizações**: Isolar estados frequentes, memoizar cálculos pesados (`useMemo`/`useCallback`) e evitar passar objetos inline para componentes filhos.
11. **Fontes Web Leves e Auto-Hospedadas**: Hospedar fontes em WOFF2, limitar aos pesos estritamente necessários (<30KB por estilo) e usar `font-display: swap`.

### Backend, APIs & Processamento Assíncrono
12. **Streaming Token por Token para IA**: Respostas de modelos de linguagem devem ser transmitidas via Server-Sent Events (SSE) ou ReadableStreams, nunca acumulando a resposta inteira em buffer síncrono.
13. **Filas em Segundo Plano para Tarefas >200ms**: Enviar e-mails, gerar relatórios, processar mídias ou chamadas pesadas fora da requisição HTTP (BullMQ, Celery, SQS), retornando `202 Accepted` de imediato com job ID.
14. **Timeouts Rígidos e Circuit Breaker**: Toda chamada a APIs externas deve ter timeout máximo de 3 a 5 segundos e mecanismo de circuit breaker para evitar bloqueio de threads.
15. **Paginação Obrigatória em Coleções**: Toda API que retorna coleções deve implementar paginação por cursor ou limit/offset com teto rígido (máx. 50 ou 100 itens). Nunca retornar tabelas inteiras.
16. **Cache com Redis (Cache-Aside Pattern)**: Proteger rotas de leitura frequente com Redis, TTLs explícitos e invalidação baseada em eventos (ao criar, editar ou excluir).
17. **Zero Dependências Não Utilizadas**: Auditar pacotes, fazer tree-shaking rigoroso e preferir APIs nativas da Web (`fetch`, `URL`, `Intl`) a bibliotecas pesadas.
18. **Compressão Brotli / Gzip**: Respostas de texto compactadas prioritariamente com Brotli (`br`) e fallback para Gzip no gateway / edge.

### Banco de Dados & Infraestrutura
19. **Eliminação de Consultas N+1**: Utilizar sempre eager loading (`include`, `JOIN`, subqueries agrupadas ou DataLoaders no GraphQL).
20. **Projeção Estrita de Colunas**: Proibido `SELECT *`. Selecionar explicitamente apenas as colunas consumidas pela interface (`SELECT id, name, email`).
21. **Connection Pooling**: Manter pool de conexões com banco ativo e reutilizável (PgBouncer, Prisma Pool, SQLAlchemy pool), evitando o custo de handshakes TCP/TLS por requisição.
22. **Headers de Cache Imutáveis**: Arquivos estáticos versionados com hash devem ter `Cache-Control: public, max-age=31536000, immutable`. O HTML de entrada deve usar `no-cache, must-revalidate`.
23. **Arquitetura 100% Stateless**: Instâncias de backend devem ser completamente sem estado, permitindo escalonamento horizontal automático atrás de um Load Balancer.
24. **CDN Global na Borda**: Servir todos os assets estáticos, imagens e scripts compilados via CDN (Cloudflare, CloudFront) com HTTP/3 e TLS 1.3.
25. **Monitoramento RUM & Testes em Redes Móveis Lentas**: Monitorar métricas reais de usuários (RUM) e testar obrigatoriamente sob emulação de rede móvel (Fast 3G / Slow 4G e CPU 4x Throttling) antes de qualquer deploy.

---

## 🛠️ Checklist de Execução ao Gerar Código

Ao receber um pedido de código, execute mentalmente e reflita no artefato:
- [ ] O componente tem estado de carregamento via **Skeleton** correspondente ao layout final?
- [ ] O input de busca possui **debounce** e cancelamento com **AbortController**?
- [ ] A API de consulta tem **paginação** e **projeção explícita de colunas**?
- [ ] Foi verificado se há consultas **N+1** nos relacionamentos do banco?
- [ ] Tarefas lentas (>200ms) foram movidas para **filas em segundo plano**?
- [ ] A resposta da IA está sendo enviada via **streaming (SSE)**?
- [ ] O design segue a hierarquia, contraste e tokens de `references/design-system.html` em vez de um template genérico de IA?
