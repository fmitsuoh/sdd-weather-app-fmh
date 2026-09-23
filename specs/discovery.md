# Discovery — Aplicação de Previsão do Tempo

## Contexto

A empresa solicitou o desenvolvimento de uma aplicação web de previsão do tempo com foco em usabilidade, rapidez de acesso e adaptação a dispositivos móveis. O produto deve permitir que os usuários pesquisem cidades, visualizem o clima atual e a previsão para os próximos 5 dias, alternem entre unidades de temperatura em Celsius e Fahrenheit e tenham uma experiência funcional em smartphones e tablets.

O contexto de negócio aponta para uma solução de informação prática e de alto valor para o usuário final, com baixo custo de operação e uso de dados abertos, sem necessidade de autenticação. A aplicação deve priorizar clareza na apresentação das condições climáticas, simplicidade na interação e confiabilidade na exibição de dados meteorológicos.

Além disso, a solução deve considerar o uso em ambientes móveis, em que a navegação é curta, a atenção do usuário é limitada e o conteúdo precisa ser legível em telas pequenas. O produto deve ser perc eptível, rápido e intuitivo, reduzindo a fricção de busca e a necessidade de múltiplos passos para obter a informação desejada.

## Personas

1. **Mariana, a Commuter Urbana**
   - **Objetivo principal:** Saber rapidamente se vai chover antes de sair de casa para o trabalho, para decidir se leva guarda-chuva ou casaco.
   - **Contexto de uso:** Mobile, em movimento (parada de ônibus, dentro de casa antes de sair), sessões muito curtas (< 30 segundos).
   - **Métrica de sucesso:** Consegue ver a condição atual e a chance de chuva do dia em no máximo 2 toques, sem precisar esperar carregamento perceptível.

2. **Rafael, o Planejador de Viagens/Fim de Semana**
   - **Objetivo principal:** Comparar a previsão dos próximos 5 dias entre a cidade onde mora e o destino que pretende visitar, para decidir programação e roupas.
   - **Contexto de uso:** Mistura de desktop (planejamento em casa, mais tempo disponível) e mobile (consulta rápida antes de sair), pesquisando múltiplas cidades na mesma sessão.
   - **Métrica de sucesso:** Consegue buscar e alternar entre 2-3 cidades diferentes e visualizar a previsão de 5 dias de cada uma sem confusão sobre qual cidade está sendo exibida.

3. **Dona Célia, Usuária Não Técnica (baixa afinidade digital)**
   - **Objetivo principal:** Entender de forma simples e direta "vai fazer calor ou frio hoje", sem se perder em termos técnicos ou telas complexas.
   - **Contexto de uso:** Mobile, tela pequena, pouca familiaridade com apps — tende a usar fonte grande do sistema e pode depender de leitor de tela ou zoom.
   - **Métrica de sucesso:** Consegue ler a temperatura atual e a condição do clima com clareza (contraste e tamanho de fonte adequados), sem precisar de ajuda de terceiros para interpretar a tela.

## Requisitos Funcionais

1. Busca de cidades
   - O usuário deve poder pesquisar cidades por nome.
   - A aplicação deve disponibilizar uma forma clara de entrada de texto para busca.
   - A busca deve retornar resultados relevantes de localidades compatíveis com a consulta.
   - Quando não houver resultado, a interface deve indicar ausência de dados de forma amigável.

2. Visualização do clima atual
   - A aplicação deve exibir o clima atual da cidade selecionada.
   - Deve apresentar pelo menos os principais indicadores meteorológicos, como temperatura e condição do clima.
   - O usuário deve ter visibilidade imediata do estado do tempo em uma tela principal ou painel contextual.

3. Previsão de 5 dias
   - A aplicação deve apresentar uma previsão de 5 dias consecutivos para a cidade selecionada.
   - Os dados devem ser organizados de forma legível, com periodicidade diária.
   - A interface deve facilitar a comparação entre os dias e a leitura das variações esperadas.

4. Alternância entre Celsius e Fahrenheit
   - O usuário deve poder alternar entre as unidades de temperatura Celsius e Fahrenheit.
   - A troca de unidade deve refletir imediatamente na visualização da temperatura.
   - A funcionalidade deve ser acessível e fácil de localizar na interface.

5. Experiência básica de feedback ao usuário
   - A aplicação deve indicar estados de carregamento enquanto busca e processa os dados.
   - Deve exibir mensagens de erro quando houver falha na consulta ou ausência de dados.
   - O fluxo deve ser orientado por estados bem definidos: carregando, sucesso e vazio/erro.

## Requisitos Não-Funcionais

1. Usabilidade
   - A interface deve ser intuitiva e de fácil compreensão para usuários sem treinamento específico.
   - A navegação deve exigir poucos passos para buscar e visualizar o clima.

