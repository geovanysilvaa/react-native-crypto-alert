# Reflexão sobre o artigo GenIA-E2ETest

Fonte: **GenIA-E2ETest: A Generative AI-Based Approach for End-to-End Test Automation** (arXiv:2510.01024v1; publicado no SBES 2025).

## 1) Qual problema a abordagem GenIA-E2ETest procura resolver?

O GenIA-E2ETest nasce de um problema conhecido: testar software manualmente é caro, demorado e sujeito a erro, e embora os LLMs já ajudem a gerar testes, a maioria das soluções ataca apenas o nível de unidade. O teste ponta a ponta (E2E), que valida fluxos completos do usuário, acabou ficando de fora. Automatizar E2E é especialmente difícil porque exige mapear elementos de interface, dominar frameworks de automação e investir bastante tempo — barreiras que afastam muitas equipes, sobretudo as ágeis ou com recursos limitados. As ferramentas comerciais que já usam IA para E2E (como Testim, Functionize e testRigor) são proprietárias, caras, pouco integradas a frameworks abertos e focadas em regressão e fluxos gravados. A proposta do artigo é preencher essa lacuna com uma abordagem open-source que converte requisitos funcionais em linguagem natural em scripts E2E executáveis.

## 2) Como a IA generativa pode ajudar na criação de testes E2E?

A contribuição central é uma estratégia de multi-level prompting, inspirada em AI Chains, que decompõe a geração em três níveis encadeados. No primeiro, a IA interpreta o cenário em linguagem natural e o reescreve como um JSON estruturado, dividindo-o em módulos por página — cada módulo delimitado por uma transição de URL — com os passos em ordem cronológica (preencher campos, clicar, verificar), usando uma persona de engenheiro de automação e prompting zero-shot. No segundo, a aplicação é rastreada (crawl) e seus elementos de interface, junto com atributos contextuais, viram um catálogo legível por máquina. No terceiro, requisitos interpretados e elementos mapeados se combinam para gerar scripts compatíveis com Robot Framework, priorizando manutenibilidade e legibilidade. O efeito é reduzir o trabalho manual de traduzir um cenário escrito em automação executável.

## 3) Quais limitações ou riscos existem ao usar testes gerados por IA?

O artigo não esconde as fragilidades. Os resultados mostram sensibilidade a navegação dependente de contexto e a conteúdo dinâmico, e a análise identificou cinco desafios recorrentes que afetaram a consistência dos resultados — o mais sério sendo a navegação dependente de contexto, em que páginas só ficam acessíveis após ações prévias (por exemplo, uma página de registro que redireciona de volta se os dados anteriores não forem informados, com a identificação dos elementos acontecendo antes do fluxo completo de execução). Além disso, a avaliação cobriu apenas duas aplicações web e teve bom desempenho em cenários web típicos, o que não garante generalização para outros contextos. Os próprios números reforçam o cuidado necessário: 77% nas métricas de elementos, 82% de precisão de execução e 85% de recall — ou seja, nem tudo sai correto de primeira.

## 4) Por que um teste gerado por IA ainda precisa de revisão humana?

Os números explicam por que a revisão humana continua indispensável. A taxa média de modificação manual foi de 10%, o que significa que praticamente todo script gerado precisou de algum ajuste. Como as métricas não chegam a 100% e há falhas conhecidas na identificação de elementos e na navegação dependente de contexto, cabe à pessoa revisar e corrigir os scripts, confirmar que seletores e asserções correspondem ao comportamento real e validar o que de fato aconteceu na execução.

## 5) Como o Maestro pode ser usado para validar, na prática, um fluxo sugerido por uma IA?

A lógica do artigo é clara: partir de um requisito em linguagem natural, gerar um script executável e então executá-lo, com foco em legibilidade, manutenibilidade e mínimo ajuste manual. É nesse último passo que o Maestro entra no nosso projeto: um fluxo sugerido pela IA se transforma num arquivo YAML declarativo em `.maestro/`, e o `maestro test` o executa contra o app real no emulador, devolvendo pass/fail e capturas de tela. Como o artigo alerta para a sensibilidade a conteúdo dinâmico e a fluxos dependentes de contexto, rodar o fluxo no Maestro é exatamente o momento de verificar se aquilo que a IA propôs funciona de verdade — com seletores, textos e estados reais — e de ajustar o que foi apenas suposto. Vale registrar que o artigo não menciona o Maestro; aqui ele aparece como motor de execução das ideias descritas.
