# Especificação — Weather App

> Gerado a partir de `specs/discovery.md`. Ambiguidades não resolvidas foram
> mantidas em **Open Questions**, sem respostas inventadas.

## Overview

O Weather App é uma aplicação web responsiva de previsão do tempo, sem
autenticação e sem persistência de servidor, que permite ao usuário buscar
cidades por nome, visualizar o clima atual e a previsão para os próximos 5
dias (hoje + 4 dias), e alternar entre Celsius e Fahrenheit (padrão: Celsius).
Os dados são obtidos da API pública Open-Meteo (sem necessidade de chave de
API). A interface é em pt-BR, prioriza uso mobile-first e deve tratar
explicitamente os estados de carregamento, erro e resultado vazio.

## Functional Requirements

### FR1 — Busca de cidades

O usuário pode pesquisar cidades por nome através de um campo de texto claro.
A busca retorna localidades compatíveis com a consulta via geocoding; o
número máximo de resultados exibidos ainda não está definido (Open Question
#8). Entradas são sanitizadas (trim + escape de HTML) antes do envio.

**Acceptance Criteria:**
- **Given** uma resposta mockada de geocoding contendo N cidades (N ≥ 1), **when** o usuário confirma a busca, **then** a aplicação renderiza exatamente N itens de resultado, cada um com nome, país e estado/região (quando disponível).
- **Given** uma resposta mockada de geocoding vazia, **when** o usuário confirma a busca, **then** a aplicação exibe o estado vazio "nenhuma cidade encontrada para '{query}'" sem expor erro técnico.
- **Given** o campo de busca contém apenas espaços em branco, **when** o usuário confirma a busca, **then** nenhuma requisição é disparada e a tela de busca permanece inalterada.
- **Given** o campo de busca contém HTML ou caracteres de controle, **when** o usuário confirma a busca, **then** o valor enviado à API é sanitizado (sem tags/scripts) antes da requisição.
- **Given** o campo de busca está focado via teclado, **when** o usuário pressiona `Enter`, **then** a busca é disparada da mesma forma que ao clicar no botão de busca.

### FR1.1 — Seleção de resultado de busca

Após a busca retornar um ou mais resultados, o usuário seleciona explicitamente
uma cidade para prosseguir à visualização do clima.

**Acceptance Criteria:**
- **Given** a lista de resultados de busca está visível, **when** o usuário seleciona (clique ou `Enter` com foco no item) um resultado, **then** a aplicação navega para a tela de clima daquela cidade específica.
- **Given** a lista de resultados contém múltiplas cidades com o mesmo nome, **when** o usuário seleciona uma delas, **then** a aplicação usa as coordenadas do item selecionado (não apenas o nome) para buscar o clima, evitando ambiguidade.

### FR2 — Visualização do clima atual

A aplicação exibe o clima atual da cidade selecionada, incluindo ao menos
temperatura e condição do clima. A tradução do código meteorológico da API
para texto segue o mapeamento fixo definido em `src/lib/weatherCodes.ts`.

**Acceptance Criteria:**
- **Given** uma resposta mockada de clima atual com um weather code específico (ex.: código 0), **when** a requisição retorna com sucesso, **then** a tela principal exibe a temperatura atual e o texto mapeado para aquele código (ex.: código 0 → "céu limpo").
- **Given** a resposta da API não contém o campo de weather code, **when** a aplicação renderiza a tela, **then** ela exibe a temperatura disponível e omite a condição do clima, sem quebrar a renderização.

### FR3 — Previsão de 5 dias

A aplicação apresenta a previsão para 5 dias consecutivos, iniciando no dia
atual (hoje + 4 dias seguintes), organizada com periodicidade diária. O "dia
atual" é calculado no fuso horário retornado pela API para a cidade
pesquisada (ver Open Question #17).

**Acceptance Criteria:**
- **Given** uma resposta mockada de previsão com 5 dias de dados, **when** a requisição retorna com sucesso, **then** a aplicação exibe exatamente 5 itens de previsão, na mesma ordem cronológica recebida.
- **Given** a previsão foi carregada, **when** o usuário visualiza qualquer um dos 5 dias, **then** o item exibe temperatura mínima e máxima daquele dia.
- **Given** uma resposta mockada de previsão com menos de 5 dias de dados, **when** a aplicação renderiza a lista, **then** ela exibe apenas os dias recebidos, sem gerar itens vazios ou lançar erro.

### FR4 — Alternância entre Celsius e Fahrenheit

O usuário pode alternar a unidade de temperatura exibida via um controle
acessível por teclado. A unidade padrão ao carregar a aplicação é Celsius e
é mantida ao navegar entre buscas na mesma sessão (persistência entre
recarregamentos de página ainda não definida — Open Question #9).

**Acceptance Criteria:**
- **Given** a aplicação é carregada pela primeira vez, **when** nenhuma interação de unidade ainda ocorreu, **then** todas as temperaturas são exibidas em Celsius.
- **Given** a aplicação está exibindo temperaturas em Celsius, **when** o usuário aciona o alternador de unidade (clique ou `Enter`/`Space` com foco no controle), **then** todas as temperaturas visíveis (atual e previsão) são recalculadas e exibidas em Fahrenheit sem recarregar a página.
- **Given** o usuário selecionou Fahrenheit e depois busca uma nova cidade, **when** o clima da nova cidade é carregado, **then** as temperaturas são exibidas em Fahrenheit (a unidade escolhida persiste entre buscas na mesma sessão).

### FR5 — Estados de feedback ao usuário

A aplicação indica estados de carregamento, erro e vazio de forma clara
durante a busca e o processamento de dados, incluindo o estado inicial antes
de qualquer busca (ver `components/states/EmptyState.tsx`).

**Acceptance Criteria:**
- **Given** a aplicação acabou de carregar e nenhuma busca foi feita, **when** o usuário visualiza a tela, **then** a aplicação exibe o estado vazio inicial (sem indicador de carregamento ou erro).
- **Given** o usuário disparou uma busca ou requisição de clima, **when** a resposta ainda não retornou, **then** a interface exibe um indicador de carregamento visível.
- **Given** o usuário disparou uma requisição, **when** ela falha por erro de rede ou API indisponível, **then** a interface exibe a mensagem de erro amigável definida e mantém o campo de busca editável e o botão de busca habilitado para nova tentativa.
- **Given** o usuário realizou uma busca, **when** a busca não retorna nenhum resultado, **then** a interface exibe um estado vazio visualmente distinto do estado de erro.

## User Stories

1. **Como Rafael (planejador de viagens),** quero pesquisar cidades pelo nome, **para** encontrar rapidamente o destino que pretendo visitar. *(FR1 — Busca de cidades)*
2. **Como Rafael (planejador de viagens),** quero selecionar a cidade correta entre resultados homônimos, **para** garantir que o clima exibido é do destino certo. *(FR1.1 — Seleção de resultado de busca)*
3. **Como Mariana (commuter urbana),** quero ver a condição atual e a temperatura da minha cidade, **para** decidir se levo guarda-chuva ou casaco antes de sair de casa. *(FR2 — Visualização do clima atual)*
4. **Como Rafael (planejador de viagens),** quero visualizar a previsão dos próximos 5 dias de uma cidade, **para** planejar minha programação e escolher as roupas certas. *(FR3 — Previsão de 5 dias)*
5. **Como Dona Célia (usuária com baixa afinidade digital),** quero alternar entre Celsius e Fahrenheit com um único toque, **para** visualizar a temperatura na unidade que já conheço, sem confusão. *(FR4 — Alternância entre Celsius e Fahrenheit)*
6. **Como Dona Célia (usuária com baixa afinidade digital),** quero ser avisada quando a busca estiver carregando ou falhar, **para** entender o que está acontecendo sem achar que o aplicativo travou. *(FR5 — Estados de feedback ao usuário)*
7. **Como Mariana (commuter urbana),** quero receber uma mensagem clara quando minha busca não encontrar a cidade, **para** corrigir rapidamente o nome digitado sem perder tempo. *(FR1 — Busca de cidades / FR5 — Estados de feedback ao usuário)*

## Acceptance Criteria

Os critérios de aceite de cada requisito funcional estão descritos junto ao
respectivo item em **Functional Requirements** (FR1–FR5). Adicionalmente:

- Todos os critérios de aceite devem ser verificáveis via testes automatizados (Vitest para lógica/unidade, Playwright para fluxo E2E).
- Nenhum critério de aceite depende de dados mockados em produção; a integração real é com a API Open-Meteo.

## Traceability Matrix

| User Story | Requisito(s) | Acceptance Criteria (resumo) | NFRs relevantes |
|---|---|---|---|
| 1. Rafael pesquisa cidades pelo nome | FR1 | Renderiza N resultados; estado vazio sem correspondência; ignora busca vazia; sanitiza input; busca via `Enter` | NFR1 Usabilidade, NFR5 Acessibilidade, NFR8 Segurança |
| 2. Rafael seleciona cidade correta entre homônimas | FR1.1 | Navega para tela de clima ao selecionar; usa coordenadas do item (não o nome) | NFR1 Usabilidade, NFR4 Confiabilidade dos dados |
| 3. Mariana vê condição atual e temperatura | FR2 | Exibe temperatura + condição mapeada por weather code; omite condição se ausente sem quebrar | NFR4 Confiabilidade dos dados, NFR1 Usabilidade |
| 4. Rafael visualiza previsão de 5 dias | FR3 | Exibe exatamente 5 itens na ordem recebida; cada item com min/max; lida com menos de 5 dias sem erro | NFR4 Confiabilidade dos dados, NFR1 Usabilidade |
| 5. Dona Célia alterna Celsius/Fahrenheit | FR4 | Padrão Celsius; recalcula em Fahrenheit ao acionar (mouse/teclado); mantém unidade entre buscas na sessão | NFR5 Acessibilidade, NFR1 Usabilidade |
| 6. Dona Célia é avisada de loading/erro | FR5 | Estado vazio inicial; indicador de loading; mensagem de erro amigável com busca reeditável; estado vazio distinto do erro | NFR4 Confiabilidade dos dados, NFR7 Disponibilidade, NFR10 Observabilidade |
| 7. Mariana recebe mensagem clara quando busca falha | FR1, FR5 | Estado vazio amigável sem erro técnico (FR1); mensagem de erro amigável mantendo busca editável (FR5) | NFR1 Usabilidade, NFR4 Confiabilidade dos dados |

> Todas as linhas cobrem, direta ou indiretamente, **NFR2 Responsividade**,
> **NFR3 Performance**, **NFR6 Manutenibilidade** e **NFR9 Compatibilidade de
> navegadores**, que são transversais a toda a interface e não a uma única
> User Story.

## Non-Functional Requirements

1. **Usabilidade** — Interface intuitiva, navegação com poucos passos para buscar e visualizar o clima.
2. **Responsividade** — Layout mobile-first, funcional em celulares, tablets e desktops; breakpoints específicos ainda não definidos (ver Open Questions).
3. **Performance** — Busca e renderização devem ocorrer em tempo adequado para uso real. Não testável até que as metas numéricas sejam fechadas (Open Question #12).
4. **Confiabilidade dos dados** — Informações consistentes, sem contradições; falhas de API tratadas com a mensagem de erro amigável definida em FR5.
5. **Acessibilidade** — Interface utilizável por pessoas com diferentes necessidades; controles acessíveis via teclado e tecnologias assistivas; nível WCAG-alvo ainda não definido (ver Open Questions).
6. **Manutenibilidade** — Separação entre apresentação, lógica de negócio e acesso a dados; código modular.
7. **Disponibilidade** — Estratégia de degradação graciosa quando a API externa estiver indisponível (cache/retry/fallback); meta de uptime formal ainda não definida.
8. **Segurança** — Comunicação via HTTPS; sanitização da entrada de busca; nenhuma chave/segredo exposto no client.
9. **Compatibilidade de navegadores** — Suporte às últimas versões estáveis de Chrome, Firefox e Safari (desktop e mobile); suporte a navegadores legados ainda não definido.
10. **Observabilidade** — Registro (logging) de erros de integração com a API e falhas de renderização.

## Edge Cases

| Edge Case | Comportamento esperado |
|---|---|
| Input vazio ou apenas espaços em branco | Nenhuma requisição é disparada; tela de busca permanece inalterada (FR1). |
| Caracteres especiais/HTML na busca | Entrada sanitizada (trim + escape) antes do envio; se o resultado sanitizado for vazio, tratado como input vazio (FR1). |
| Cidade inexistente / geocoding sem resultados | Lista de geocoding vazia → estado vazio amigável "nenhuma cidade encontrada para '{query}'", distinto do estado de erro (FR1). |
| Cidade com nome duplicado em países/regiões diferentes | Resultados exibem nome, país e estado/região; seleção usa coordenadas do item, não o nome (FR1.1). |
| Timeout de requisição | Requisição abortada após o limite definido (Open Question #11); tratada como falha de API (FR5). |
| API externa indisponível ou erro 5xx | Mensagem de erro amigável definida em FR5; campo de busca permanece editável para nova tentativa. |
| Resposta parcial da API (campo ausente) | Weather code ausente → condição do clima omitida sem quebrar a renderização (FR2); dia de previsão sem min/max não é exibido como item vazio (FR3). |
| Perda de conexão de rede durante o uso | Tratada como falha de API (FR5); nenhuma tela em branco ou travamento. |
| Temperaturas extremas na conversão C/F | Conversão usa a fórmula padrão sem arredondamento adicional além do já especificado em `src/lib/temperature.ts`; sem limites artificiais de valor. |
| Viewport muito pequeno (< 360px de largura) | Layout mantém legibilidade e nenhum elemento interativo é cortado ou sobreposto (NFR 2 — breakpoints a definir em Open Question #10). |

## Assumptions

1. A solução é uma aplicação web responsiva, não um app nativo.
2. A fonte de dados meteorológicos é a API pública Open-Meteo, sem autenticação de usuário.
3. Não há registro, login ou persistência de dados de usuário no servidor.
4. A busca por cidade é suficiente para atender ao objetivo principal do produto (sem geolocalização automática, pendente de confirmação — ver Open Questions).
5. O usuário compreende a alternância entre Celsius e Fahrenheit sem treinamento adicional.
6. A experiência mobile é prioridade de design, mas o produto também funciona em desktop.
7. A previsão é exibida em intervalos diários para os próximos 5 dias, começando no dia atual.
8. A interface é apresentada em pt-BR.

## Risks

| # | Risco | Probabilidade | Impacto | Mitigação |
|---|-------|----------------|---------|-----------|
| 1 | Indisponibilidade ou instabilidade da API Open-Meteo | Média | Alto | Cache da última consulta bem-sucedida, retry com backoff, mensagem de fallback |
| 2 | Ambiguidade na busca de cidades (nomes duplicados) | Alta | Médio | Exibir país/estado nos resultados; exigir seleção explícita |
| 3 | Erros de conversão Celsius/Fahrenheit | Baixa | Alto | Centralizar conversão em função pura testada unitariamente |
| 4 | Layout quebrado em mobile | Média | Alto | Definir breakpoints formais e testar em viewports reais via Playwright |
| 5 | Ausência de tratamento para estados de erro/vazio | Média | Médio | Endereçado por FR5; falta apenas fechar o timeout exato (Open Question #11) |
| 6 | Rate limiting da API pública por uso excessivo | Média | Alto | Debounce na busca e cache client-side de consultas recentes |
| 7 | Escopo funcional insuficiente (métricas incompletas) | Alta | Médio | Fechar Open Question sobre métricas obrigatórias antes do Plan |
| 8 | Falta de meta de acessibilidade (nível WCAG) | Média | Médio | Fixar nível AA como padrão mínimo e validar com testes automatizados |

## Out of Scope

- Autenticação, cadastro ou login de usuário (`discovery.md` → Decisão 4).
- Persistência de dados no servidor (histórico de buscas, favoritos) (`discovery.md` → Decisão 4).
- Aplicativo nativo (iOS/Android).
- Suporte a idiomas além de pt-BR nesta versão (`discovery.md` → Decisão 5).
- Geolocalização automática do usuário (pendente de decisão — ver Open Question #4; se aprovada, requer novo requisito funcional).

## Open Questions

1. A busca de cidades deve priorizar resultados automáticos enquanto digita, ou apenas após a confirmação do usuário?
2. A aplicação deve permitir seleção explícita de país/estado quando houver cidades com o mesmo nome?
3. A aplicação deve exibir apenas temperatura, ou também outras métricas como umidade, vento e precipitação?
4. A aplicação precisa suportar geolocalização automática do usuário, ou apenas busca manual por cidade?
5. Além do idioma pt-BR já decidido, há foco específico de público em um país/região que deva influenciar unidades padrão (ex.: milhas vs. km) ou formatos de data?
6. A aplicação deve seguir um design dark-mode de glassmorphism (já sugerido em `copilot-instructions.md`), ou existe outra diretriz visual a confirmar formalmente?
7. Quais são as metas de disponibilidade e tempo de resposta esperados (números concretos)?
8. Há limite máximo de resultados de busca e tolerância a erros de digitação (fuzzy search)?
9. A preferência de unidade (Celsius/Fahrenheit) deve persistir entre sessões via `localStorage`?
10. Quais breakpoints e dispositivos de referência devem ser usados para validar responsividade?
11. Qual o timeout máximo de uma busca antes de exibir erro, e as mensagens devem ser diferenciadas por tipo de falha (rede, cidade inexistente, API indisponível)?
12. Quais são os alvos numéricos de performance (ex.: tempo de carregamento, tempo de resposta da busca)?
13. Em caso de falha da API externa, o app deve exibir o último dado em cache (com timestamp) ou apenas mensagem de erro?
14. Qual o nível mínimo de conformidade WCAG exigido (A, AA ou AAA)?
15. É necessário aplicar debounce/rate limiting nas chamadas de busca para evitar bloqueio da API pública por excesso de uso?
16. Há requisito de suporte a navegadores legados, ou apenas as últimas versões dos principais browsers?
17. O "dia atual" da previsão de 5 dias (FR3) deve ser calculado no fuso horário da cidade pesquisada ou no fuso horário do dispositivo do usuário?
