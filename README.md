# RUM-Datadog-React-Native
Instrumentacao de logs do SDK Datadog em React Native para observar chamadas do Firebase via proxies JavaScript.

## Visao geral
Este projeto usa um proxy do modulo `firebase/firestore` para interceptar funcoes e registrar logs com duracao, status e erro. Os logs sao enviados via `DdLogs` e ficam correlacionaveis com a sessao RUM do app.

## Onde esta a instrumentacao
- `instrumentedFirebase.ts`: proxy do Firestore e mapeamento das funcoes instrumentadas.
- `utils/datadogInstrumentation.ts`: wrapper generico `instrumentCall` que gera logs de inicio/sucesso/erro.
- `App.tsx`: uso dos imports instrumentados no app e configuracao do Datadog SDK.

## Como a instrumentacao funciona
1. O proxy intercepta chamadas de funcoes do Firestore (ex: `getDocs`, `addDoc`).
2. Ele cria `operationName` e `context` (tags/payload) para a chamada.
3. A chamada original retorna uma `Promise`, que eh passada para `instrumentCall`.
4. `instrumentCall` registra:
   - Inicio da chamada
   - Sucesso com duracao
   - Erro com duracao, mensagem e stack

## Campos de log gerados
Os logs usam um payload padronizado (ver `utils/datadogInstrumentation.ts`):
- `callId`: id unico por chamada
- `operationName`: nome da operacao (ex: `Firestore: Get Docs`)
- `logType`: `instrumented_call`
- `callStatus`: `initiated` | `success` | `failed`
- `durationMs`: duracao em ms (somente no fim)
- `payload`: dados enviados (quando aplicavel)
- `collectionName`, `docId`: tags derivadas do Firestore
- `errorMessage`, `errorCode`, `errorStack`: quando falha

## Funcoes do Firestore instrumentadas
No arquivo `instrumentedFirebase.ts`, as funcoes com instrumentacao declarada sao:
- `getDocs`
- `addDoc`
- `updateDoc`
- `deleteDoc`

Observacao: `onSnapshot` nao esta instrumentado no proxy por ser baseado em callbacks. Se precisar, crie um wrapper dedicado para logar cada emissao ou o unsubscribe.

## Como usar no app
Importe sempre pelo proxy:
```ts
import { getDocs, addDoc, updateDoc, deleteDoc, collection, doc } from './instrumentedFirebase';
```

Qualquer chamada usando esses exports gera logs automaticamente.

## Como estender
Para adicionar novas funcoes:
1. Edite `functionsToInstrument` em `instrumentedFirebase.ts`.
2. Defina `operationName` e `context` com tags/payload.
3. Exporte a funcao no final do arquivo.

## Boas praticas
- Nao registre dados sensiveis no `payload`.
- Use sempre a mesma estrutura de campos para facilitar queries.
- Filtre por `logType=instrumented_call` e `operationName` no Datadog.
- Ajuste o `operationName` para refletir a acao real do app.

## Configuracao do Firebase
Preencha `FirebaseConfig.ts` com suas credenciais antes de rodar o app.

## Observacoes sobre RUM
O app inicializa o Datadog SDK em `App.tsx`. Os logs enviados por `DdLogs` podem ser correlacionados com a sessao RUM, permitindo analisar comportamento do usuario junto das chamadas do Firebase.
