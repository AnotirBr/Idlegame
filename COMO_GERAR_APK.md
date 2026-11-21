# Runa Eterna: como gerar o APK com modo online

Este guia transforma o `index.html do jogo` num app Android com chat, ranking, guildas, amigos e chefe de guilda funcionando de verdade, além de notificações.

Você vai precisar de:
- um computador com **Node.js** (versão 18 ou mais nova);
- o **Android Studio** instalado;
- uma conta Google para o **Firebase** (o plano gratuito basta para começar).

---

## 1. Criar o projeto do app (Capacitor)

No terminal:

```bash
mkdir runa-eterna && cd runa-eterna
npm init -y
npm install @capacitor/core @capacitor/cli @capacitor/android @capacitor/local-notifications
npx cap init "Runa Eterna" com.seunome.runaeterna --web-dir www
mkdir www
```

Copie o arquivo do jogo para dentro da pasta `www`, com o nome `index.html`:

```bash
cp /caminho/para/index.html do jogo www/index.html
```

---

## 2. Ligar o servidor (Firebase)

1. Acesse **console.firebase.google.com** e crie um projeto.
2. Clique em **Adicionar app › Web (</>)**, dê um nome e copie o objeto `firebaseConfig` que aparece.
3. Abra `www/index.html` num editor, procure por `const FIREBASE_CONFIG=null;` e troque `null` pela configuração copiada. Exemplo:

```js
const FIREBASE_CONFIG={apiKey:"...",authDomain:"...",projectId:"...",storageBucket:"...",messagingSenderId:"...",appId:"..."};
```

4. Em **Authentication › Método de login**, ative **E-mail/senha**. Os jogadores criam conta dentro do jogo.
5. Em **Firestore Database**, crie o banco (modo produção) e cole estas **regras**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    function signed() { return request.auth != null; }
    function gm() { return signed() && exists(/databases/$(db)/documents/admins/$(request.auth.uid)); }
    function banDoc() { return get(/databases/$(db)/documents/bans/$(request.auth.uid)).data; }
    function hasBan() { return exists(/databases/$(db)/documents/bans/$(request.auth.uid)); }
    function banned() { return hasBan() && banDoc().until > request.time.toMillis(); }
    function hardBan() { return banned() && banDoc().get('type', 'mute') == 'ban'; }
    match /admins/{uid}  { allow read: if signed(); allow write: if false; }
    match /saves/{uid}   { allow read, write: if signed() && request.auth.uid == uid; allow delete: if gm(); }
    match /players/{uid} {
      allow read: if signed();
      allow write: if signed() && request.auth.uid == uid && !hardBan();
      allow delete: if gm();
    }
    match /chat/{id} {
      allow read: if signed();
      allow create: if signed() && !banned() && request.resource.data.u == request.auth.uid && request.resource.data.t.size() <= 140;
      allow delete: if gm() || (signed() && resource.data.u == request.auth.uid) || (signed() && resource.data.ts < request.time.toMillis() - 172800000);
    }
    match /bans/{uid}    { allow read: if signed(); allow write: if gm(); }
    match /config/{id}   { allow read: if signed(); allow write: if gm(); }
    match /grants/{id}   { allow read: if signed(); allow write: if gm(); }
    match /gmlog/{id}    { allow read, write: if gm(); }
    match /guilds/{id}   { allow read: if signed(); allow write: if signed() && !hardBan(); allow delete: if gm() || (signed() && !hardBan()); }
    match /gboss/{id}    { allow read, write: if signed() && !hardBan(); }
    match /gdmg/{id}     { allow read: if signed(); allow write: if signed() && !hardBan() && request.resource.data.u == request.auth.uid; }
    match /market/{id} {
      allow read: if signed();
      allow create: if signed() && !hardBan() && request.resource.data.s == request.auth.uid && request.resource.data.p >= 5;
      allow delete: if signed() && !hardBan();
    }
    match /sales/{id} {
      allow read: if signed() && resource.data.to == request.auth.uid;
      allow create: if signed() && !hardBan();
      allow delete: if signed() && resource.data.to == request.auth.uid;
    }
    match /gifts/{id} {
      allow read: if signed();
      allow create: if signed() && !hardBan() && request.resource.data.from == request.auth.uid;
      allow delete: if signed() && resource.data.to == request.auth.uid;
    }
  }
}
```

6. **Game Master.** Crie sua conta no jogo, abra Social e toque em *copiar* ao lado do seu ID. No Firestore, crie a coleção `admins`, um documento com esse ID e o campo `gm` (booleano) = `true`. Ao entrar de novo, a aba **GM** aparece.

7. **Índices.** Na primeira vez que o chat e o chefe de guilda rodarem, o Firestore vai pedir dois índices compostos: `chat` com (`ch`, `ts` decrescente) e `gdmg` com (`g`, `wk`). O erro aparece no console do app com um link. Abra o link e o índice é criado com um clique.

Pronto: com `FIREBASE_CONFIG` preenchido, o jogo se conecta sozinho. Sem ele, mostra "recursos online indisponíveis".

---

## 3. Gerar o APK

```bash
npx cap add android
npx cap sync
npx cap open android
```

No Android Studio:
- **APK para testar:** menu **Build › Build Bundle(s) / APK(s) › Build APK(s)**. O arquivo fica em `android/app/build/outputs/apk/debug/`.
- **Para a Play Store:** **Build › Generate Signed Bundle / APK › Android App Bundle**. Crie a chave de assinatura e guarde-a bem.

Sempre que atualizar o `index.html`, rode `npx cap sync` de novo.

---

## 4. Notificações

Com o plugin `@capacitor/local-notifications` instalado, o jogo já agenda sozinho, ao ser minimizado:
- **recompensas offline cheias:** após 8 horas, mais as horas da Runa da Eternidade;
- **chaves de masmorra recarregadas;**
- **bolsa quase cheia.**

No Android 13 ou mais novo, o jogo pede permissão de notificação na primeira vez.

---

## 5. Compra de gemas com dinheiro real (opcional)

Os botões da loja de gemas chamam `window.BILLING.buy(quantidade)` se essa função existir. Para cobrar de verdade, use o **Google Play Billing**, por exemplo com o plugin `cordova-plugin-purchase` ou o RevenueCat:

1. Crie os produtos (80, 450, 1000 e 2500 gemas) no **Play Console › Monetizar › Produtos no app**.
2. No `index.html`, defina `window.BILLING = { buy: async (gemas) => { /* abre a compra no plugin; quando confirmar: */ S.gems += gemas; save(); } }`.
3. Para evitar fraude, valide a compra num servidor, por exemplo com Firebase Cloud Functions, antes de entregar as gemas.

Sem isso, os botões só mostram um aviso e **nenhuma cobrança é feita**.

---

## Observações

- As fontes pixeladas vêm do Google Fonts. Sem internet o jogo usa uma fonte reserva.
- O progresso fica salvo no próprio aparelho. Trocar de celular ainda não leva o save junto (dá para fazer depois com o Firebase).
