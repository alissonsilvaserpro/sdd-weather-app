# Plano Técnico — Weather App

Este plano deriva exclusivamente da especificação em `specs/weather-app-spec.md` e define contratos para o MVP. A implementação será uma SPA client-side, sem backend próprio, autenticação ou persistência de histórico.

## Architecture

A aplicação usará arquitetura em camadas, com fluxo unidirecional e responsabilidades pequenas:

- **Apresentação (`src/components/`)**: componentes React focados em renderização e interação, recebendo dados e callbacks por props.
- **Orquestração (`src/hooks/`)**: `useWeather` coordena busca, seleção da cidade, carregamento, erro, retry e dados da cidade ativa.
- **Integração (`src/services/`)**: `weatherService` encapsula `fetch`, timeout, construção de URLs e validação/mapeamento das respostas da Open-Meteo.
- **Domínio puro (`src/lib/`)**: conversão de temperatura, formatação de datas, tradução de códigos meteorológicos e validações sem efeitos colaterais.
- **Contratos (`src/types/`)**: tipos compartilhados entre serviço, hook e componentes.

A UI não fará chamadas HTTP nem conhecerá o formato bruto da Open-Meteo. A troca de unidade ocorrerá localmente sobre os dados normalizados, sem novo request.

## Tech Stack

| Camada | Tecnologia | Decisão |
| --- | --- | --- |
| Linguagem | TypeScript strict | Contratos explícitos e detecção de respostas inválidas em compilação. |
| UI | React 19 + Vite | SPA simples, rápida e adequada para deploy estático. |
| Estilo | Tailwind CSS | Responsividade mobile-first e uso consistente do tema existente. |
| Dados | `fetch` nativo + Open-Meteo | Sem API key e sem dependência adicional para o MVP. |
| Testes unitários | Vitest + Testing Library + `user-event` | Testes rápidos de funções, componentes e fluxos de interação. |
| Testes E2E | Playwright | Validação do fluxo completo e de viewports móveis, com APIs interceptadas. |
| Qualidade | Biome | Lint, formatação e checagens do código existente. |

Compatibilidade assumida: navegadores evergreen atuais em desktop e mobile, com suporte a `fetch`, `AbortController` e APIs web padrão. O layout será validado a partir de 320px, conforme a especificação.

## Project Structure

```text
src/
├── components/
│   ├── SearchBar.tsx
│   ├── CityResults.tsx
│   ├── CurrentWeather.tsx
│   ├── ForecastList.tsx
│   ├── ForecastCard.tsx
│   ├── UnitToggle.tsx
│   └── states/
│       ├── LoadingState.tsx
│       ├── ErrorState.tsx
│       └── EmptyState.tsx
├── hooks/
│   └── useWeather.ts
├── lib/
│   ├── temperature.ts
│   ├── weatherCodes.ts
│   └── format.ts
├── services/
│   └── weatherService.ts
├── types/
│   └── weather.ts
├── App.tsx
├── main.tsx
└── index.css

tests/
├── lib/
├── services/
├── components/
└── e2e/
```

Cada componente deve ter uma responsabilidade clara. O `App` compõe o fluxo e mantém apenas o estado de preferência de unidade ou delega o estado operacional ao hook. Não será criada uma camada de repositório, cache global ou gerenciamento de estado externo sem necessidade demonstrada.

## Data Model

Os dados do domínio ficam sempre normalizados em Celsius. Valores recebidos da API devem ser convertidos para números finitos e validados antes de formar `WeatherData`.

```ts
export type Unit = 'celsius' | 'fahrenheit';

export type WeatherStatus = 'idle' | 'loading' | 'success' | 'empty' | 'error';

export type LoadingOperation = 'city-search' | 'weather-fetch';

export interface City {
  id: number;
  name: string;
  country?: string;
  countryCode?: string;
  admin1?: string;
  timezone?: string;
  latitude: number;
  longitude: number;
}

export interface CurrentWeather {
  time: string;
  temperatureC: number;
  weatherCode: number;
  humidity: number;
  windSpeedKmh: number;
  precipitationMm: number;
  pressureHpa: number;
}

export interface ForecastDay {
  date: string;
  minTemperatureC: number;
  maxTemperatureC: number;
  weatherCode: number;
  precipitationProbability: number;
}

export interface WeatherData {
  city: City;
  current: CurrentWeather;
  forecast: [ForecastDay, ForecastDay, ForecastDay, ForecastDay, ForecastDay];
}

export interface WeatherError {
  code: 'validation' | 'network' | 'api' | 'timeout' | 'partial-response' | 'unknown';
  message: string;
  retryable: boolean;
}

export interface WeatherViewState {
  status: WeatherStatus;
  loadingOperation: LoadingOperation | null;
  query: string;
  results: City[];
  activeCity: City | null;
  data: WeatherData | null;
  error: WeatherError | null;
}
```

