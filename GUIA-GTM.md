# Guia de configuração do Google Tag Manager — Reymaster LP B2B

Este guia mostra como ligar os eventos que a landing page já dispara no `dataLayer`
às conversões do **Google Ads** e do **Meta Ads (Facebook Pixel)**, mais a instalação
do **pixel da Tintim** via GTM.

> Pré-requisito no site: o ID do container precisa estar preenchido no lugar de `GTM-XXXXXXX`
> em `index.html` (linhas 10 e 299) e `obrigado.html` (linhas 20 e 102). Enquanto for `GTM-XXXXXXX`,
> o GTM **não carrega** (é uma trava proposital).

---

## 1. Eventos que a LP envia para o dataLayer

A página já empurra estes eventos sozinha — você só precisa "ouvi-los" no GTM.

| Evento (`event`) | Quando dispara | Página | Campos disponíveis |
|---|---|---|---|
| `whatsapp_click` | Clique em qualquer botão de WhatsApp (hero, form, float, footer, thankyou) | LP e obrigado | `lead_type`, `cta_position`, `cta_text`, `link_url`, `tintim_link` |
| `cotacao_cta_click` | Clique em CTA que leva ao formulário (header, hero, steps, band, reviews) | LP | `event_category`, `cta_position`, `cta_text` |
| `lead_form_submit` | Carregamento da página de obrigado (lead enviado no RD) | obrigado | `lead_type`, `page_type`, `form_name` |

**Mapa de conversão recomendado:**

- `whatsapp_click` → Google Ads (conversão de clique no WhatsApp) + Meta `Contact`
- `lead_form_submit` → Google Ads (conversão principal / lead) + Meta `Lead`
- `cotacao_cta_click` → só engajamento (opcional; não marque como lead)

---

## 2. Variáveis a criar no GTM

Em **Variáveis → Variáveis definidas pelo usuário → Nova → Variável da camada de dados**.
O "Nome da variável da camada de dados" tem que ser **idêntico** ao campo do dataLayer.

| Nome da variável (sugestão) | Nome da camada de dados |
|---|---|
| `dlv.cta_position` | `cta_position` |
| `dlv.cta_text` | `cta_text` |
| `dlv.lead_type` | `lead_type` |
| `dlv.link_url` | `link_url` |
| `dlv.form_name` | `form_name` |

Ative também as variáveis internas em **Variáveis → Configurar**: `Page URL`, `Page Path`,
`Click URL`, `Click Element` (úteis para depurar).

---

## 3. Acionadores (Triggers)

Em **Acionadores → Novo → Evento personalizado**.

### 3.1 — `trg_whatsapp_click`
- Tipo: **Evento personalizado**
- Nome do evento: `whatsapp_click`
- Dispara em: Todos os eventos personalizados

### 3.2 — `trg_lead_form_submit`
- Tipo: **Evento personalizado**
- Nome do evento: `lead_form_submit`
- Dispara em: Todos os eventos personalizados

### 3.3 — `trg_cotacao_cta_click` (opcional)
- Tipo: **Evento personalizado**
- Nome do evento: `cotacao_cta_click`

### 3.4 — `trg_all_pages` (para o pixel da Tintim e tags base)
- Tipo: **Inicialização / All Pages** (use "Initialization - All Pages" para tags de pixel).

---

## 4. Pixel da Tintim (via GTM)

> Como o pixel é gerenciado pelo GTM, **não** cole o pixel no HTML também — senão dispara 2x.
> O slot no `index.html` (linha 19) deve ficar vazio.

1. **Tags → Nova → HTML personalizado**.
2. Cole o código do pixel da Tintim (painel Tintim → *Informações do Cliente*).
3. Acionador: **`trg_all_pages`** (Initialization - All Pages).
4. Nome sugerido: `Tintim - Pixel`.

Isso garante que a Tintim capture `gclid`/`fbclid`/`utm` na chegada e amarre o lead à campanha.
A LP já reforça isso repassando esses parâmetros para o link rastreável no clique.

---

## 5. Google Ads

### 5.1 — Tag base (Google Ads Conversion Linker)
- **Tags → Nova → Vinculador de conversão (Conversion Linker)**.
- Acionador: **All Pages**.
- Necessário para o Google Ads reconhecer os cliques (gclid) corretamente.

