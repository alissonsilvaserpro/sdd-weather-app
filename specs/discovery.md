# Discovery — Aplicação de Previsão do Tempo

## Contexto

A empresa solicitou o desenvolvimento de uma aplicação web para consulta de previsão do tempo, com foco em usabilidade, rapidez e adaptação a dispositivos móveis. O produto deve permitir que usuários busquem cidades, visualizem as condições climáticas atuais e a previsão dos próximos 5 dias, além de alternar entre unidades de temperatura em Celsius e Fahrenheit.

O contexto de uso principal é a consulta rápida de informações meteorológicas em situações do dia a dia, como planejamento de rotina, viagens, deslocamentos e atividades ao ar livre. Como a solução será disponibilizada em navegador, é essencial que a experiência seja responsiva, com carregamento ágil e interface clara mesmo em telas pequenas.

Também é importante considerar que a aplicação depende de uma fonte de dados externa para clima e geolocalização. Portanto, o produto deve ser resiliente a falhas de rede, respostas incompletas e buscas sem resultado, preservando uma experiência coerente e informativa para o usuário.

## Requisitos Funcionais

1. Busca de cidades
   - O usuário deve poder buscar cidades por nome.
   - O sistema deve retornar resultados plausíveis com base na consulta informada.
   - Quando não houver resultados, a interface deve indicar claramente que nada foi encontrado.

2. Visualização do clima atual
   - O usuário deve visualizar as condições climáticas atuais da cidade selecionada.
   - A tela deve exibir pelo menos informações como temperatura, descrição do clima, umidade, vento, precipitação e pressão.

3. Previsão de 5 dias
   - O sistema deve apresentar a previsão dos próximos 5 dias para a cidade escolhida.
   - Cada dia deve conter indicadores relevantes, como temperatura mínima, máxima e probabilidade de precipitação.

4. Alternância entre Celsius e Fahrenheit
   - O usuário deve poder alternar entre exibição em Celsius e Fahrenheit.
   - A conversão deve ser aplicada de forma consistente na interface.
   - A troca de unidade não deve exigir nova busca ou recarga de dados da API.

5. Experiência mobile
   - A aplicação deve ser funcional e legível em dispositivos móveis.
   - Os elementos devem ser adaptados para telas pequenas, com navegação e leitura confortáveis.
   - Campos de busca, botões e cards de previsão devem manter a usabilidade em layout responsivo.

6. Tratamento de estado e feedback
   - O sistema deve informar estados de carregamento.
   - O sistema deve informar erros de rede ou falhas na consulta.
   - O sistema deve oferecer uma forma simples de repetir a operação em caso de falha.

7. Acessibilidade básica
   - A interface deve apresentar labels e estrutura semântica adequadas para navegação.
   - Botões e controles devem ser operáveis por teclado e com feedback visual suficiente.

## Requisitos Não-Funcionais

1. Desempenho
   - A aplicação deve responder rapidamente às consultas do usuário.
   - O carregamento inicial e as buscas subsequentes devem ser percebidos como rápidos e leves.

2. Responsividade
   - A interface deve funcionar adequadamente em diferentes tamanhos de tela, priorizando mobile first.
   - Layouts e componentes devem ajustar sem quebrar a usabilidade.

3. Confiabilidade
   - O sistema deve lidar com falhas na API externa de forma amigável.
   - Mensagens de erro devem ser compreensíveis para o usuário final.

4. Manutenibilidade
   - A solução deve manter separação clara entre interface, regras de negócio e acesso a dados.
   - O código deve permitir evolução futura sem acoplamento excessivo.

5. Acessibilidade
   - A aplicação deve seguir boas práticas de inclusão digital.
   - Elementos interativos devem ter contraste e semântica adequadas.

6. Observabilidade básica
   - O sistema deve permitir diagnóstico de falhas em consultas e processamento de dados.
   - Menos de forma operacional avançada, mas com logs e mensagens de erro úteis no fluxo de uso.

## Riscos

1. Dependência de API externa
   - A qualidade da experiência depende da disponibilidade e do desempenho da API de previsão do tempo.
   - Falhas de rede, timeout ou respostas incompletas podem impactar diretamente a usabilidade.

