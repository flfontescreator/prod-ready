# Guia de Engenharia de Performance & Arquitetura Prod-Ready

Este guia detalha os **25 Mandamentos de Performance & Arquitetura** exigidos pela skill `prod-ready`. Toda aplicação desenvolvida sob esta regra deve implementar estes padrões desde a primeira linha de código, eliminando gargalos, desperdício de recursos e a geração de interfaces lentas e genéricas.

---

## Índice dos 25 Parâmetros

1. [Zero-Lag Architecture & Core Web Vitals](#1-zero-lag-architecture--core-web-vitals)
2. [Telas de Carregamento com Skeleton Screens](#2-telas-de-carregamento-com-skeleton-screens)
3. [Imagens Otimizadas em WebP / AVIF & Responsive Srcset](#3-imagens-otimizadas-em-webp--avif--responsive-srcset)
4. [Deduplicação de Chamadas de API em Voo](#4-deduplicação-de-chamadas-de-api-em-voo)
5. [Debounce e Cancelamento na Busca e Autocomplete](#5-debounce-e-cancelamento-na-busca-e-autocomplete)
6. [Paginação Obrigatória em Todas as APIs de Coleção](#6-paginação-obrigatória-em-todas-as-apis-de-coleção)
7. [Distribuição de Arquivos Estáticos via CDN Global](#7-distribuição-de-arquivos-estáticos-via-cdn-global)
8. [Filas Assíncronas em Segundo Plano para Tarefas >200ms](#8-filas-assíncronas-em-segundo-plano-para-tarefas-200ms)
9. [Eliminação Total de Consultas N+1](#9-eliminação-total-de-consultas-n1)
10. [Compressão Brotli e Gzip no Edge / Proxy Reverso](#10-compressão-brotli-e-gzip-no-edge--proxy-reverso)
11. [Cache Estratégico com Redis (Cache-Aside Pattern)](#11-cache-estratégico-com-redis-cache-aside-pattern)
12. [Auditoria e Eliminação de Dependências Desnecessárias](#12-auditoria-e-eliminação-de-dependências-desnecessárias)
13. [Timeouts Rígidos e Circuit Breakers em APIs Externas](#13-timeouts-rígidos-e-circuit-breakers-em-apis-externas)
14. [Streaming de Respostas de IA Token por Token (SSE)](#14-streaming-de-respostas-de-ia-token-por-token-sse)
15. [Atualização Otimista da Interface (Optimistic UI)](#15-atualização-otimista-da-interface-optimistic-ui)
16. [Lazy Loading Nativo e Carregamento Sob Demanda](#16-lazy-loading-nativo-e-carregamento-sob-demanda)
17. [Divisão de Código por Rota (Route-Based Code Splitting)](#17-divisão-de-código-por-rota-route-based-code-splitting)
18. [Virtualização de Listas e Tabelas Longas (>50 itens)](#18-virtualização-de-listas-e-tabelas-longas-50-itens)
19. [Projeção Estrita de Colunas SQL (Zero `SELECT *`)](#19-projeção-estrita-de-colunas-sql-zero-select-)
20. [Reutilização e Pooling de Conexões de Banco de Dados](#20-reutilização-e-pooling-de-conexões-de-banco-de-dados)
21. [Prevenção de Re-renderizações Desnecessárias no Frontend](#21-prevenção-de-re-renderizações-desnecessárias-no-frontend)
22. [Headers de Cache Imutáveis no Navegador](#22-headers-de-cache-imutáveis-no-navegador)
23. [Arquitetura 100% Stateless e Escalonamento Horizontal](#23-arquitetura-100-stateless-e-escalonamento-horizontal)
24. [Fontes Web Auto-Hospedadas, Subsettadas e Otimizadas](#24-fontes-web-auto-hospedadas-subsettadas-e-otimizadas)
25. [Monitoramento RUM de Velocidade Real e Testes em Redes Lentas](#25-monitoramento-rum-de-velocidade-real-e-testes-em-redes-lentas)

---

### 1. Zero-Lag Architecture & Core Web Vitals
- **Meta**: Garantir que o aplicativo nunca perca usuários por lentidão perceptível.
- **Métricas Alvo**:
  - **TTFB (Time to First Byte)**: < 150ms
  - **LCP (Largest Contentful Paint)**: < 1.2s
  - **INP (Interaction to Next Paint)**: < 50ms
  - **CLS (Cumulative Layout Shift)**: 0.00
- **Regra**: Toda funcionalidade nova deve ser projetada para manter o thread principal desimpedido. Operações síncronas no render cycle que excedam 16ms são estritamente proibidas.

---

### 2. Telas de Carregamento com Skeleton Screens
- **Anti-padrão (AI Slop)**: Um spinner giratório centralizado que não comunica a estrutura do conteúdo e causa grande salto de layout (CLS) quando os dados chegam.
- **Padrão Prod-Ready**: Skeletons construídos com elementos semânticos que espelham exatamente a geometria final do card, tabela ou formulário, animados com gradiente sutil via CSS (`@keyframes shimmer`).

```tsx
// Prod-Ready Skeleton Card
export function CardSkeleton() {
  return (
    <div className="rounded-lg border border-neutral-200 p-4 space-y-3 bg-white" aria-busy="true" aria-live="polite">
      <div className="h-4 w-1/3 bg-neutral-200 rounded animate-pulse" />
      <div className="h-8 w-2/3 bg-neutral-200 rounded animate-pulse" />
      <div className="space-y-2 pt-2">
        <div className="h-3 w-full bg-neutral-100 rounded animate-pulse" />
        <div className="h-3 w-4/5 bg-neutral-100 rounded animate-pulse" />
      </div>
    </div>
  );
}
```

---

### 3. Imagens Otimizadas em WebP / AVIF & Responsive Srcset
- **Regra**: Nenhuma imagem bruta em PNG/JPEG de tamanho arbitrário em produção.
- **Implementação**:
  - Servir prioritariamente AVIF com fallback para WebP.
  - Especificar sempre atributos explícitos `width` e `height` (ou `aspect-ratio` no CSS) para reservar espaço e anular CLS.
  - Utilizar `srcset` e `sizes` para entregar o menor payload compatível com o viewport do dispositivo.

```html
<picture>
  <source type="image/avif" srcset="avatar-320.avif 320w, avatar-640.avif 640w" sizes="(max-width: 640px) 100vw, 320px">
  <source type="image/webp" srcset="avatar-320.webp 320w, avatar-640.webp 640w" sizes="(max-width: 640px) 100vw, 320px">
  <img src="avatar-320.jpg" alt="Avatar do usuário" width="320" height="320" loading="lazy" decoding="async">
</picture>
```

---

### 4. Deduplicação de Chamadas de API em Voo
- **Anti-padrão**: 3 componentes irmãos no mesmo dashboard disparando 3 requisições HTTP idênticas para `/api/user/profile`.
- **Padrão Prod-Ready**: Deduplicador de requisições em voo via Promise Cache ou biblioteca com suporte nativo a cache key deduplication (TanStack Query / SWR / Fetch Cache nativo).

```ts
// In-flight Request Deduplicator
const inflightRequests = new Map<string, Promise<any>>();

export function deduplicatedFetch<T>(url: string, init?: RequestInit): Promise<T> {
  const key = `${init?.method || 'GET'}:${url}`;
  if (inflightRequests.has(key)) {
    return inflightRequests.get(key)!;
  }

  const promise = fetch(url, init)
    .then(async (res) => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json() as Promise<T>;
    })
    .finally(() => {
      inflightRequests.delete(key);
    });

  inflightRequests.set(key, promise);
  return promise;
}
```

---

### 5. Debounce e Cancelamento na Busca e Autocomplete
- **Regra**: Campos de pesquisa e autocomplete devem aplicar debounce de 250ms a 300ms E cancelar a requisição anterior em andamento usando `AbortController` quando um novo caractere for digitado.

```ts
export function useDebouncedSearch(query: string, delay = 250) {
  const [results, setResults] = useState([]);
  const abortControllerRef = useRef<AbortController | null>(null);

  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    const handler = setTimeout(async () => {
      // Aborta a requisição anterior se ainda estiver em trânsito
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
      abortControllerRef.current = new AbortController();

      try {
        const res = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
          signal: abortControllerRef.current.signal,
        });
        const data = await res.json();
        setResults(data);
      } catch (err: any) {
        if (err.name !== 'AbortError') console.error('Erro na busca', err);
      }
    }, delay);

    return () => clearTimeout(handler);
  }, [query, delay]);

  return results;
}
```

---

### 6. Paginação Obrigatória em Todas as APIs de Coleção
- **Anti-padrão**: `SELECT * FROM orders` retornando 45.000 registros em um JSON de 28MB que derruba o navegador do cliente.
- **Padrão Prod-Ready**: Toda rota que lista recursos DEVE ter paginação por cursor (preferencial para feeds e dados em tempo real) ou `page` + `limit` com um teto máximo rígido (ex.: `Math.min(limit, 100)`).
- **Cabeçalhos / Metadados**: Retornar `total`, `page`, `limit`, `totalPages`, ou `nextCursor`.

---

### 7. Distribuição de Arquivos Estáticos via CDN Global
- **Regra**: Servidores de aplicação (Node/Python/Go) nunca devem servir arquivos estáticos em produção.
- **Implementação**:
  - Edge CDN (Cloudflare, CloudFront, Fastly) na frente de todos os assets (`/static/`, `/_next/static/`, `/dist/`).
  - Hashing no nome do arquivo (ex: `app.9f8a3c.js`).
  - Compressão HTTP/3 e TLS 1.3 ativados na borda.

---

### 8. Filas Assíncronas em Segundo Plano para Tarefas >200ms
- **Regra**: Qualquer operação que leve mais de 200ms (envio de e-mail, geração de PDF/relatório, sincronização externa, IA complexa) NUNCA deve ser executada no ciclo síncrono da requisição HTTP do usuário.
- **Implementação**:
  - Enfileirar a tarefa em fila gerenciada (BullMQ no Redis, Celery, SQS, RabbitMQ).
  - Responder HTTP `202 Accepted` com `{ jobId, status: "pending" }`.
  - Notificar conclusão via WebSocket, SSE ou polling de status com intervalo progressivo.

---

### 9. Eliminação Total de Consultas N+1
- **Anti-padrão**: Buscar 50 posts e, em um loop, executar `SELECT * FROM users WHERE id = post.author_id`, gerando 51 consultas ao banco de dados.
- **Padrão Prod-Ready**:
  - No SQL: Usar `JOIN` ou subqueries agrupadas.
  - No ORM (Prisma/TypeORM/Drizzle): Usar `include: { author: true }` ou batching.
  - No GraphQL: DataLoader obrigatório para relacionamentos.

---

### 10. Compressão Brotli e Gzip no Edge / Proxy Reverso
- **Regra**: Respostas de texto (HTML, CSS, JS, JSON, SVG) devem ser compactadas.
- **Prioridade**: Brotli (`br`) como primeira escolha (compressão 15-20% mais eficiente que gzip), com fallback automático para Gzip (`gzip`) para navegadores legados.

---

### 11. Cache Estratégico com Redis (Cache-Aside Pattern)
- **Implementação**:
  1. Verificar existência da chave no Redis (`redis.get(key)`).
  2. Se existir (Cache Hit), retornar instantaneamente deserializado (< 2ms).
  3. Se não existir (Cache Miss), consultar o banco de dados.
  4. Gravar o resultado no Redis com TTL explícito (`SETEX key 300 value`).
  5. Invalidação baseada em eventos (ao criar/atualizar/deletar o recurso, remover ou atualizar a chave do cache).

---

### 12. Auditoria e Eliminação de Dependências Desnecessárias
- **Regras**:
  - Proibido importar bibliotecas gigantes para tarefas pontuais (ex: não importe Moment.js quando existe `Intl.DateTimeFormat` nativo; não importe Lodash inteiro quando um método JS nativo resolve).
  - Executar análise periódica com `bundle-analyzer`.
  - Tree-shaking verificado em builds de produção.

---

### 13. Timeouts Rígidos e Circuit Breakers em APIs Externas
- **Regra**: Toda chamada a APIs de terceiros (Gateways de pagamento, serviços de e-mail, provedores LLM) DEVE possuir um timeout estrito (máximo 3000ms a 5000ms).
- **Circuit Breaker**: Se a API externa falhar 5 vezes consecutivas, o circuit breaker abre temporariamente, retornando fallback imediato sem prender threads do servidor.

```ts
export async function fetchWithTimeout(url: string, options: RequestInit = {}, timeoutMs = 4000) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, { ...options, signal: controller.signal });
    clearTimeout(id);
    return response;
  } catch (error: any) {
    clearTimeout(id);
    if (error.name === 'AbortError') {
      throw new Error(`Tempo limite excedido (${timeoutMs}ms) ao conectar a ${url}`);
    }
    throw error;
  }
}
```

---

### 14. Streaming de Respostas de IA Token por Token (SSE)
- **Anti-padrão**: Esperar a LLM gerar 1000 tokens em 15 segundos em tela branca para depois cuspir o texto completo de uma vez.
- **Padrão Prod-Ready**: Utilizar streaming contínuo via Server-Sent Events (`text/event-stream`) ou ReadableStreams. O primeiro token deve surgir na tela em < 500ms, proporcionando percepção de velocidade imediata ao usuário.

---

### 15. Atualização Otimista da Interface (Optimistic UI)
- **Regra**: Em ações de alta frequência (curtir, favoritar, reordenar itens, marcar checkbox, alternar status), a interface DEVE atualizar o estado imediatamente na tela do usuário antes da resposta do servidor.
- **Rollback Garantido**: Se a API retornar erro de rede ou validação, reverter automaticamente para o estado original e exibir uma notificação (toast) discreta informando a falha.

---

### 16. Lazy Loading Nativo e Carregamento Sob Demanda
- **Regras**:
  - Imagens abaixo da linha de dobra recebem `loading="lazy"`.
  - Componentes pesados (gráficos complexos, editores de texto rico, modais pesados) carregados sob demanda apenas quando o usuário rolar até eles ou clicar no botão de abertura.
  - Uso de CSS `content-visibility: auto` para seções extensas de conteúdo fora do viewport.

---

### 17. Divisão de Código por Rota (Route-Based Code Splitting)
- **Regra**: O bundle JavaScript inicial do usuário deve conter APENAS o código estritamente necessário para renderizar a página solicitada.
- **Implementação**:
  - Imports dinâmicos (`import()`).
  - Separar bibliotecas pesadas de visualização de dados (Chart.js, Three.js, etc.) em chunks assíncronos isolados.

---

### 18. Virtualização de Listas e Tabelas Longas (>50 itens)
- **Anti-padrão**: Renderizar 1.000 nós DOM em uma tabela, consumindo centenas de megabytes de RAM e travando a rolagem do usuário.
- **Padrão Prod-Ready**: Virtualização de DOM (usando TanStack Virtual, react-window ou CSS content-visibility). Apenas os ~15-20 elementos visíveis na janela de visualização do usuário são montados no DOM real.

---

### 19. Projeção Estrita de Colunas SQL (Zero `SELECT *`)
- **Regra**: Proibido executar `SELECT *` em tabelas de produção.
- **Padrão**: Especificar explicitamente apenas as colunas que a interface realmente utiliza (`SELECT id, name, email, status FROM users`).
- **Impacto**: Economia massiva de I/O em disco, largura de banda de rede entre banco e aplicação e menor uso de memória heap.

---

### 20. Reutilização e Pooling de Conexões de Banco de Dados
- **Anti-padrão**: Abrir uma nova conexão TCP/TLS com o banco a cada requisição HTTP e fechá-la em seguida (custo de handshake de 50-100ms por request).
- **Padrão Prod-Ready**:
  - Pool de conexões ativo (PgBouncer, pool nativo do driver).
  - Parâmetros ajustados de `min_connections`, `max_connections` e `idleTimeout`.
  - Reutilização transparente de conexões abertas entre requisições.

---

### 21. Prevenção de Re-renderizações Desnecessárias no Frontend
- **Regras**:
  - Isolar estados de alta frequência (ex: posição do cursor, valor de input em digitação) em componentes folha.
  - Utilizar memoização consciente (`useMemo`, `useCallback`, `React.memo`) em cálculos pesados ou componentes com listas filhas.
  - Não passar objetos literais ou funções inline não memoizadas para componentes puros.

---

### 22. Headers de Cache Imutáveis no Navegador
- **Arquivos Estáticos Versionados** (com hash no nome):
  `Cache-Control: public, max-age=31536000, immutable`
- **Documentos HTML** (que referenciam os assets):
  `Cache-Control: public, no-cache, must-revalidate`
- **Impacto**: O navegador nunca mais gasta requisição de rede para baixar novamente um JS/CSS já baixado.

---

### 23. Arquitetura 100% Stateless e Escalonamento Horizontal
- **Regra**: Instâncias da aplicação não devem armazenar estado local em memória ou disco (sessões, uploads temporários, contadores).
- **Armazenamento de Estado**:
  - Sessões e tokens em Redis ou JWT criptografado.
  - Uploads direto para Object Storage (S3, R2, GCS) via signed URLs.
  - Aplicação pronta para rodar com 1, 10 ou 100 réplicas atrás de um Load Balancer sem quebras de consistência.

---

### 24. Fontes Web Auto-Hospedadas, Subsettadas e Otimizadas
- **Regra**: Proibido carregar fontes bloqueantes de CDNs externas que atrasam o primeiro render de texto.
- **Implementação**:
  - Hospedar os arquivos `.woff2` no próprio domínio/CDN.
  - Declarar `font-display: swap` em todo `@font-face`.
  - Definir pilha de fallback de fontes do sistema nativas (evita FOUT/FOIT abrupto).
  - Subsetting: remover glifos e idiomas não utilizados, mantendo o peso da fonte < 30KB por estilo.

---

### 25. Monitoramento RUM de Velocidade Real e Testes em Redes Lentas
- **RUM (Real User Monitoring)**: Coletar e emitir métricas reais de Web Vitals dos usuários em tempo real para painel de observabilidade.
- **Testes Obrigatórios de Pré-Deploy**:
  - Simulação de conexão móvel **Fast 3G / Slow 4G** (1.6 Mbps download, 750 Kbps upload, 150ms RTT).
  - Throttling de CPU de 4x para validar usabilidade em aparelhos Android de entrada.
  - Garantir que a aplicação permaneça funcional, informativa e rápida mesmo sob condições adversas.
