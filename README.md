# Reymaster LP B2B RD Station

Repositorio publico para compartilhar a landing page B2B da Reymaster e sua pagina de obrigado.

## Paginas

- `index.html`: landing page principal.
- `obrigado.html`: pagina de obrigado para redirecionamento apos envio do formulario.

## Assets incluidos

As logos usadas na secao de marcas estao em `assets/marcas/`.

## Tracking: GTM, Google Ads, Meta Ads e Tintim

Antes de publicar, substitua `GTM-XXXXXXX` no `index.html` e no `obrigado.html` pelo ID real do container do Google Tag Manager. Dentro do GTM, use estes eventos de dataLayer:

- `whatsapp_click`: dispara quando qualquer link com `data-wa` e link rastreavel da Tintim e clicado. Use para Google Ads conversao de clique em WhatsApp e Meta Pixel `Contact`.
- `lead_form_submit`: dispara na pagina `obrigado.html`, quando o RD Station redireciona apos envio do formulario. Use para Google Ads conversao principal e Meta Pixel `Lead`, se essa conversao nao estiver duplicada por outra integracao do RD.
- `cotacao_cta_click`: mede cliques em CTAs que levam ao formulario, sem marcar lead.

No `index.html`, substitua os placeholders `https://SEU-LINK-TINTIM-*` pelos links rastreaveis do Tintim. As mensagens padrao ficam em `TINTIM_MESSAGES` e sao aplicadas no parametro `text` do link rastreavel.

Para campanhas Meta Ads, inclua o parametro `TINTIM_FBID` nas UTMs/parametros da campanha, conforme orientacao da Tintim.

## Botoes de WhatsApp

- Todo botao de WhatsApp usa o atributo `data-wa="<posicao>"` (hero, form, float, cta, footer, thankyou). O JS troca o `href` pelo link rastreavel da Tintim correspondente em `TINTIM_LINKS` (nunca monte `wa.me` manual: e o redirect da Tintim que preserva a atribuicao de campanha).
- Ha um botao flutuante (`.wa-float`, `data-wa="float"`) fixo no canto inferior direito.
- `buildTintimUrl()` repassa automaticamente os parametros de campanha que chegam na URL da LP (`utm_*`, `gclid`, `gbraid`, `wbraid`, `fbclid`, `ttclid`, `msclkid`) para dentro do link da Tintim, alem de aplicar a mensagem padrao em `text`. Isso reforca a atribuicao em campanhas que caem direto na LP, sem depender so do cookie do pixel.
- Enquanto os links forem placeholders (`SEU-LINK-TINTIM-*`), o `href` fica inalterado e nada quebra.
