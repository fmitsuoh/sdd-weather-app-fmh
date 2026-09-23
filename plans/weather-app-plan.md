# Plano Técnico — Weather App

> Fonte da verdade: `specs/weather-app-spec.md`. Este plano documenta a
> arquitetura e as decisões técnicas — inclusive as que já estão refletidas na
> implementação atual em `src/` — e rastreia cada decisão de volta a um
> requisito da spec.

## 1. Architecture Overview

Aplicação client-side pura (SPA), sem backend próprio. Camadas:

```
UI (components/)
  ↓ usa
Hooks (hooks/useWeather.ts) — orquestra estado e chamadas
  ↓ usa
Services (services/weatherService.ts) — acesso à API Open-Meteo
  ↓ usa
Lib (lib/temperature.ts, weatherCodes.ts, format.ts) — funções puras
  ↓ tipadas por
Types (types/weather.ts) — contratos de domínio
```

Não há estado global (Redux/Context) — o estado vive no hook `useWeather` e é
passado por props, mantendo a Manutenibilidade (NFR6).

## 2. Tech Stack

| Tecnologia | Justificativa |
|---|---|
| React + Vite | Já definido no `copilot-instructions.md`; build rápido, SPA simples. |
| TypeScript (strict) | Contratos de dados explícitos (FR2/FR3 dependem de shape correto da API). |
| Tailwind CSS | Estilo utilitário, tema dark glassmorphism (resolve parcialmente Open Question #6 da spec — já implementado em `App.tsx`/`index.css`). |
| Vitest + Testing Library | Testes unitários de lib/hooks/componentes (NFR "testável"). |
| Playwright | Testes E2E dos fluxos de busca → seleção → clima (FR1–FR5). |
| Biome | Lint + formatação, já configurado em `biome.json`. |
| Sem gerenciador de estado externo | Escopo pequeno (5 FRs); `useState`/hook customizado é suficiente — evita over-engineering. |

## 3. Project Structure

```
src/
  components/          # 1 componente por arquivo (react.instructions.md)
    SearchBar.tsx       # FR1 — input + submit, desabilitado durante loading
    CurrentWeather.tsx  # FR2 — temperatura + condição atual
    ForecastList.tsx    # FR3 — lista de 5 dias
    ForecastCard.tsx    # FR3 — item individual de previsão
    UnitToggle.tsx       # FR4 — alternância C/F, acessível (aria-pressed)
    states/
      LoadingState.tsx   # FR5 — indicador de carregamento
      ErrorState.tsx     # FR5 — mensagem de erro + retry
      EmptyState.tsx     # FR5 — estado vazio (inicial e "sem resultados")
  hooks/
    useWeather.ts        # Orquestra busca → seleção → carregamento (FR1, FR1.1, FR5)
  services/
    weatherService.ts     # Chamadas HTTP à Open-Meteo (geocoding + forecast)
  lib/
    temperature.ts         # Conversão C/F pura e testável (FR4, mitiga Risco 3)
    weatherCodes.ts         # Mapeamento weather_code → label/ícone pt-BR (FR2, FR3)
    format.ts               # Formatação de datas/labels auxiliares
  types/
    weather.ts               # City, CurrentWeather, ForecastDay, WeatherData, Unit
  App.tsx                     # Composição da tela e máquina de estados
```

## 4. Data Model

```typescript
type Unit = 'celsius' | 'fahrenheit';

interface City {
  id: number;
  name: string;
  country: string;
  admin1?: string;       // estado/região — resolve ambiguidade de homônimos (FR1.1, Risco 2)
  latitude: number;
  longitude: number;
}

interface CurrentWeather {
  temperature: number;    // sempre em °C internamente (FR4)
  weatherCode: number;     // WMO code — mapeado por weatherCodes.ts (FR2)
  humidity: number;
  windSpeed: number;
  pressure: number;
  precipitation: number;
  time: string;
}

interface ForecastDay {
  date: string;
  min: number;
  max: number;
  weatherCode: number;
  precipitationProbability: number;
}

interface WeatherData {
  city: City;
  current: CurrentWeather;
  forecast: ForecastDay[];   // até 5 itens (FR3)
}
```

**Decisão de arquitetura:** temperaturas são sempre armazenadas em Celsius e
convertidas apenas na camada de apresentação (`formatTemperature`), para que
a troca de unidade (FR4) nunca dispare um novo request.

## 5. Data Flow

1. Usuário digita e confirma busca em `SearchBar` (FR1) → `useWeather.search(name)`.
2. Hook chama `searchCities(name)` → geocoding API. Resultado vazio → status `empty` (FR1, FR5).
3. Primeira cidade retornada é carregada automaticamente; lista completa fica disponível para o usuário trocar (FR1.1).
4. `useWeather` chama `getWeather(city)` → forecast API → popula `WeatherData`.
5. `App.tsx` renderiza `CurrentWeather` + `ForecastList` com base no `status` (`idle | loading | success | error | empty`) — máquina de estados cobre FR5 integralmente.
6. Troca de unidade (`UnitToggle`) é local ao `App.tsx`, sem novo request — apenas reformatação via `lib/temperature.ts` (FR4).

## 6. External APIs

**Geocoding:** `GET https://geocoding-api.open-meteo.com/v1/search`
- Parâmetros: `name`, `count=5` (limite fixo — resolve parcialmente Open Question #8), `language=pt`, `format=json`.

**Forecast:** `GET https://api.open-meteo.com/v1/forecast`
- Parâmetros: `latitude`, `longitude`, `current=temperature_2m,relative_humidity_2m,wind_speed_10m,surface_pressure,precipitation,weather_code`, `daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max`, `forecast_days=5`, `timezone=auto`.
- `timezone=auto` resolve a Open Question #17 da spec: o "dia atual" é calculado no fuso horário da cidade pesquisada (retornado pela própria API), não no fuso do dispositivo.

Nenhuma API key é necessária (Decisão 1 do `discovery.md`).

## 7. State Management

- Estado local via `useState`/hook customizado (`useWeather`), sem lib externa.
- Estado de unidade (`Unit`) vive em `App.tsx`; não persiste entre reloads (Open Question #9 permanece em aberto — comportamento atual: sempre reseta para Celsius).
- Máquina de estados explícita: `idle | loading | success | error | empty`, evitando estados combinados inconsistentes (ex.: loading + error simultâneos).

## 8. Error Handling Strategy

| Situação | Tratamento | Requisito |
|---|---|---|
| Timeout de requisição | `AbortController` com limite de 10s (`REQUEST_TIMEOUT_MS`), convertido em `WeatherServiceError` | Resolve Open Question #11 na implementação atual |
| Falha de rede / API 5xx | `WeatherServiceError` com mensagem amigável, status `error`, botão de retry (`ErrorState`) | FR5 |
| Geocoding sem resultados | Lista vazia → status `empty`, sem lançar erro | FR1, FR5 |
| Resposta de forecast incompleta (`current`/`daily` ausentes) | Lança `WeatherServiceError` tratado como falha genérica | FR2/FR3 edge case |
| Input vazio/whitespace | `SearchBar` bloqueia submit (`disabled={!value.trim()}`); `useWeather.search` também short-circuita | FR1 |

## 9. Testing Strategy

- **Vitest + Testing Library:** `lib/temperature.ts`, `lib/weatherCodes.ts`, `lib/format.ts` (funções puras), componentes isolados (`SearchBar`, `UnitToggle`), e `weatherService` com `fetch` mockado (casos de sucesso, erro, timeout, resposta parcial).
- **Playwright:** fluxo E2E completo — busca → seleção → visualização de clima atual e previsão → troca de unidade → estados de erro/vazio (mock de rede via `page.route`).
- Cada Acceptance Criteria (Given/When/Then) da spec deve mapear a pelo menos um teste automatizado, conforme a Traceability Matrix da spec.

## 10. Risks & Trade-offs

| Risco/Trade-off | Decisão tomada | Trade-off aceito |
|---|---|---|
| Sem cache/fallback de última resposta bem-sucedida (Risco 1 da spec) | Não implementado nesta versão — falha de API sempre mostra erro, mesmo se já houve sucesso antes | Simplicidade > resiliência; revisitar se uptime da Open-Meteo for um problema real |
| Sem debounce/rate limiting na busca (Risco 6, Open Question #15) | Busca só dispara em submit explícito do formulário, não em cada tecla — já reduz risco de rate limit sem debounce adicional | Menor responsividade de UX (sem autocomplete) em troca de simplicidade e menor risco de bloqueio de API |
| Unidade de temperatura não persiste entre reloads (Open Question #9) | Estado local em `App.tsx`, resetado a cada carregamento | Simplicidade > persistência; fácil de adicionar `localStorage` depois sem mudar contratos |
| Limite fixo de 5 resultados de geocoding (Open Question #8) | Hardcoded `count=5` no service | Evita paginação/scroll infinito no MVP; pode não cobrir cidades muito ambíguas com >5 homônimos |
