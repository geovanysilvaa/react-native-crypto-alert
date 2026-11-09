# Planejamento de cenários E2E com IA — Crypto Alert

Documento da Etapa 2: registro do prompt usado, da resposta bruta da IA,
dos cenários selecionados e da análise crítica feita comparando cada cenário
com a interface real do app (`src/`).

---

## 1) Prompt utilizado

Este é o prompt enviado à IA (uma única vez, sem acesso ao código nesta etapa):

> Você é engenheiro de QA de testes E2E mobile com Maestro. App Crypto Alert,
> 3 abas: List (criptomoedas com preço e variação 24h via API CoinGecko, com
> tela de erro e retry), Convert (converte valor de cripto para fiat USD, BRL,
> EUR, GBP, JPY) e Alerts (listar, criar alerta com cripto, preço alvo e
> condição acima/abaixo, e excluir; alertas ficam salvos no aparelho).
> Proponha de 10 a 12 cenários E2E com título, pré-condição, passos, resultado
> esperado e prioridade. Diga quais automatizar com Maestro e liste as
> suposições que fez sobre a interface.

---

## 2) Resposta original da IA

> A seguir está a resposta da IA **copiada sem edição**, exatamente como
> retornada (nenhuma correção aplicada — as divergências com o app real estão
> analisadas na seção 4).

### Cenários E2E propostos

**C01 — Listar criptomoedas com preço e variação 24h**
- Pré-condição: app instalado, com internet e API CoinGecko disponível.
- Passos: abrir o app; aguardar o carregamento; observar a lista.
- Resultado esperado: lista com várias criptomoedas, cada item com nome/símbolo, preço em USD e variação de 24h (com sinal e percentual).
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C02 — Estado de erro e retry na lista**
- Pré-condição: primeira carga sem conexão (ou API indisponível).
- Passos: abrir o app sem rede; observar a mensagem de erro; tocar em "Tentar novamente".
- Resultado esperado: mensagem de erro visível; o retry reexecuta a busca e exibe a lista quando a rede volta.
- Prioridade: Alta. Automatizar com Maestro: Sim (com injeção de falha).

**C03 — Navegação entre as 3 abas**
- Pré-condição: app aberto.
- Passos: tocar em List; tocar em Convert; tocar em Alerts.
- Resultado esperado: cada aba exibe seu título e seu conteúdo correspondente.
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C04 — Conversão de cripto para fiat (formato válido)**
- Pré-condição: app aberto na aba Convert, com dados carregados.
- Passos: selecionar uma cripto (ex.: Bitcoin); selecionar a moeda fiat (USD); digitar um valor; ler o resultado.
- Resultado esperado: resultado numérico formatado; ao trocar fiat/cripto o valor é recalculado.
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C05 — Conversão com valor inválido**
- Pré-condição: aba Convert.
- Passos: digitar "0"; observar a mensagem; apagar o campo.
- Resultado esperado: mensagem de validação (algo como "valor inválido"); ao apagar, volta ao estado inicial/placeholder.
- Prioridade: Média. Automatizar com Maestro: Sim.

**C06 — Criar alerta "acima"**
- Pré-condição: aba Alerts sem alertas.
- Passos: tocar no botão de criar; selecionar a cripto; informar o preço alvo; selecionar a condição "acima"; salvar.
- Resultado esperado: o alerta aparece na lista com cripto, preço e condição.
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C07 — Criar alerta "abaixo"**
- Pré-condição: aba Alerts.
- Passos: abrir criação; selecionar cripto; informar preço; selecionar a condição "abaixo"; salvar.
- Resultado esperado: o alerta aparece com a condição "abaixo".
- Prioridade: Média. Automatizar com Maestro: Sim.

**C08 — Botão salvar desabilitado com o formulário vazio**
- Pré-condição: tela de criar alerta aberta com os campos vazios.
- Passos: observar o botão de salvar; tentar tocar nele.
- Resultado esperado: o botão fica desabilitado e nada é criado.
- Prioridade: Média. Automatizar com Maestro: Sim.

