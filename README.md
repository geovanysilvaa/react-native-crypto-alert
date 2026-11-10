# Crypto Alert — Testes E2E com Maestro

App React Native (Expo) fork de `react-native-crypto-alert` com três abas — **List**, **Convert** e **Alerts** — e uma suíte de testes end-to-end automatizada com o [Maestro](https://maestro.mobile.dev/).

- **List**: lista criptomoedas com preço e variação de 24h (API CoinGecko), com tela de erro e retry.
- **Convert**: converte um valor de criptomoeda para moeda fiduciária (USD, BRL, EUR, GBP, JPY).
- **Alerts**: lista, cria (cripto + preço alvo + condição acima/abaixo) e exclui alertas, que ficam salvos no aparelho.

App ID (Android e iOS): **`com.cryptoalert.app`**.

## Pré-requisitos

- **Node.js** (LTS) e **npm**.
- **JDK 17** com a variável `JAVA_HOME` configurada.
- **Android Studio** com o SDK do Android e um **emulador (AVD)** criado; `platform-tools` (para o `adb`) no `PATH`.
- **Maestro CLI** instalado (ver seção específica abaixo).

## 1. Instalar as dependências

```powershell
npm install
```

## 2. Configurar variáveis de ambiente (`.env`)

Copie o arquivo de exemplo e preencha a chave da API CoinGecko:

```powershell
Copy-Item .env.example .env
```

Conteúdo esperado do `.env`:

```
EXPO_PUBLIC_COINGECKO_API_KEY=YOUR_COINGECKO_API_KEY_HERE
```

O `.env` **não** deve ser commitado (já está no `.gitignore`). Como o build `release` embute as variáveis `EXPO_PUBLIC_*` em tempo de compilação, ele precisa estar preenchido **antes** de gerar o app.

## 3. Abrir o emulador Android

```powershell
# listar os AVDs disponíveis
& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -list-avds

# iniciar um AVD (troque NOME_DO_AVD pelo nome listado acima)
& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -avd NOME_DO_AVD

# confirmar que o device está online (deve aparecer como "device")
adb devices
```

## 4. Gerar e instalar o app

O app precisa estar **instalado** no emulador antes de rodar os testes:

```powershell
npx expo run:android --variant release
```

Isso compila o projeto e instala o APK com o id `com.cryptoalert.app`. Para conferir:

```powershell
adb shell pm list packages | findstr cryptoalert
```

## 5. Instalar o Maestro

Requisito: **Java 17+** (`java -version`).

1. Baixe o `maestro.zip` mais recente em: `https://github.com/mobile-dev-inc/maestro/releases/latest/download/maestro.zip`.
2. Extraia para um diretório fixo (ex.: `C:\maestro`).
3. Adicione `C:\maestro\bin` ao `PATH` e reinicie o terminal:

```powershell
setx PATH "%PATH%;C:\maestro\bin"
```

4. Valide a instalação:

```powershell
maestro --help
```

## 6. Executar os fluxos

```powershell
# Todos os fluxos da pasta .maestro/
maestro test .maestro/

# Um fluxo específico
maestro test .maestro/04_criar_alerta_acima.yaml

# Salvando as capturas (takeScreenshot) em uma pasta
maestro test --test-output-dir=evidencias/execucao .maestro/05_excluir_alerta.yaml
```

> Observação: os fluxos que **não** usam `clearState` (como o de persistência) reaproveitam o estado do app no aparelho. Para rodar tudo do zero, garanta que o app esteja instalado e feche-o antes.

## Fluxos em `.maestro/`

| Arquivo | Cenário | O que valida |
|---|---|---|
| `01_navegacao_entre_abas.yaml` | Navegação entre abas | Abas **List / Convert / Alerts** e os títulos **Cryptocurrencies**, **Convert Crypto**, **Price Alerts** |
| `02_conversao_valida.yaml` | Conversão válida | Troca de criptomoeda e de moeda fiduciária; valida o **formato** do resultado (não o valor) |
| `03_conversao_valor_invalido.yaml` | Valor inválido | Digitar `0` exibe **Invalid amount**; apagar volta ao placeholder **Enter an amount** |
| `04_criar_alerta_acima.yaml` | Criar alerta (acima) | **Save Alert** desabilitado com o formulário vazio; criação e exibição do card |
| `05_excluir_alerta.yaml` | Excluir alerta | Exclusão via `accessibilityLabel` **Delete alert**; volta ao estado vazio |
| `06_alerta_persiste_apos_reiniciar.yaml` | Persistência | `stopApp` + `launchApp` sem `clearState` mantém o alerta salvo |
| `subflows/criar_alerta.yaml` | Subfluxo reutilizável | Cria um alerta parametrizado por `CRYPTO_NAME`, `PRICE` e `CONDITION_TITLE` |

## Problemas comuns

### Erro de caminho longo no Windows (`ninja: Filename longer than 260 characters`)

No Windows, o build do Android pode falhar porque o caminho dos arquivos gerados ultrapassa o limite de 260 caracteres do sistema (erro do `ninja`). Solução planejada:

1. Habilitar caminhos longos no Windows (requer terminal como administrador):

```powershell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

2. Habilitar caminhos longos no Git:

```powershell
git config --system core.longpaths true
```

3. Usar um caminho de projeto **mais curto** — por exemplo, mover o projeto para `C:\crypto` ou mapear um drive curto:

```powershell
subst X: "C:\Users\batis\Desktop\Crypt\react-native-crypto-alert"
# depois rode o build a partir de X:\
```

### Aba **List** sem dados / tela de erro

Se a lista não carregar, verifique se `EXPO_PUBLIC_COINGECKO_API_KEY` está preenchida no `.env` e **regenere** o app (a variável é embutida no build).

### O Maestro não encontra o app

Confirme que o app está instalado no emulador (`adb shell pm list packages | findstr cryptoalert`) e que o `appId` nos fluxos é `com.cryptoalert.app`.

### Onde as capturas (`takeScreenshot`) são salvas

Sem `--test-output-dir`, as imagens vão para a pasta de saída padrão do Maestro. Com `--test-output-dir=evidencias/execucao`, elas são salvas em `evidencias/execucao/`.

## Evidências

**Status: pendente.** A geração das capturas não foi possível porque o build do Android falhou no Windows por caminho longo demais (`ninja: Filename longer than 260 characters`). Após aplicar a solução descrita em *Problemas comuns* e rodar os fluxos 04 e 05 com `--test-output-dir=evidencias/execucao`, copie as imagens para `evidencias/` e o README exibirá:

```markdown
![Alerta criado](evidencias/04_criar_alerta_card.png)
![Alerta excluído](evidencias/05_alerta_excluido.png)
```

Basta remover o bloco de código acima e transformá-lo em imagens depois que os arquivos existirem na pasta `evidencias/`.
