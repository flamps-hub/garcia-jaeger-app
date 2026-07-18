# Atualização automática do programa (à distância)

Quando você lançar uma versão nova, o programa nas máquinas dos funcionários se
atualiza **sozinho**: baixa a nova versão, substitui a antiga e reabre já
atualizado. Se a nova versão falhar ao abrir, ele **volta sozinho** para a
anterior (rollback). Você publica as versões no **GitHub** (grátis).

## Como funciona (resumo)

1. Ao abrir, o programa lê um arquivo `versao.json` que você publica no GitHub.
2. Se a versão publicada for maior que a instalada, aparece um aviso e, ao clicar
   em **Atualizar agora** (ou automaticamente), ele baixa o novo `.exe`, confere a
   integridade e agenda a troca.
3. O programa fecha, um pequeno script troca o `.exe` (guardando o anterior como
   `.bak`) e reabre a versão nova.
4. A nova versão, ao abrir com sucesso, apaga o `.bak`. Se ela **não** abrir, na
   tentativa seguinte o `.bak` é restaurado automaticamente.

Nada é executado além de baixar o `.exe` do endereço que **você** publicou.

## Pré-requisito do build

O programa precisa ser gerado em **arquivo único** (`--onefile`), como já está no
guia do executável — é isso que permite trocar um `.exe` só. Inclua o
`atualizador.py` junto com os demais arquivos.

## Passo 1 — Criar o repositório no GitHub

1. Crie uma conta em https://github.com (grátis) e um repositório, por exemplo
   `garcia-jaeger-app` (pode ser público).
2. Nesse repositório você vai manter dois itens: o **instalador/.exe** de cada
   versão (em *Releases*) e o arquivo **`versao.json`**.

## Passo 2 — Publicar uma versão

1. Gere o `.exe` novo (com o número de versão novo — veja o Passo 4).
2. No GitHub, crie uma **Release** e anexe o `.exe`. Copie o link direto do arquivo
   (algo como `https://github.com/SEU-USUARIO/garcia-jaeger-app/releases/download/v1.1.0/Garcia-e-Jaeger-Prospector.exe`).
3. Crie/atualize o `versao.json` na raiz do repositório:

```json
{
  "versao": "1.1.0",
  "url": "https://github.com/SEU-USUARIO/garcia-jaeger-app/releases/download/v1.1.0/Garcia-e-Jaeger-Prospector.exe",
  "notas": "Nova aba de Direito Médico e correções na Proposta PJ.",
  "sha256": "COLE_AQUI_O_HASH_DO_EXE"
}
```

O campo **`sha256`** é opcional, mas recomendado: é uma "impressão digital" do
arquivo, que o programa confere antes de instalar (garante que o download não veio
corrompido ou adulterado). Para gerá-lo no Windows:

```
certutil -hashfile "Garcia-e-Jaeger-Prospector.exe" SHA256
```

Copie o código exibido para o campo `sha256`.

4. Pegue o link **raw** do `versao.json`:
   `https://raw.githubusercontent.com/SEU-USUARIO/garcia-jaeger-app/main/versao.json`.

## Passo 3 — Apontar o programa para o versao.json (uma vez por máquina)

1. No programa: **Configuração → Atualizações do programa**.
2. Cole a URL **raw** do `versao.json` e clique **Salvar URL**.
3. **Verificar agora** para testar.

Feito isso, cada máquina passa a verificar e se atualizar sozinha ao abrir.

## Passo 4 — O número da versão

No `app.py`, no início, existe:

```python
APP_VERSAO = "1.0.0"
```

Antes de gerar um novo `.exe`, **aumente** esse número (ex.: `1.1.0`). É ele que o
programa compara com o `versao.json`. Se o `versao.json` tiver um número maior, a
atualização dispara.

## Segurança e transparência

- A atualização só roda no programa **instalado** (`.exe`); em teste/desenvolvimento
  não faz nada.
- O programa guarda a versão anterior como `.bak` até a nova abrir com sucesso —
  é o que permite o **rollback** automático.
- O `versao.json` só diz "existe versão nova, aqui está o link". Ele não executa
  nada; o programa apenas baixa o `.exe` do link e confere o `sha256`.
- Sem **assinatura digital** (certificado pago), o Windows/antivírus pode alertar na
  primeira execução ou na troca. Com poucas máquinas de confiança, basta liberar o
  programa no antivírus uma vez. Se quiser eliminar os avisos, é possível assinar o
  executável com um certificado (a partir de ~R$ 400/ano) — podemos ver isso depois.
- Regras tributárias (`regras.json`) e a lista de CNAEs já se atualizam pela internet
  sem novo instalador. A atualização do programa é para quando mudam telas/funções.

---

## Proposta PJ — busca de clientes

Na aba **Geração → Proposta PJ** agora há uma **lupa** para buscar clientes já
cadastrados (por nome, CPF/CNPJ ou e-mail) e trazer os dados para a proposta com um
clique. Além disso, na aba **Prospecção**, cada empresa encontrada tem o atalho
**"Proposta PJ →"**, que leva a razão social e o CNPJ daquela empresa direto para a
Proposta PJ.