**C09 — Excluir alerta**
- Pré-condição: existir ao menos 1 alerta.
- Passos: tocar no ícone de lixeira do card; confirmar a exclusão (se houver diálogo).
- Resultado esperado: o card desaparece da lista.
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C10 — Persistência de alertas após reiniciar**
- Pré-condição: existir 1 alerta salvo.
- Passos: fechar o app; reabrir o app.
- Resultado esperado: o alerta continua na lista (salvo no aparelho).
- Prioridade: Alta. Automatizar com Maestro: Sim.

**C11 — Estado vazio de alertas**
- Pré-condição: nenhum alerta cadastrado.
- Passos: abrir a aba Alerts.
- Resultado esperado: mensagem de estado vazio orientando a criar o primeiro alerta.
- Prioridade: Baixa. Automatizar com Maestro: Sim.

**C12 — Formatação de números (casas decimais e símbolo)**
- Pré-condição: aba List ou Convert.
- Passos: observar a formatação de preços e do resultado.
- Resultado esperado: preços e resultados com formatação consistente (símbolo de moeda e casas decimais fixas).
- Prioridade: Baixa. Automatizar com Maestro: Parcial (checar formato, não o valor).

### Quais automatizar com Maestro

- **Automatizar:** C01, C03, C04, C05, C06, C08, C09, C10, C11.
- **Parcial:** C02 (depende de forçar erro de rede), C07, C12.
- **Sugestão de ordem:** navegação → conversão → alertas (criar, excluir, persistir).

### Suposições feitas sobre a interface

1. Os nomes/rótulos das abas são **List**, **Convert** e **Alerts**.
2. O **título de cada tela é igual ao nome da aba** (ex.: tela List tem título "List").
3. Existe um **botão flutuante "+"** na aba Alerts para criar alerta.
4. O botão de **salvar** fica **desabilitado** quando o formulário está vazio.
5. Os campos de seleção abrem um **modal/dropdown** com a lista de opções.
6. Cada card de alerta tem um **ícone de lixeira** para excluir, com **confirmação** antes de apagar.
7. Há uma **mensagem específica** para valor inválido (texto aproximado "valor inválido").
8. As condições aparecem como **"Acima"/"Abaixo"** em português.
9. Ao apagar o valor digitado, o **placeholder** volta a aparecer.
10. Os preços da List são exibidos em **USD** e a variação mostra **sinal e %**.
11. O app **persiste** os alertas no aparelho (sobrevivem ao reinício).

---

## 3) Cenários selecionados

Para os fluxos Maestro, selecionei um subconjunto que cobre o "caminho feliz"
e pontos de validação visíveis, priorizando o que é estável e verificável sem
mock de rede:

| # | Cenário de origem | Fluxo Maestro | Observação |
|---|---|---|---|
| 1 | C03 | `.maestro/01_navegacao_entre_abas.yaml` | Navegação pelas 3 abas + títulos reais das telas. |
| 2 | C04 | `.maestro/02_conversao_valida.yaml` | Troca de fiat e de cripto; valida o **formato** do resultado. |
| 3 | C05 | `.maestro/03_conversao_valor_invalido.yaml` | Valor "0" mostra a mensagem real; apagar volta ao placeholder. |
| 4 | C06 + C08 | `.maestro/04_criar_alerta_acima.yaml` | Salvar desabilitado com form vazio; preencher; salvar; ver o card. |
| 5 | C09 | `.maestro/05_excluir_alerta.yaml` | Exclusão pelo `accessibilityLabel` real da lixeira. |
| 6 | C10 | `.maestro/06_alerta_persiste_apos_reiniciar.yaml` | `stopApp` + `launchApp` sem `clearState`. |

**Descartados para automação (nesta entrega):**
- **C02** (erro/retry): exige forçar falha de rede (modo avião / mock); instável de automatizar sem ferramenta extra. Fica como verificação manual.
- **C07** (alerta "abaixo"): coberto estruturalmente por C06; manteria como extra depois.
- **C11** (estado vazio de alertas): depende de estado limpo; é fácil, mas menos prioritário que criar/excluir.
- **C12** (formatação): embutido nas asserções de formato de C04.

---

## 4) Análise crítica (comparação com `src/`)