2. Responsividade
   - A aplicação deve funcionar em diferentes tamanhos de tela e resoluções, priorizando mobile first.
   - A experiência deve preservar legibilidade e usabilidade em celulares, tablets e desktops.
   - Os elementos devem manter legibilidade e funcionalidade em layouts responsivos, com interação por toque, espaçamento adequado e navegação simplificada em mobile.

3. Performance
   - A busca e a renderização das informações devem ocorrer em tempo adequado para uso real.
   - A aplicação deve apresentar feedback rápido para o usuário durante carregamento.

4. Confiabilidade dos dados
   - As informações meteorológicas devem ser apresentadas com consistência e sem contradições.
   - A aplicação deve tratar falhas de API ou indisponibilidade de dados com mensagens claras.

5. Acessibilidade
   - A interface deve ser utilizável por pessoas com diferentes necessidades, incluindo leitura clara, contraste adequado e elementos marcados semanticamente.
   - Controles como busca e alternador de unidade devem ser acessíveis via teclado e por tecnologias assistivas.

6. Manutenibilidade
   - A arquitetura deve favorecer separação entre apresentação, lógica de negócios e acesso a dados.
   - O código deve ser modular para facilitar manutenção e evolução futura.

7. Disponibilidade
   - A aplicação deve definir uma meta de uptime (ex.: 99,5%) e uma estratégia de degradação graciosa quando a API externa estiver indisponível (cache, retry, mensagem de fallback).

8. Segurança
   - Toda comunicação deve usar HTTPS.
   - A entrada de busca deve ser sanitizada para evitar injeção de conteúdo malicioso.
   - Nenhuma chave ou segredo deve ser exposto no código client-side.

9. Compatibilidade de navegadores e dispositivos
   - A aplicação deve suportar as últimas versões estáveis de Chrome, Firefox e Safari, incluindo suas variantes mobile (Chrome Android, Safari iOS).

10. Observabilidade
    - Erros de integração com a API e falhas de renderização devem ser registrados (logging) para permitir diagnóstico proativo de indisponibilidade.

## Riscos

| # | Risco | Tipo | Probabilidade | Impacto | Estratégia de Mitigação |
|---|-------|------|----------------|---------|--------------------------|
| 1 | Indisponibilidade ou instabilidade da API Open-Meteo | Técnico | Média | Alto — sem dados, o app perde sua função principal | Implementar cache de última consulta bem-sucedida (com timestamp), retry com backoff exponencial, e mensagem de fallback amigável |
| 2 | Ambiguidade na busca de cidades (nomes duplicados) | Produto | Alta | Médio — usuário pode ver o clima da cidade errada sem perceber | Exibir país/estado nos resultados de busca e exigir seleção explícita antes de mostrar dados |
| 3 | Erros de conversão Celsius/Fahrenheit | Técnico | Baixa | Alto — inconsistência de dados mina a confiança na aplicação inteira | Centralizar lógica de conversão em função pura única (`src/lib/temperature.ts`) com testes unitários cobrindo casos extremos |
| 4 | Layout quebrado em mobile (responsividade) | Técnico | Média | Alto — maioria do público-alvo é mobile, conforme discovery | Definir breakpoints formais, testar em dispositivos reais/emulados, incluir testes E2E de viewport no Playwright |
| 5 | Ausência de tratamento para estados de erro/vazio | Produto | Média | Médio — usuário fica sem feedback em falhas, parecendo app quebrado | Definir timeout de requisição e mensagens diferenciadas por tipo de falha (rede, cidade inexistente, API indisponível) |
| 6 | Rate limiting / bloqueio da API pública por uso excessivo | Técnico | Média | Alto — bloqueio afeta todos os usuários simultaneamente | Aplicar debounce na busca e possivelmente cache client-side de consultas recentes |
| 7 | Escopo funcional insuficiente (métricas climáticas incompletas) | Produto | Alta | Médio — pode gerar retrabalho de UI/API após feedback do stakeholder | Fechar a Pergunta em Aberto #3 (métricas obrigatórias) antes da fase de Plan |
| 8 | Falta de meta de acessibilidade (nível WCAG) | Produto | Média | Médio — risco de exclusão de usuários e possível não conformidade legal | Definir nível AA como padrão mínimo e validar com testes automatizados (axe) antes do merge |
| 9 | Falta de definição de disponibilidade/SLA | Produto | Baixa | Médio — expectativa de negócio desalinhada com esforço de engenharia | Confirmar com stakeholder se existe SLA formal; se não, tratar como "best effort" documentado |
| 10 | Divergência de interpretação da previsão de 5 dias (dia atual incluso?) | Produto | Média | Médio — pode gerar retrabalho visual e falha em critério de aceite | Fechar a Pergunta em Aberto #10 antes de iniciar o Plan Agent |
| 11 | Suporte a navegadores não definido (legados vs. modernos) | Técnico | Baixa | Baixo — pode gerar polyfills desnecessários ou bugs em browsers não testados | Fixar como "últimas 2 versões" dos principais browsers e validar via `browserslist`/CI |
| 12 | Persistência de preferência de unidade não definida | Produto | Baixa | Baixo — inconsistência de UX entre sessões | Decidir uso de `localStorage` para lembrar unidade escolhida |