Contratos de serviço:

```ts
export interface WeatherService {
  searchCities(query: string, signal?: AbortSignal): Promise<City[]>;
  getWeather(city: City, signal?: AbortSignal): Promise<WeatherData>;
}
```

`forecast` é uma tupla de exatamente cinco dias, tornando o requisito de AC-04.1 verificável também no domínio. A unidade persistida e exibida usa `Unit`; a API recebe unidades métricas e o domínio não armazena Fahrenheit.

## Data Flow

1. `SearchBar` normaliza espaços nas bordas e envia o termo ao `useWeather`.
2. O hook ignora vazio, bloqueia termos com menos de dois caracteres e, para uma entrada válida, define `loading` antes do primeiro `await` (para atender ao limite percebido de 300 ms) e chama `weatherService.searchCities`.
3. O serviço consulta o endpoint de geocoding e devolve `City[]` já mapeado e ordenado pela ordem retornada pela API.
4. `CityResults` exibe os resultados e solicita seleção explícita de um `City`, preservando distinções de cidade, região e país.
5. A seleção chama `weatherService.getWeather(city)`. O hook limpa os dados anteriores durante a nova operação e mantém a cidade selecionada identificável.
6. O serviço consulta o forecast, valida os campos obrigatórios e transforma arrays paralelos em `WeatherData` com cinco itens.
7. Em sucesso, `CurrentWeather` e `ForecastList` recebem o mesmo `WeatherData` da cidade ativa.
8. `UnitToggle` altera apenas `unit`. Componentes derivam valores com `convertTemperature`, sem repetir a busca.
9. Uma nova busca ou seleção cancela a operação anterior com `AbortController`; resultados obsoletos não podem substituir a cidade ativa mais recente.

```mermaid
flowchart TD
  A[Input de busca] --> B{Entrada válida?}
  B -->|Vazia: ignorar| C[Fim sem request]
  B -->|Menos de 2 caracteres| D[Hook de estado: validation]
  D --> E[UI: mensagem curta]
  B -->|Sim| F[Weather service: geocoding]
  F -->|Falha de rede ou API| G[Hook de estado: error]
  G --> H[UI: erro + tentar novamente]
  F -->|Lista vazia| I[Hook de estado: empty]
  I --> J[UI: nenhuma cidade encontrada]
  F -->|City[]| K[UI: seleção de cidade]
  K --> L[Weather service: forecast]
  L -->|Timeout, rede ou API| M[Hook de estado: error]
  M --> H
  L -->|Resposta parcial ou inválida| N[Hook de estado: error]
  N --> H
  L -->|WeatherData válido| O[Hook de estado: success]
  O --> P[UI: clima atual + previsão de 5 dias]
  Q[UnitToggle] --> R[Conversão derivada Celsius/Fahrenheit]
  R --> P
```

## External APIs

### Geocoding Open-Meteo

```text
GET https://geocoding-api.open-meteo.com/v1/search
  ?name={encodedQuery}
  &count=5
  &language=pt
  &format=json
```

Parâmetros relevantes:

- `name`: termo de busca normalizado e codificado para URL.
- `count=5`: limita a lista aos resultados necessários para seleção.
- `language=pt`: prioriza nomes localizados em português quando disponíveis.
- `format=json`: solicita resposta JSON.

Exemplo resumido de resposta:

```json
{
  "results": [
    {
      "id": 3448439,
      "name": "São Paulo",
      "latitude": -23.55,
      "longitude": -46.63,
      "elevation": 760,
      "feature_code": "PPLA",
      "country_code": "BR",
      "country": "Brasil",
      "admin1": "São Paulo",
      "timezone": "America/Sao_Paulo"
    }
  ],
  "generationtime_ms": 0.4
}
```

