# Suporte FACITEC — RustDesk customizado

Este código já está com **servidor próprio integrado** e a **logo trocada**. Abaixo está exatamente o que foi alterado e como gerar o build.

---

## 1. Servidor integrado

As credenciais foram fixadas em tempo de compilação em
`libs/hbb_common/src/config.rs`:

| Campo | Valor |
|-------|-------|
| Servidor de ID (Rendezvous) | `suporte.facitecfaculdade.com.br` |
| Servidor de Relay | `suporte.facitecfaculdade.com.br` |
| Key (chave pública) | `odTpR6XvsLZKPDf9frhfosAdjaBh8CAm++pT9ZhzZWA=` |

Alterações feitas (4 pontos no `config.rs`):

1. `RENDEZVOUS_SERVERS` → `&["suporte.facitecfaculdade.com.br"]`
2. `RS_PUB_KEY` → a key da FACITEC
3. `PROD_RENDEZVOUS_SERVER` (valor inicial) → o host da FACITEC
4. `DEFAULT_SETTINGS` já vem pré-preenchido com `custom-rendezvous-server`,
   `relay-server` e `key`, de modo que a tela **Configurações → Rede** já abre
   com tudo preenchido.

O **API server** não precisa ser fixado: quando fica vazio, o RustDesk deriva
automaticamente `http://suporte.facitecfaculdade.com.br:21114` (porta padrão do
hbbs). O relay também é entregue pelo próprio hbbs, então a conexão funciona sem
configuração no cliente.

> ⚠️ **IMPORTANTE — não rode `git submodule update`.**
> O `libs/hbb_common` normalmente é um submódulo Git e veio **vazio** no seu zip
> original. Ele já foi incluído aqui como código normal (com as edições acima
> aplicadas). Se você rodar `git submodule update --init`, o `config.rs` será
> **sobrescrito** e você perde a integração do servidor. Se precisar reinstalar o
> submódulo, reaplique os 4 pontos acima.

---

## 2. Logo trocada

O emblema (círculo azul + cubo **FT** amarelo) foi recortado da sua arte e
regenerado em **todos** os formatos/tamanhos de ícone das plataformas:

- **Windows:** `flutter/windows/runner/resources/app_icon.ico` e `res/icon.ico`
  (multi-tamanho 16→256), `res/tray-icon.ico`.
- **Linux:** `res/icon.png`, `res/32x32.png`, `res/64x64.png`,
  `res/128x128.png`, `res/128x128@2x.png` e `res/scalable.svg` (ícone do app no
  hicolor).
- **Android:** `ic_launcher`, `ic_launcher_round` (com máscara circular),
  `ic_launcher_foreground` (ícone adaptativo, com área de segurança) e
  `ic_stat_logo` (silhueta branca da notificação) em todas as densidades;
  fundo do ícone adaptativo mudado para o azul-marinho da marca (`#0b1429`).
- **iOS:** todo o `AppIcon.appiconset` (sem canal alpha, como o iOS exige).
- **macOS:** `res/mac-icon.png`.
- **Dentro do app:** `flutter/assets/icon.png` (ícone quadrado) e
  `flutter/assets/logo.png` (logo do cabeçalho da tela inicial), ambos com fundo
  transparente para ficar bem no tema claro e escuro.

Também deixei a arte completa em `res/facitec-logo-full.png` caso queira reusar
em splash/tela "sobre".

> Observação: para nitidez máxima, o ideal é gerar os ícones a partir de um
> **vetor** do emblema. Estes foram gerados a partir do PNG 1254×1254 enviado,
> o que é ótimo até 256 px (tamanho máximo de ícone). Se um dia tiver o SVG do
> emblema, dá para regenerar com mais nitidez.

---

## 3. Como compilar

O RustDesk (Flutter + Rust + bibliotecas nativas via vcpkg) **não** compila num
ambiente simples — precisa do toolchain completo da plataforma-alvo. Escolha um
dos caminhos:

### Caminho A — Windows (cliente para quem recebe suporte)

Pré-requisitos:
- **Visual Studio 2022** com "Desktop development with C++".
- **Rust** (rustup) — canal estável recente.
- **LLVM/Clang** (defina `LIBCLANG_PATH`).
- **Flutter 3.22.x** no PATH.
- **vcpkg** com as libs nativas:
  ```
  git clone https://github.com/microsoft/vcpkg
  vcpkg\bootstrap-vcpkg.bat
  set VCPKG_ROOT=C:\caminho\vcpkg
  vcpkg\vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static
  ```
- **flutter_rust_bridge_codegen** (versão 1.80.1, como no projeto):
  ```
  cargo install flutter_rust_bridge_codegen --version 1.80.1
  ```

Build:
```
# na raiz do projeto
flutter pub get
flutter_rust_bridge_codegen ^
  --rust-input src/flutter_ffi.rs ^
  --dart-output flutter/lib/generated_bridge.dart ^
  --c-output flutter/macos/Runner/bridge_generated.h

python build.py --flutter
```
O instalador/executável sai em `flutter/build/windows/` (ou na pasta indicada
ao final do `build.py`).

### Caminho B — GitHub Actions (mais simples, sem montar ambiente)

1. Suba este código para um repositório **privado** seu no GitHub.
2. Como o servidor já está fixo no `config.rs`, você **não precisa** configurar
   os secrets `RENDEZVOUS_SERVER`/`RS_PUB_KEY` — mas se preferir usá-los, o
   workflow `.github/workflows/playground.yml` já os lê.
3. Rode o workflow (Actions → *playground* → *Run workflow*). Os binários das
   plataformas saem como *artifacts*.

### Caminho C — Android (APK)

Com Flutter + NDK configurados:
```
flutter pub get
python build.py --flutter --android   # ou: cd flutter && flutter build apk --release
```

Documentação oficial de build (dependências detalhadas por SO):
https://rustdesk.com/docs/en/dev/build/

---

## 4. Como conferir que ficou certo

Depois de instalar o build, abra **Configurações → Rede**: os campos
*ID Server*, *Relay Server* e *Key* já devem aparecer preenchidos com os dados da
FACITEC. Faça um teste de conexão entre duas máquinas — o ID de 9 dígitos é
emitido pelo **seu** servidor `suporte.facitecfaculdade.com.br`.

---

## 5. Opcional — trocar o nome "RustDesk" por "Suporte FACITEC"

Não fiz isso porque você pediu só a logo, e o nome interno (`APP_NAME`) afeta
pastas de config, nomes de serviço e pipes de IPC. Se quiser mudar o nome
**exibido** com segurança, os pontos principais são:
- `libs/hbb_common/src/config.rs` → `APP_NAME` (linha ~72).
- `flutter/pubspec.yaml` (campo `name`/descrição) e os manifestos de cada
  plataforma (`AndroidManifest.xml`, `Info.plist`, `Runner.rc`, `.desktop`).

Recomendo fazer isso num passo separado e testar, pois mexe em mais lugares.
