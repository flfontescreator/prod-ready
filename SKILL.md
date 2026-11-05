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

A numeração abaixo é a mesma de `references/performance-rules.md` e do `README.md`.

1. **Zero-Lag Architecture** *(geral)*: Métricas estritas de Core Web Vitals (LCP < 1.2s, INP < 50ms, CLS 0, TTFB < 150ms).
2. **Skeleton Screens** *(frontend)*: Telas de carregamento com esqueletos estruturados no formato exato do conteúdo final. Jamais use spinners de tela inteira que empurram o layout.
3. **Imagens em WebP/AVIF** *(frontend)*: Servir formatos modernos com tag `<picture>`, `srcset` responsivo e dimensões explícitas `width`/`height` para eliminar Cumulative Layout Shift (CLS). A imagem principal da tela (LCP) nunca recebe `loading="lazy"`, e sim `fetchpriority="high"`.
4. **Deduplicação de Requisições** *(frontend)*: Deduplicar apenas leituras (GET) idênticas disparadas simultaneamente (Promise cache / cache key). Escritas nunca são deduplicadas no cliente.
5. **Debounce com Cancelamento** *(frontend)*: Aplicar debounce de 250ms–300ms em buscas e autocompletes, cancelando requisições anteriores em voo via `AbortController`, verificando `res.ok` e abortando a requisição no cleanup do `useEffect`.
6. **Paginação Obrigatória em Coleções** *(backend)*: Toda API que retorna coleções deve implementar paginação por cursor ou limit/offset com teto rígido (máx. 50 ou 100 itens). Nunca retornar tabelas inteiras.
7. **CDN Global na Borda** *(infra)*: Servir todos os assets estáticos, imagens e scripts compilados via CDN (Cloudflare, CloudFront) com HTTP/3 e TLS 1.3.
8. **Filas em Segundo Plano para Tarefas >200ms** *(backend)*: Enviar e-mails, gerar relatórios, processar mídias ou chamadas pesadas fora da requisição HTTP (BullMQ, Celery, SQS), retornando `202 Accepted` de imediato com job ID.
9. **Eliminação de Consultas N+1** *(banco)*: Utilizar sempre eager loading (`include`, `JOIN`, subqueries agrupadas ou DataLoaders no GraphQL).
10. **Compressão Brotli / Gzip** *(infra)*: Respostas de texto compactadas prioritariamente com Brotli (`br`) e fallback para Gzip no gateway / edge.
11. **Cache com Redis (Cache-Aside Pattern)** *(backend)*: Proteger rotas de leitura frequente com Redis, TTLs explícitos e invalidação baseada em eventos (ao criar, editar ou excluir). A chave de cache inclui sempre o escopo de acesso (tenant e, quando o dado depende de permissão, o usuário). Ex.: `t:{tenantId}:orders:list:{hash(filtros)}`. Invalidação sempre depois do commit.
12. **Zero Dependências Não Utilizadas** *(geral)*: Auditar pacotes, fazer tree-shaking rigoroso e preferir APIs nativas da Web (`fetch`, `URL`, `Intl`) a bibliotecas pesadas.
13. **Timeouts Rígidos e Circuit Breaker** *(backend)*: Toda chamada a APIs externas deve ter timeout máximo de 3 a 5 segundos e mecanismo de circuit breaker para evitar bloqueio de threads.
14. **Streaming Token por Token para IA** *(backend)*: Respostas de modelos de linguagem devem ser transmitidas via Server-Sent Events (SSE) ou ReadableStreams, nunca acumulando a resposta inteira em buffer síncrono.
15. **Atualização Otimista (Optimistic UI)** *(frontend)*: Atualizar a interface instantaneamente nas ações do usuário com mecanismo de rollback automático em caso de erro na API.
16. **Lazy Loading Sob Demanda** *(frontend)*: Ativar `loading="lazy"` para imagens abaixo da dobra e `import()` dinâmico / `content-visibility: auto` para seções extensas. Nunca aplicar lazy na imagem principal da tela (LCP).
17. **Divisão de Código por Rota (Code Splitting)** *(frontend)*: Carregar no bundle inicial apenas o JavaScript essencial para a rota atual.
18. **Virtualização de Listas e Tabelas** *(frontend)*: Listas ou tabelas com mais de 50 registros devem usar virtualização DOM (TanStack Virtual / react-window), renderizando apenas os itens visíveis.
19. **Projeção Estrita de Colunas** *(banco)*: Proibido `SELECT *`. Selecionar explicitamente apenas as colunas consumidas pela interface (`SELECT id, name, email`).
20. **Connection Pooling** *(banco)*: Manter pool de conexões com banco ativo e reutilizável (PgBouncer, Prisma Pool, SQLAlchemy pool), evitando o custo de handshakes TCP/TLS por requisição.
21. **Prevenção de Re-renderizações** *(frontend)*: Isolar estados frequentes, memoizar cálculos pesados (`useMemo`/`useCallback`) e evitar passar objetos inline para componentes filhos.
22. **Headers de Cache Imutáveis** *(infra)*: Arquivos estáticos versionados com hash devem ter `Cache-Control: public, max-age=31536000, immutable`. O HTML de entrada deve usar `no-cache, must-revalidate`. HTML e APIs autenticadas: `Cache-Control: private, no-store`.
23. **Arquitetura 100% Stateless** *(infra)*: Instâncias de backend devem ser completamente sem estado, permitindo escalonamento horizontal automático atrás de um Load Balancer.
24. **Fontes Web Leves e Auto-Hospedadas** *(frontend)*: Hospedar fontes em WOFF2, limitar aos pesos estritamente necessários (<30KB por estilo) e usar `font-display: swap`.
25. **Monitoramento RUM & Testes em Redes Móveis Lentas** *(geral)*: Monitorar métricas reais de usuários (RUM) e testar obrigatoriamente sob emulação de rede móvel (Fast 3G / Slow 4G e CPU 4x Throttling) antes de qualquer deploy.

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