O serviço deve fazer `encodeURIComponent` do termo, verificar `response.ok` e validar que `results` é um array. Cada item precisa conter `id`, `name`, `latitude` e `longitude`. O mapeamento para `City` é direto: `country_code` → `countryCode`, `admin1` → `admin1` e `timezone` → `timezone`; campos como `elevation` e `feature_code` não fazem parte do modelo do MVP. Ausência de `results` ou lista vazia representa sucesso com `City[]` vazio, não erro de rede.

### Forecast Open-Meteo

```text
GET https://api.open-meteo.com/v1/forecast
  ?latitude={latitude}
  &longitude={longitude}
  &current=temperature_2m,relative_humidity_2m,wind_speed_10m,surface_pressure,precipitation,weather_code
  &daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max
  &forecast_days=5
  &timezone=auto
  &temperature_unit=celsius
  &wind_speed_unit=kmh
  &precipitation_unit=mm
```

Parâmetros relevantes:

- `latitude` e `longitude`: coordenadas da cidade selecionada.
- `current`: campos atuais necessários para `CurrentWeather`.
- `daily`: séries necessárias para `ForecastDay`.
- `forecast_days=5`: solicita o dia atual e os quatro dias seguintes.
- `timezone=auto`: retorna horários e datas no fuso da coordenada.
- `temperature_unit=celsius`, `wind_speed_unit=kmh` e `precipitation_unit=mm`: tornam explícitas as unidades armazenadas no domínio.

Exemplo resumido de resposta:

```json
{
  "latitude": -23.55,
  "longitude": -46.63,
  "timezone": "America/Sao_Paulo",
  "current": {
    "time": "2026-09-23T12:00",
    "temperature_2m": 22.4,
    "relative_humidity_2m": 68,
    "wind_speed_10m": 14.2,
    "surface_pressure": 1014.8,
    "precipitation": 0.0,
    "weather_code": 2
  },
  "daily": {
    "time": ["2026-09-23", "2026-09-24", "2026-09-25", "2026-09-26", "2026-09-27"],
    "weather_code": [2, 61, 3, 80, 1],
    "temperature_2m_max": [25.1, 23.4, 24.8, 22.7, 26.0],
    "temperature_2m_min": [17.8, 18.2, 17.5, 16.9, 17.1],
    "precipitation_probability_max": [10, 70, 20, 60, 5]
  }
}
```

Mapeamento para o modelo:

| Resposta Open-Meteo | Modelo | Regra |
| --- | --- | --- |
| `current.time` | `CurrentWeather.time` | Preservar como string ISO/local retornada pela API. |
| `current.temperature_2m` | `temperatureC` | A requisição usa unidades métricas; armazenar em Celsius. |
| `current.relative_humidity_2m` | `humidity` | Percentual relativo. |
| `current.wind_speed_10m` | `windSpeedKmh` | Velocidade em km/h da unidade padrão solicitada. |
| `current.surface_pressure` | `pressureHpa` | Pressão em hPa da unidade padrão solicitada. |
| `current.precipitation` | `precipitationMm` | Precipitação atual em milímetros. |
| `current.weather_code` | `weatherCode` | Código WMO, traduzido posteriormente na UI. |
| `daily.time[i]` | `ForecastDay.date` | Data do item `i`. |
| `daily.temperature_2m_min[i]` | `minTemperatureC` | Mínima do item `i`, em Celsius. |
| `daily.temperature_2m_max[i]` | `maxTemperatureC` | Máxima do item `i`, em Celsius. |
| `daily.weather_code[i]` | `weatherCode` | Código WMO do item `i`. |
| `daily.precipitation_probability_max[i]` | `precipitationProbability` | Probabilidade máxima do item `i`, em percentual. |

O serviço deve validar `current` e `daily`, inclusive a presença de todos os campos acima, valores numéricos finitos e cinco posições em cada série diária. Os arrays são combinados pelo mesmo índice para formar os cinco `ForecastDay` e então associados à `City` selecionada, produzindo `WeatherData`. Qualquer campo obrigatório ausente, não numérico ou com tamanho diferente de cinco resulta em `partial-response`; não haverá preenchimento silencioso com dados incompletos. O `weatherCode` será traduzido apenas na camada de apresentação por uma tabela local baseada em WMO.

