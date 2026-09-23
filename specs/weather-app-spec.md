# Weather App — Especificação do Produto (MVP)

## 1. Objetivo

A aplicação permite que o usuário consulte o clima de uma cidade, visualize o estado do tempo atual e veja a previsão dos próximos 5 dias. O produto deve priorizar velocidade, clareza e usabilidade em telas pequenas, sem exigir autenticação ou backend próprio.

A solução usa Open-Meteo como fonte de dados, com interface em português do Brasil, temperatura padrão em Celsius e suporte à conversão para Fahrenheit.

## 2. Escopo do MVP

### Inclui
- busca por nome de cidade;
- seleção de cidade entre resultados retornados;
- exibição do clima atual;
- exibição da previsão dos próximos 5 dias;
- alternância entre °C e °F;
- estados de carregamento, erro e vazio;
- responsividade mobile-first;
- acessibilidade básica.

### Exclui do MVP
- autenticação;
- histórico de buscas persistido no servidor;
- geolocalização automática;
- favoritos ou perfil do usuário;
- alertas meteorológicos severos;
- mapas interativos;
- PWA como requisito obrigatório.

## 3. Personas e uso principal

### Maria, 34 anos
Usuária diária que quer saber rapidamente se precisa levar guarda-chuva e qual roupa usar.

### Carlos, 28 anos
Viajante que compara previsões de cidades antes de decidir destino ou deslocamento.

### Ana, 26 anos
Usuária mobile-first que busca informar-se em poucos segundos sem navegar em menus complexos.

### Histórias de usuário

- US-01: Como Maria, usuária de rotina, quero buscar uma cidade para saber rapidamente se preciso levar guarda-chuva ou escolher uma roupa adequada para o dia.
- US-02: Como Carlos, viajante em planejamento, quero consultar a previsão de 5 dias para comparar condições climáticas de diferentes destinos antes de me deslocar.
- US-03: Como Ana, usuária mobile-first, quero consultar o clima atual em poucos segundos para tomar decisões rápidas durante o dia, sem precisar navegar em uma interface complicada.
- US-04: Como qualquer usuário, quero alternar entre Celsius e Fahrenheit para visualizar a temperatura na unidade que eu entendo melhor.
- US-05: Como usuário da aplicação, quero receber uma mensagem clara quando a busca falha para entender o problema e tentar novamente sem fricção.
- US-06: Como usuário, quero pesquisar o nome de uma cidade e receber resultados relevantes para escolher a localização correta antes de consultar o clima.
- US-07: Como usuário mobile, quero visualizar o clima atual e a previsão em um layout adaptado para telas pequenas para consumir as informações com conforto.

### Tabela de rastreabilidade

| User Story | Critérios de aceite relevantes | NFRs relevantes |
|---|---|---|
| US-01 | AC-01.4, AC-01.5, AC-03.1, AC-03.2, AC-07.1, AC-07.3 | NFR-01, NFR-03, NFR-06 |
| US-02 | AC-04.1, AC-04.2, AC-04.3, AC-04.4 | NFR-01, NFR-03 |
| US-03 | AC-06.1, AC-06.2, AC-08.1, AC-08.2, AC-08.3 | NFR-02, NFR-04 |
| US-04 | AC-05.1, AC-05.2, AC-05.3, AC-05.4 | NFR-01, NFR-05 |
| US-05 | AC-07.1, AC-07.2, AC-07.3, AC-06.3 | NFR-03, NFR-06 |
| US-06 | AC-01.1, AC-01.2, AC-01.3, AC-01.4, AC-01.5 | NFR-01, NFR-06 |
| US-07 | AC-08.1, AC-08.2, AC-08.3, AC-09.1, AC-09.2, AC-09.3 | NFR-02, NFR-04 |

## 4. Requisitos funcionais

### FR-01 — Busca por cidade
O sistema deve permitir que o usuário envie um termo de busca para encontrar localidades relevantes.

Critérios de aceite:
- AC-01.1: Dado que o usuário digitou um termo com espaços extras, Quando ele envia a busca, Então o sistema deve normalizar a entrada antes da consulta.
- AC-01.2: Dado que o usuário envia uma string vazia ou apenas espaços, Quando o formulário é submetido, Então o sistema deve ignorar a ação e não disparar requisição.
- AC-01.3: Dado que o termo tem menos de 2 caracteres, Quando o usuário envia a busca, Então o sistema deve bloquear a requisição e exibir mensagem de validação curta.
- AC-01.4: Dado que a API retorna uma lista de cidades, Quando a busca termina com sucesso, Então a interface deve exibir os resultados em ordem relevante e legível.
- AC-01.5: Dado que a API retorna zero resultados, Quando a busca termina, Então a interface deve exibir estado vazio com mensagem: “Nenhuma cidade encontrada para <termo>”.

### FR-02 — Seleção de cidade
O sistema deve permitir que o usuário escolha uma das localidades retornadas pela busca.