2. Ambiguidade na busca por cidade
   - Nomes de cidades repetidos em diferentes países ou estados podem gerar resultados múltiplos e confusão.
   - Sem seleção clara, o usuário pode receber a cidade errada.

3. Variação no dado meteorológico
   - Diferentes regiões podem ter condições climáticas muito distintas, e a previsão pode parecer inconsistente se a cidade não for corretamente identificada.

4. Design para mobile
   - Componentes que funcionam bem em desktop podem ficar compactos ou pouco legíveis em telas pequenas.
   - Há risco de qualidade de uso se a interface não for testada em dispositivos reais.

5. Cobertura de testes
   - Se a aplicação não for validada em cenários de erro, vazio e responsividade, problemas podem passar despercebidos antes do lançamento.

## Perguntas em Aberto

1. A aplicação deve ter autenticação de usuário?
   - Até o momento, não há indicação de login ou perfil do usuário.

2. A busca deve aceitar apenas cidades ou também regiões, países e coordenadas?
   - O briefing menciona apenas cidades, mas pode haver necessidade de suporte expandido no futuro.

3. Existe uma preferência por uma experiência PWA ou somente web responsiva?
   - O briefing menciona uso em dispositivos móveis, mas não especifica instalação em tela inicial.

4. A aplicação deve exibir dados em tempo real ou apenas pré-processados/atualizados pela API?
   - O briefing menciona previsão do tempo, mas não define atualização em tempo real.

5. Qual é o público-alvo principal: usuários casuais, viajantes, profissionais do setor ou geral?
   - Isso influencia a profundidade e a clareza da interface.

6. Há requisitos de internacionalização?
   - O produto pode precisar de suporte a diferentes idiomas ou formatos regionais.

7. A empresa já definiu a API de clima a ser usada?
   - O briefing aponta para uso de uma solução externa, mas não especifica a integração exacta.

## Decisões

1. Fonte de dados: Open-Meteo (sem API key)
   - Justificativa: elimina a necessidade de autenticação e reduz a fricção operacional, mantendo uma solução de dados meteorológicos de código aberto, gratuita e facilmente acessível para um projeto de treinamento ou MVP.
   - Resolve: elimina a ambiguidade sobre a API de clima e reduz a dependência de configurações sensíveis ou credenciais de conta.

2. "5 dias" = hoje + 4 dias
   - Justificativa: define claramente o escopo da previsão, evitando ambiguidades sobre contagem de dias e alinhando o comportamento esperado pela equipe e pelo usuário.
   - Resolve: responde à pergunta sobre a regra de previsão e evita interpretações inconsistentes sobre o período exibido.

3. Unidade padrão: Celsius
   - Justificativa: Celsius é o padrão mais comum em aplicações de clima em contextos internacionais e em interfaces com foco em experiência simples e familiar para a maioria dos usuários.
   - Resolve: define a unidade inicial da tela e garante que a alternância para Fahrenheit seja tratada como escolha do usuário, não como comportamento indefinido.

4. Sem autenticação e sem persistência de servidor
   - Justificativa: a aplicação é uma SPA simples para consulta de clima e não exige gerenciamento de usuários, perfis ou armazenamento de dados no backend.
   - Resolve: responde diretamente à questão sobre autenticação e reduz a complexidade de desenvolvimento, manutenção e operação.

5. Idioma da UI: pt-BR
   - Justificativa: alinha a interface com o contexto local do usuário e com o objetivo de uma experiência clara e natural para o público alvo do projeto.
   - Resolve: fecha a incerteza sobre internacionalização mínima da interface e padroniza labels, mensagens e textos visuais.

## Suposições

1. A aplicação será uma SPA web, sem necessidade de backend próprio.
2. A integração com API externa será suficiente para obter dados de geocodificação e clima.
3. Os usuários esperam uma experiência simples, direta e em tempo real, com navegação mínima.
4. O foco principal será em mobile, mas a aplicação também deve funcionar em desktop.
5. A unidade de temperatura será seletiva na interface, sem alteração do dado bruto de origem.
6. A busca por cidade será suficiente para atender o escopo inicial do produto.
7. O sistema deve priorizar clareza, usabilidade e estabilidade em vez de recursos mais avançados que não constam no briefing.