As requisições usarão timeout via `AbortController`. O serviço não manterá cache ou credenciais, pois o escopo não exige esses recursos.

## State Management

Será usado um hook local, sem Redux, Zustand, React Query ou outro gerenciador externo.

- `useWeather` mantém `WeatherViewState`, cidade ativa e referências dos controladores de abortamento.
- `unit` começa em `celsius` e é carregada do `localStorage` com validação; valores ausentes ou inválidos voltam para Celsius.
- A preferência é salva apenas quando muda, sob a chave versionada `weather-app:unit`.
- A máquina de estados operacional é explícita: `idle`, `loading`, `success`, `empty` e `error`.
- `loadingOperation` distingue `city-search` (geocoding) de `weather-fetch` (clima da cidade selecionada), sem criar estados adicionais na UI.
- `idle` representa a tela inicial sem consulta concluída; `loading` bloqueia a ação redundante correspondente e exibe indicador visível.
- `success` exige `data` válido; `empty` representa apenas uma busca sem cidades; `error` exige `WeatherError` e permite nova tentativa.
- O retry repete a última operação válida: a última busca de cidades ou o carregamento da cidade selecionada.
- Ao iniciar uma busca válida, o hook define `loading` imediatamente, limpa `data` e `results` anteriores e só atualiza `activeCity` após o forecast da nova seleção concluir com sucesso. Assim, o estado `empty` nunca exibe clima de uma cidade anterior, enquanto a cidade ativa permanece identificável durante a sessão.

A conversão é derivada durante a renderização. `WeatherData` mantém todas as temperaturas em Celsius; `lib/temperature.ts` recebe o valor Celsius e a `Unit` ativa, aplica `C = celsius` ou `F = celsius * 9 / 5 + 32`, arredonda para exibição e devolve o valor formatado. O componente atual e cada `ForecastCard` usam a mesma função, portanto a troca de unidade altera somente a apresentação e nunca dispara novo request.

## Error Handling

- Entradas vazias são ignoradas sem requisição.
- Entradas com menos de dois caracteres geram erro de validação curto e acessível.
- Falha de conexão, DNS ou `fetch` rejeitado gera `code: 'network'`, com mensagem em pt-BR e retry habilitado.
- Resposta HTTP não-OK ou erro explícito no payload da Open-Meteo gera `code: 'api'`; o usuário recebe uma mensagem genérica e pode tentar novamente.
- O timeout usa `AbortController` e gera `code: 'timeout'`, com retry habilitado e sem bloquear novas tentativas após o encerramento da operação.
- Campos obrigatórios ausentes, tipos inválidos ou arrays diários com menos de cinco itens geram `code: 'partial-response'`; a aplicação descarta a resposta inteira e não renderiza dados misturados ou incompletos.
- `AbortError` causado por uma nova operação é tratado como cancelamento esperado, sem exibir erro transitório.
- Dados anteriores são removidos ao iniciar uma nova seleção para não exibir uma cidade errada durante o carregamento ou após erro.
- Busca válida sem resultados renderiza `EmptyState` com o termo normalizado e orientação para tentar outra cidade.
- A busca válida sem resultados mantém `activeCity` apenas como referência da sessão, mas mantém `data` nulo e não renderiza seus blocos meteorológicos.
- Erros de `localStorage` não impedem o uso: a unidade permanece em memória e a preferência simplesmente não é persistida.
- `ErrorState` deve ter mensagem legível, ação de nova tentativa e foco/announcement adequado para leitor de tela.
- Falhas relevantes devem usar `console.error` em desenvolvimento, sem expor detalhes técnicos como mensagem principal ao usuário.

## Testing Strategy

Os testes serão derivados dos critérios de aceite e não dependerão da rede real. Vitest cobre unidades isoladas e componentes; Playwright cobre o comportamento integrado no navegador.

### Vitest