Critérios de aceite:
- AC-02.1: Dado que a busca retornou múltiplos resultados, Quando o usuário seleciona uma opção, Então o sistema deve carregar os dados daquela cidade como cidade ativa.
- AC-02.2: Dado que o usuário alterou a cidade selecionada, Quando a nova busca for concluída, Então todos os blocos de clima e previsão devem refletir a nova cidade ativa.
- AC-02.3: Dado que a cidade selecionada muda, Quando os dados forem carregados, Então o nome da cidade deve permanecer visível na interface durante a sessão atual.

### FR-03 — Clima atual
O sistema deve exibir os dados meteorológicos atuais da cidade ativa.

Critérios de aceite:
- AC-03.1: Dado que uma cidade foi selecionada com sucesso, Quando os dados forem carregados, Então a tela deve mostrar a temperatura atual na unidade ativa.
- AC-03.2: Dado que os dados da cidade ativa são válidos, Quando a UI for renderizada, Então devem aparecer, no mínimo: temperatura, umidade, vento, precipitação e pressão atmosférica.
- AC-03.3: Dado que a cidade ativa existe, Quando a UI renderiza o clima atual, Então o nome da cidade e, quando disponível, o país ou região devem aparecer.
- AC-03.4: Dado que a condição meteorológica possui código ou descrição válido, Quando a tela for renderizada, Então o usuário deve ver um ícone ou texto que represente o estado do clima.

### FR-04 — Previsão de 5 dias
O sistema deve apresentar a previsão para os próximos 5 dias a partir do dia atual.

Critérios de aceite:
- AC-04.1: Dado que a cidade ativa foi carregada com sucesso, Quando a previsão for exibida, Então a lista deve conter exatamente 5 itens.
- AC-04.2: Dado que cada item da previsão possui dados válidos, Quando a lista for renderizada, Então cada item deve mostrar data, temperatura mínima, temperatura máxima e probabilidade de precipitação.
- AC-04.3: Dado que o usuário seleciona uma nova cidade, Quando a consulta retorna, Então a previsão exibida deve pertencer apenas à cidade ativa.
- AC-04.4: Dado que a previsão não estiver disponível ou vier incompleta, Quando o processamento finalizar, Então o sistema deve exibir erro sem quebrar o layout.

### FR-05 — Unidade de temperatura
O sistema deve permitir alternar entre Celsius e Fahrenheit sem recarregar os dados da API.

Critérios de aceite:
- AC-05.1: Dado que a aplicação foi aberta pela primeira vez, Quando a interface renderiza, Então a unidade padrão deve ser Celsius.
- AC-05.2: Dado que o usuário clica no toggle de unidade, Quando a ação é confirmada, Então a interface deve alternar entre °C e °F.
- AC-05.3: Dado que a temperatura foi exibida em Celsius, Quando o usuário troca para Fahrenheit, Então todos os valores visíveis devem ser convertidos para a nova unidade sem nova busca.
- AC-05.4: Dado que o usuário revisita a aplicação, Quando a página abre novamente, Então a última unidade escolhida deve ser reaplicada a partir do armazenamento local.

### FR-06 — Indicador de carregamento
O sistema deve avisar quando uma busca ou carregamento está em andamento.

Critérios de aceite:
- AC-06.1: Dado que uma consulta está sendo processada, Quando a operação ainda não concluiu, Então a interface deve exibir indicador de carregamento visível.
- AC-06.2: Dado que a busca está em andamento, Quando o usuário tenta disparar outra busca, Então o botão principal deve estar desabilitado para evitar duplicidade de requisições.
- AC-06.3: Dado que a operação termina com sucesso ou falha, Quando o resultado chega, Então o indicador deve desaparecer.

### FR-07 — Tratamento de erro
O sistema deve tratar falhas de rede, respostas vazias e respostas incompletas sem quebrar a experiência.

Critérios de aceite:
- AC-07.1: Dado que a requisição falha por timeout, indisponibilidade ou erro de rede, Quando o usuário tenta buscar uma cidade, Então a aplicação deve exibir mensagem de erro em português e permitir nova tentativa.
- AC-07.2: Dado que a API retorna resposta incompleta, Quando o processamento ocorrer, Então o sistema deve apresentar erro e não renderizar dados inconsistentes.
- AC-07.3: Dado que não existem resultados para a busca, Quando a operação finalizar, Então a aplicação deve exibir estado vazio com instrução clara ao usuário.

### FR-08 — Responsividade mobile
A interface deve funcionar para uso em smartphones e continuar acessível em desktop.

Critérios de aceite:
- AC-08.1: Dado que a viewport tem 375px de largura, Quando a página renderiza, Então não deve haver scroll horizontal desnecessário.
- AC-08.2: Dado que o usuário acessa em mobile, Quando ele usa o campo de busca e o botão de unidade, Então esses elementos devem permanecer operáveis e visíveis sem sobreposição.
- AC-08.3: Dado que a previsão de 5 dias é exibida em mobile, Quando os cards renderizam, Então o texto deve manter legibilidade e a estrutura visual deve permanecer intacta.

