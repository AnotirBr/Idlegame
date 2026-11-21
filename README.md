# Masmorra Idle: Crônicas Pixel

RPG idle em pixel art 2D: 8 classes, 16 subclasses, 128 habilidades, 14 regiões, masmorras, Torre Infinita, pets, visuais, bestiário, runas, guildas e chat.

## Estrutura

```
www/index.html                    o jogo inteiro (um arquivo só)
capacitor.config.json             configuração do app Android
package.json                      dependências do Capacitor
.github/workflows/gerar-apk.yml   gera o APK a cada envio para a main
.github/workflows/github-pages.yml publica o jogo no navegador
COMO_GERAR_APK.md                 guia completo (Firebase, notificações, compras)
```

## Baixar o APK

Cada envio para a branch `main` dispara o workflow **Gerar APK**. Em uns 5 minutos o arquivo aparece em:

- **Releases** (lado direito da página do repositório): baixe `masmorra-idle.apk` direto no celular e instale. O Android vai pedir para permitir "instalar apps desconhecidos".
- **Actions › Gerar APK › execução mais recente › Artifacts**, como alternativa.

Para gerar de novo sem mudar nada: **Actions › Gerar APK › Run workflow**.

## Jogar no navegador (GitHub Pages)

Uma vez só: **Settings › Pages › Build and deployment › Source: GitHub Actions**. Depois disso, o jogo fica em `https://anotirbr.github.io/Idlegame/` e atualiza a cada envio.

## Modo online (chat, ranking, guildas, amigos)

No APK e no GitHub Pages, o modo online precisa do Firebase. Cole a configuração do seu projeto em `www/index.html`, no lugar de `const FIREBASE_CONFIG=null;`, seguindo o passo 2 do `COMO_GERAR_APK.md`.

## Atualizar o jogo

Troque o `www/index.html` pela versão nova e faça o commit na `main`. O APK e o site são gerados de novo automaticamente.