## Decisões

1. **Fonte de dados: Open-Meteo (sem API key)**
   - **Justificativa:** API pública, gratuita e sem necessidade de autenticação, alinhada à Suposição 2 (dados abertos, sem login).
   - **Resolve:** Confirma a Suposição 2 como decisão formal e reduz o Risco 1 (dependência de fonte externa) ao fixar qual provedor será integrado, permitindo que o Plan Agent defina contratos de API concretos.

2. **"5 dias" = hoje + 4 dias**
   - **Justificativa:** Interpretação mais comum em apps de clima (dia atual + previsão dos próximos dias) e evita ambiguidade sobre se a previsão começa amanhã.
   - **Resolve:** Pergunta em Aberto #10 (se a previsão inclui o dia atual) e o Risco 10 associado (divergência de interpretação da previsão de 5 dias).

3. **Unidade padrão: Celsius**
   - **Justificativa:** Simplifica o estado inicial da aplicação e atende à maioria dos usuários de mercados que usam o sistema métrico.
   - **Resolve:** Elimina a ambiguidade sobre qual unidade exibir antes de qualquer interação do usuário com o toggle Celsius/Fahrenheit (Requisito Funcional 4).

4. **Sem autenticação e sem persistência de servidor**
   - **Justificativa:** Mantém o escopo do MVP enxuto, sem backend próprio nem armazenamento de dados de usuário, reduzindo custo de operação.
   - **Resolve:** Pergunta em Aberto #6 (histórico/favoritos) — fica definido que não há persistência de cidades consultadas no MVP. Também reforça as Suposições 2 e 3.

5. **Idioma da UI: pt-BR**
   - **Justificativa:** Público-alvo inicial é de usuários de língua portuguesa, simplificando o escopo de i18n do MVP.
   - **Resolve:** Parcialmente a Pergunta em Aberto #5 (público-alvo genérico ou foco regional) — fixa o idioma da interface, mesmo que a origem dos usuários não seja exclusivamente Brasil.

## Perguntas em Aberto

1. A busca de cidades deve priorizar resultados automáticos enquanto digita, ou apenas após a confirmação do usuário?
2. A aplicação deve permitir seleção explícita de país/estado quando houver cidades com o mesmo nome?
3. A aplicação deve exibir apenas temperatura, ou também outras métricas como umidade, vento e precipitação?
4. A aplicação precisa suportar geolocalização automática do usuário, ou apenas busca manual por cidade?
5. O público-alvo é genérico, ou há foco específico em usuários de um país/região?
6. Existe um requisito de persistência de cidades consultadas, como histórico ou favoritos?
7. A aplicação deve seguir um design dark-mode de glassmorphism, ou existe outra diretriz visual?
8. Quais são as metas de disponibilidade e tempo de resposta esperados para a aplicação?
9. Há limite máximo de resultados de busca e tolerância a erros de digitação (fuzzy search)?
10. A previsão de 5 dias inclui o dia atual ou começa no dia seguinte, e cada dia mostra só min/max ou também períodos (manhã/tarde/noite)?
11. A preferência de unidade (Celsius/Fahrenheit) deve persistir entre sessões do navegador?
12. Quais breakpoints e dispositivos de referência devem ser usados para validar responsividade?
13. Qual o timeout máximo de uma busca antes de exibir erro, e as mensagens devem ser diferenciadas por tipo de falha (rede, cidade inexistente, API indisponível)?
14. Quais são os alvos numéricos de performance (ex.: tempo de carregamento, tempo de resposta da busca)?
15. Em caso de falha da API externa, o app deve exibir o último dado em cache (com timestamp) ou apenas mensagem de erro?
16. Qual o nível mínimo de conformidade WCAG exigido (A, AA ou AAA)?
17. É necessário aplicar debounce/rate limiting nas chamadas de busca para evitar bloqueio da API pública por excesso de uso?
18. Há requisito de suporte a navegadores legados, ou apenas as últimas versões dos principais browsers?

## Suposições

1. A solução será desenvolvida como aplicação web responsiva, não como app nativo.
2. A API de dados meteorológicos será fornecida por serviço externo sem necessidade de autenticação do usuário.
3. O escopo principal é o consumo de dados em tempo real e a apresentação de informações climáticas, sem registro e login.
4. A busca por cidade será suficiente para atender ao objetivo principal do produto.
5. O usuário conseguirá entender e usar a alternância entre Celsius e Fahrenheit sem treinamento adicional.
6. A experiência móvel será tratada como prioridade de design, mas o produto também deve funcionar em desktop.
7. Os dados de previsão serão exibidos em intervalos diários para os próximos 5 dias, conforme o briefing.
8. A aplicação deve apresentar mensagens amigáveis para ausência de resultados e falhas de integração.