### FR-09 — Acessibilidade básica
O produto deve suportar uso com teclado e semântica acessível.

Critérios de aceite:
- AC-09.1: Dado que o usuário navega com teclado, Quando ele entra em um controle interativo, Então o foco deve ser visível.
- AC-09.2: Dado que um campo de busca ou botão está presente, Quando o usuário busca entendê-lo, Então deve existir label ou texto acessível que descreva a ação.
- AC-09.3: Dado que a interface é usada por leitor de tela, Quando os controles são acessados, Então o nome acessível deve ser suficiente para identificar a ação.

## 5. Regras de negócio

- A unidade padrão da interface é Celsius.
- O idioma da interface é pt-BR.
- A previsão exibida é de 5 dias, incluindo o dia atual e os próximos 4 dias.
- A cidade ativa é a última cidade selecionada com sucesso.
- Submissões vazias são ignoradas.
- Qualquer falha de rede ou resposta inválida deve manter a aplicação em estado estável.
- A preferência de unidade deve ser persistida localmente no navegador.

## 6. Estados da interface

### Estado inicial
- sem cidade selecionada;
- exibe campo de busca e estado vazio ou placeholder do clima.

### Estado de carregamento
- mostra spinner ou indicação equivalente;
- bloqueia novas ações redundantes.

### Estado de sucesso
- exibe clima atual e previsão de 5 dias para a cidade ativa.

### Estado de erro
- exibe mensagem amigável em português;
- mantém a possibilidade de tentar novamente.

### Estado vazio
- exibe mensagem específica para consulta sem resultados;
- não mostra dados de cidade anterior.

## 7. Requisitos não funcionais

### NFR-01 — Performance
- a busca deve responder em tempo percebido como imediato em rede estável;
- o estado de carregamento deve aparecer em no máximo 300 ms após a ação do usuário;
- a tela não deve congelar durante a busca.

### NFR-02 — Responsividade
- a interface deve funcionar em smartphones e desktops;
- o layout deve evitar overflow horizontal em larguras a partir de 320px;
- os elementos principais devem permanecer acessíveis em telas pequenas.

### NFR-03 — Confiabilidade
- falhas de rede e API não devem quebrar a interface;
- a aplicação deve recuperar com nova tentativa sem reinicialização manual;
- respostas incompletas devem ser rejeitadas com fallback seguro.

### NFR-04 — Acessibilidade
- o contraste mínimo deve respeitar padrão de acessibilidade de 4.5:1 para textos críticos;
- controles interativos devem ter foco visível;
- campos e botões devem possuir rótulos semânticos ou texto acessível.

### NFR-05 — Manutenibilidade
- lógica de API deve ficar isolada em serviço específico;
- conversão de unidade deve ser centralizada em utilitário;
- a UI não deve conter regras de negócio complexas de consulta e transformação de dados.

### NFR-06 — Observabilidade
- erros de rede e resposta inválida devem resultar em mensagens legíveis ao usuário;
- falhas relevantes devem ser rastreáveis em logs de desenvolvimento e diagnósticos de execução.

## 8. Critérios de aceitação globais

- A aplicação deve permitir buscar uma cidade e consultar o clima em um fluxo simples com até 3 ações principais: digitar, selecionar e visualizar.
- A interface deve estar em português do Brasil e operar em Celsius por padrão.
- O usuário deve conseguir alternar a unidade de temperatura sem recarregar a página ou repetir a busca.
- A aplicação deve informar claramente estados de carregamento, erro e ausência de resultados.
- Em mobile, a interface deve manter legibilidade e ausência de overflow horizontal.

## 9. Casos de borda

1. Busca com texto vazio
2. Busca com menos de 2 caracteres
3. Cidade não encontrada
4. Múltiplas cidades com o mesmo nome
5. Falha de rede
6. Timeout da API
7. Resposta da API sem dados obrigatórios
8. Temperatura ausente em um dia da previsão
9. Tela em largura muito estreita
10. Usuário navegando com teclado

## 10. Riscos e mitigação

### Risco: dependência de API externa
Mitigação: validar resposta, tratar timeouts, isolar a integração em serviço único.

### Risco: ambiguidade de cidade
Mitigação: exibir lista de resultados e exigir seleção explícita.

### Risco: falha na experiência mobile
Mitigação: validar layout em telas estreitas e evitar overflow.

### Risco: regressões em estados de erro
Mitigação: priorizar testes de busca vazia, erro de rede e troca de unidade.

## 11. Definição de pronto

A funcionalidade será considerada pronta quando:
- a busca por cidade funciona conforme critérios de aceite;
- a cidade ativa e a previsão são consistentes;
- a unidade de temperatura alterna corretamente;
- estados de carregamento, erro e vazio estão implementados e testados;
- a interface funciona em mobile sem overflow horizontal;
- os requisitos de acessibilidade básicos estão cobertos.

7. Qual é o nível mínimo de compatibilidade desejado com navegadores e dispositivos?