- **Funções puras (`lib/`)**: conversão Celsius/Fahrenheit nos dois sentidos, arredondamento, formatação de datas, tradução dos códigos WMO e validação de consultas. Incluir limites, valores negativos e precisão esperada.
- **Serviços (`services/`)**: mockar `globalThis.fetch` e testar geocoding e forecast separadamente. Cobrir URL/parâmetros, resposta válida, lista vazia, HTTP não-OK, erro de rede, timeout/abort, JSON inválido, campos obrigatórios ausentes, valores não numéricos e arrays diários com tamanho diferente de cinco.
- **Hook de orquestração (`hooks/`)**: validar transições `idle → loading → success`, `loading → empty` e `loading → error`, limpeza de dados anteriores, retry, cancelamento de request obsoleto e preservação da cidade ativa correta.
- **Componentes (`components/`)**: renderizar estados `loading`, `error`, `empty` e `success`; verificar mensagens em pt-BR, inclusive `Nenhuma cidade encontrada para <termo>`, seleção de cidade, envio de busca, botão desabilitado durante loading e troca de unidade sem nova chamada ao serviço.
- **Persistência e acessibilidade**: testar leitura/escrita de `localStorage`, fallback para Celsius quando o valor é inválido, labels, nomes acessíveis, foco visível e navegação por teclado usando Testing Library e `user-event`.

Os testes de serviço recebem payloads fixos e os componentes recebem props controladas. Assim, falhas de rede e mudanças de horário não tornam a suíte instável.

### Playwright

As rotas da Open-Meteo serão interceptadas com `page.route`, usando respostas determinísticas. Os fluxos principais são:

- busca válida → lista de cidades → seleção → clima atual e previsão de exatamente cinco dias;
- troca de Celsius para Fahrenheit, verificando que os valores mudam sem uma nova requisição de forecast;
- busca sem resultados, com `EmptyState` e termo pesquisado;
- falha de rede/API, mensagem amigável e nova tentativa bem-sucedida;
- operação com teclado, foco visível e nomes acessíveis dos controles;
- viewport mobile de 320px e 375px, verificando ausência de scroll horizontal, controles operáveis e cards de previsão legíveis.

Playwright não deve testar a disponibilidade real da Open-Meteo; a integração real é responsabilidade dos testes do serviço e de uma verificação manual/diagnóstica separada.

### Verificação de entrega

Executar `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e` antes de considerar o MVP pronto.

## Risks & Trade-offs

| Decisão | Alternativa considerada | Trade-off e justificativa |
| --- | --- | --- |
| Hook local para estado | Redux, Zustand ou React Query | Menos dependências e cerimônia. O escopo tem uma tela e poucas transições; uma biblioteca seria reavaliada se surgirem cache, múltiplas telas ou sincronização complexa. |
| Sem cache de forecast | React Query ou cache próprio | Uma nova seleção refaz a consulta, mas mantém comportamento previsível e implementação pequena. O custo é latência e chamadas repetidas. |
| `fetch` nativo | Axios | Evita dependência adicional, pois timeout, abortamento e parsing JSON já podem ser encapsulados no serviço. |
| Domínio armazenado em Celsius | Armazenar Celsius e Fahrenheit | Evita estado duplicado e inconsistência; a conversão na renderização tem custo desprezível. |
| Mocks determinísticos no E2E | Testar contra a Open-Meteo real | Mocks evitam flakiness, lentidão e dependência de disponibilidade externa. A cobertura de contrato fica nos testes de serviço. |
| Validar e descartar payload parcial | Renderizar campos disponíveis com fallback `—` | Rejeitar a resposta protege contra dados misturados e atende à spec, mas reduz informação disponível quando a API retorna algo incompleto. |
| Sem backend próprio | Proxy/serverless para a API | Deploy estático e ausência de credenciais são mais simples no MVP; um backend seria considerado para cache, observabilidade ou controle de limites. |
| Compatibilidade evergreen | Polyfills e suporte a navegadores legados | Reduz complexidade e bundle. A cobertura prioriza os dispositivos atuais e larguras mínimas da especificação. |

Riscos operacionais adicionais: a Open-Meteo pode falhar ou mudar o contrato, e nomes de cidades podem ser ambíguos. A mitigação é concentrar a integração em um serviço validado, usar timeout, permitir retry e exigir seleção explícita com país/região visíveis. O tema Tailwind existente também deve permanecer uma preocupação visual; ele não deve introduzir lógica de negócio nos componentes.