Aqui **sim** li o código (`src/screens`, `src/components`, `src/context`,
`src/data`, `src/app`) e comparei cada cenário/suposição com a interface real.
Resumo: a IA acertou o essencial (3 abas, fluxos de conversão e alerta,
persistência), mas errou **títulos de tela**, **gatilho de criar alerta** e
**confirmação de exclusão**, e supôs vários textos em português quando na verdade são em inglês.

### 4.1 O que foi **aproveitado**

- **C03 (navegação):** os rótulos das abas realmente são **List**, **Convert** e **Alerts** em [src/app/(tabs)/_layout.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/app/(tabs)/_layout.tsx#L29-L55). ✔
- **C04 (conversão):** confere. Em [useConversionScreen.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/ConversionScreen/useConversionScreen.ts#L45-L74), o resultado é `symbol + formatPrice(valor)` (ex.: `$123.45`); trocar cripto/fiat recalcula. Automatizável validando **formato**. ✔
- **C05 (valor inválido):** confere com o comportamento real. `showInvalid = Boolean(amount) && !isValid` → digitar `0` mostra a mensagem; apagar volta ao placeholder ([ConversionScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/ConversionScreen/ConversionScreen.tsx#L64-L77)). ✔
- **C06 (criar alerta):** confere. O card mostra o **nome** da cripto, o texto "Alert when price goes above/below" e o preço ([AlertCard.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/components/AlertCard/AlertCard.tsx#L34-L43)). ✔
- **C08 (salvar desabilitado):** confere. O botão tem `disabled={!isFormValid}` ([CreateAlertScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CreateAlertScreen/CreateAlertScreen.tsx#L89-L94)). ✔
- **C10 (persistência):** confere. Os alertas são salvos em AsyncStorage com a chave `cryptoAlerts` ([useAlertProvider.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/context/AlertProvider/useAlertProvider.ts#L11-L60)). ✔
- **Suposição 5 (seleção abre modal):** confere — `SelectField` usa um `Modal` de baixo para cima ([SelectField.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/components/SelectField/SelectField.tsx#L55-L94)). ✔
- **Suposição 6 (há ícone de lixeira):** confere — `accessibilityLabel="Delete alert"` ([AlertCard.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/components/AlertCard/AlertCard.tsx#L45-L52)). ✔ (mas ver 4.2 sobre a confirmação)
- **Suposição 9 (placeholder volta):** confere — placeholder do resultado é "Enter an amount"; o do input é "0.00". ✔
- **Suposição 10 (USD + sinal/%):** confere — `formatChange` adiciona sinal e "%" ([utils/index.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/utils/index.ts#L8-L11)). ✔

### 4.2 O que foi **corrigido** (a IA supôs errado)

- **Suposição 2 — título da tela = nome da aba: ERRADO.** Os títulos internos são outros:
  - Aba **List** → título interno **"Cryptocurrencies"** ([CryptoListScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CryptoListScreen/CryptoListScreen.tsx#L16)).
  - Aba **Convert** → título interno **"Convert Crypto"** ([ConversionScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/ConversionScreen/ConversionScreen.tsx#L30)).
  - Aba **Alerts** → título interno **"Price Alerts"** ([PriceAlertsScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/PriceAlertsScreen/PriceAlertsScreen.tsx#L16)).
  - Tela de criação → título **"Create Alert"** ([CreateAlertScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CreateAlertScreen/CreateAlertScreen.tsx#L32)).
  - **Consequência nos fluxos:** as asserções de título usam esses textos reais, não os nomes das abas.
- **Suposição 3 — botão flutuante "+": ERRADO.** Não há botão flutuante. Existe um botão **"New Alert"** no cabeçalho da tela de alertas (com ícone `add`) e, no estado vazio, um botão **"Create Alert"** ([PriceAlertsScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/PriceAlertsScreen/PriceAlertsScreen.tsx#L15-L39)).
- **Suposição 6 — confirmação antes de excluir: ERRADO.** A exclusão é **imediata**, sem diálogo. O toque na lixeira chama `onDelete` direto ([useAlertCard.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/components/AlertCard/useAlertCard.ts#L9-L11)).
- **Suposição 7 — texto de "valor inválido": AJUSTADO.** O texto real é **"Invalid amount"** (não "valor inválido").
- **Suposição 8 — "Acima"/"Abaixo" em português: ERRADO.** As opções e o card estão em **inglês**: "Above target price" / "Below target price" e o card mostra "Alert when price goes **above**/**below**" (o valor cru, minúsculo) ([CreateAlertScreen.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CreateAlertScreen/CreateAlertScreen.tsx#L70-L86), [AlertCard.tsx](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/components/AlertCard/AlertCard.tsx#L36-L39)).
  - **Correção de texto:** todo o app está em **inglês**; os fluxos usam rótulos/botões em inglês.
- **Suposição 11 (persistência):** correta em essência, mas atenção — o **estado de alertas é global** (`AlertProvider` no root), então `clearState: true` apaga os alertas. O fluxo 06 tem de usar `stopApp`/`launchApp` **sem** `clearState`.
- **C01 (lista):** funciona, porém os itens vêm da **API** e variam. As asserções devem checar **formato** (existe `$` + número, `%`), nunca valores.
- **Validação da tela de criação:** a condição **já vem pré-selecionada** como "above" ([useCreateAlertScreen.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CreateAlertScreen/useCreateAlertScreen.ts#L10)). Logo, o form vazio deixa Salvar desabilitado por causa de **cripto e preço** vazios, não da condição.
- **Lista de criptos na criação:** vem da **lista estática** (`cryptocurrencies`), então os nomes são **determinísticos** (ex.: "Bitcoin", "Ethereum") — bom para asserções. Já a **conversão** usa dados da API com fallback para a lista estática.

### 4.3 O que foi **descartado** e por quê

- **C02 (erro/retry):** descartado da automação. Forçar falha de rede de forma confiável exige modo avião/mock; fica como verificação manual. O `ErrorState` só aparece quando `loadingState === "error"`, o que depende de a chamada falhar ([useCryptoListScreen.ts](file:///c:/Users/batis/Desktop/Crypt/react-native-crypto-alert/src/screens/CryptoListScreen/useCryptoListScreen.ts#L26-L30)).
- **C07 ("abaixo"):** descartado como fluxo próprio nesta entrega (estruturalmente igual ao C06); fica como candidato a extra.
- **C11 (estado vazio de alertas):** descartado por baixo valor nesta entrega.
- **C12 (formatação como fluxo separado):** descartado como fluxo; incorporado às asserções de formato do C04.

### 4.4 A CONFIRMAR NO EMULADOR

- **A CONFIRMAR NO EMULADOR — estado da aba List:** o `.env` ficou com o placeholder `YOUR_COINGECKO_API_KEY_HERE`; no build `release` isso pode fazer a API retornar erro e a List cair em **"Failed to load"**. Verificar se a aba List mostra a **lista** ou o **estado de erro**. Isso muda o fluxo 01.
- **A CONFIRMAR NO EMULADOR — rótulos das abas:** confirmar que a tab bar exibe literalmente "List", "Convert" e "Alerts" (são os `title` das `Tabs.Screen`).
- **A CONFIRMAR NO EMULADOR — acesso à tela de criação:** confirmar se o botão do cabeçalho é "New Alert" e/ou o botão do estado vazio é "Create Alert", e se ambos abrem a tela modal "Create Alert".
- **A CONFIRMAR NO EMULADOR — seleção de cripto/fiat:** confirmar como o `Modal` do `SelectField` se comporta (rolagem da lista, fechamento ao tocar na opção) para ajustar os `tapOn`.
- **A CONFIRMAR NO EMULADOR — exclusão sem confirmação:** confirmar que tocar na lixeira remove o card na hora (sem diálogo do sistema).
- **A CONFIRMAR NO EMULADOR — persistência:** confirmar que `stopApp` + `launchApp` (sem `clearState`) mantém o alerta.
- **A CONFIRMAR NO EMULADOR — teclado:** confirmar se o teclado numérico (decimal-pad) cobre o campo/botão "Save Alert" e se é preciso fechar o teclado antes de tocar em salvar.
