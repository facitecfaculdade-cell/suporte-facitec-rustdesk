# Passo a passo — compilar no GitHub (Windows .exe + Android .apk)

## O que já está pronto
- Servidor (`suporte.facitecfaculdade.com.br`) e a logo **embutidos no código**.
- Workflow enxuto **`FACITEC Build (Windows + APK)`** já incluído — compila só
  **Windows x64** e **APK Android arm64** (rápido e barato).
- Os gatilhos automáticos (nightly agendado e CI em cada push) foram **desligados**,
  então **nada roda sozinho** gastando seus minutos.
- **Não precisa configurar nenhum secret.**
- O `hbb_common` foi incluído como código normal e o `.gitmodules` foi removido,
  então o checkout do GitHub **não sobrescreve** suas alterações do servidor.

---

## Passo 1 — Criar o repositório
1. Entre em **github.com** e clique em **New** (novo repositório).
2. Nome: por exemplo `suporte-facitec-rustdesk`.
3. Escolha **Private** ou **Public** (veja a nota de custo no fim).
4. **Não** marque "Add a README". Clique em **Create repository**.

## Passo 2 — Enviar o código (via Git)
O projeto tem milhares de arquivos, então o upload pela web não dá conta — use Git.
No PC, com o **Git instalado**, abra o terminal **dentro da pasta descompactada**
(a que contém o `Cargo.toml`) e rode:

```bash
git init
git add .
git commit -m "Suporte FACITEC: servidor + logo"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/suporte-facitec-rustdesk.git
git push -u origin main
```

- Troque `SEU_USUARIO` e o nome do repositório.
- Quando pedir **senha**, use um **Personal Access Token** (não a senha da conta):
  github.com → *Settings* → *Developer settings* → *Personal access tokens* →
  *Generate new token*, marque o escopo **`repo`**, gere e cole o token no lugar da senha.

## Passo 3 — Rodar o build
1. No repositório, abra a aba **Actions**.
2. Se aparecer um aviso para habilitar workflows, clique em **"I understand… enable"**.
3. No menu à esquerda, clique em **"FACITEC Build (Windows + APK)"**.
4. Clique em **"Run workflow"** (botão à direita) → **"Run workflow"** de novo para confirmar.
5. Aguarde. O Windows leva ~30–60 min; o Android roda em paralelo, no Linux.

## Passo 4 — Baixar os arquivos
Quando terminar (✓ verde):
1. Clique na execução que rodou.
2. Role até a seção **"Artifacts"** no fim da página.
3. Baixe:
   - **`rustdesk-unsigned-windows-x86_64`** → dentro está o **.exe / instalador** do Windows.
   - **`rustdesk-<versão>-aarch64-apk`** → dentro está o **.apk** do Android.
4. O `.exe` não é assinado com certificado, então o **SmartScreen** do Windows pode
   avisar na primeira execução ("Mais informações → Executar assim mesmo"). É normal
   para build próprio; depois dá para assinar com um certificado seu, se quiser.
5. O `.apk` é assinado com chave **debug** (normal para distribuição interna). No
   celular, habilite "instalar de fontes desconhecidas" para instalar.

---

## Nota sobre custo de minutos
- **Repositório privado:** o plano grátis dá **2.000 min/mês**, e o runner do Windows
  conta em **dobro**. Dá para fazer vários builds, mas fique de olho no uso em
  *Settings → Billing*.
- **Repositório público:** os builds são **grátis e ilimitados** nos runners padrão.
  Aqui é seguro deixar público: o código só contém a **chave pública** do servidor
  (feita para ficar nos clientes) e o endereço público — **nenhum segredo**. Se quiser
  economizar minutos, público é o caminho.

## Quer os outros sistemas depois?
O workflow completo (`flutter-build.yml`) já traz Linux, macOS e iOS — eles só estão
**desativados** com `if: ${{ false }}`. Para ligar um deles, é só remover essa linha
do job correspondente. Posso fazer isso para você quando precisar.

## Se algo falhar
Abra o job vermelho e veja qual passo quebrou. Causas comuns: cache do vcpkg na
primeira execução (só rodar de novo costuma resolver) ou versão do Flutter. Me mande
a mensagem de erro que eu ajusto.