### 5.2 — Conversão de clique no WhatsApp
1. No Google Ads: **Metas → Conversões → Nova ação de conversão → Importar/Manual** e crie
   uma conversão (ex.: "Clique WhatsApp"). Anote o **ID de conversão** e o **Rótulo**.
2. No GTM: **Tags → Nova → Acompanhamento de conversões do Google Ads**.
   - ID de conversão: `AW-XXXXXXXXX`
   - Rótulo de conversão: `xxxxxxxxxxx`
   - Acionador: **`trg_whatsapp_click`**
   - Nome: `GAds - Conv WhatsApp`

### 5.3 — Conversão principal (lead)
1. Crie no Google Ads a conversão "Lead cotação" (anote ID + Rótulo).
2. No GTM: nova tag de conversão do Google Ads.
   - Acionador: **`trg_lead_form_submit`**
   - Nome: `GAds - Conv Lead`

> ⚠️ Só ative `GAds - Conv Lead` se o RD Station já **não** estiver enviando essa conversão
> por outra integração — para não contar o mesmo lead duas vezes.

---

## 6. Meta Ads (Facebook Pixel)

### 6.1 — Pixel base
- **Tags → Nova → HTML personalizado** com o código base do Meta Pixel (`fbq('init', ...)` + `PageView`).
- Acionador: **All Pages**.
- Nome: `Meta - Pixel Base`.
- (Alternativa: usar o modelo da galeria de templates do GTM "Facebook Pixel".)

### 6.2 — Evento `Contact` (clique WhatsApp)
- **Tags → Nova → HTML personalizado**:
  ```html
  <script>
    fbq('track', 'Contact', {
      content_name: {{dlv.cta_position}}
    });
  </script>
  ```
- Acionador: **`trg_whatsapp_click`**
- Nome: `Meta - Contact`

### 6.3 — Evento `Lead` (página de obrigado)
- **Tags → Nova → HTML personalizado**:
  ```html
  <script>
    fbq('track', 'Lead', {
      content_name: {{dlv.form_name}}
    });
  </script>
  ```
- Acionador: **`trg_lead_form_submit`**
- Nome: `Meta - Lead`

> Ative o `Meta - Lead` apenas se o RD não estiver enviando o mesmo Lead por outra via.

---

## 7. Ordem de carregamento (importante)

Para os eventos de pixel funcionarem, o pixel base precisa carregar **antes** dos eventos.
- Garanta que `Meta - Pixel Base` e `Tintim - Pixel` usem **All Pages / Initialization**.
- Nas tags `Meta - Contact` / `Meta - Lead`, em **Configurações avançadas → Sequência de tags**,
  você pode definir "Disparar uma tag antes desta: Meta - Pixel Base" se houver corrida de timing.

---

## 8. Checklist de teste (modo Visualizar / Preview do GTM)

1. Entre em **Visualizar** no GTM e abra a LP (use `?gclid=TESTE123&utm_source=google`).
2. **All Pages**: confirme que `Tintim - Pixel`, `Conversion Linker` e `Meta - Pixel Base` dispararam.
3. Clique num botão de WhatsApp → no painel do Preview deve aparecer o evento `whatsapp_click`
   e as tags `GAds - Conv WhatsApp` + `Meta - Contact` disparando.
4. Verifique que o `href` do botão levou os parâmetros (`?gclid=TESTE123...`) para o link Tintim.
5. Envie um lead de teste e chegue na página de obrigado → confirme `lead_form_submit`
   e as tags de Lead.
6. Confira no **Google Ads** (conversões recentes) e no **Meta Events Manager** (Test Events)
   se os eventos chegaram. Na **Tintim**, confira se o lead foi atribuído à campanha.
7. Só então **Enviar / Publicar** o container no GTM.

---

## 9. Resumo do que vai em cada lugar

| Onde | O quê |
|---|---|
| **HTML do site** | ID do GTM (4 lugares) + links rastreáveis da Tintim em `TINTIM_LINKS` |
| **GTM** | Pixel da Tintim, Conversion Linker, Meta Pixel base, tags de conversão GAds/Meta, triggers |
| **Painel Tintim** | Número de WhatsApp + mensagem padrão de cada link rastreável |
| **Google Ads / Meta** | Criar as ações de conversão e pegar IDs/labels para colar no GTM |
