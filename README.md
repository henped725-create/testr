# Inject Baileys --- README de uso

Documentação prática do `inject.js`, baseada nas funções e atalhos
realmente implementados no arquivo.

> O injector modifica `socket.sendMessage()` e adiciona vários helpers
> ao socket do Baileys. Ele tenta carregar, nesta ordem: `baileys`,
> `@whiskeysockets/baileys`, `@adiwajshing/baileys` e `nightwaveconect`.

## Instalação

Coloque `inject.js` no projeto e injete o socket **uma vez**, depois
de criá-lo:

``` js
const { injectButtons } = require("./inject");

const sock = makeWASocket({
  // suas configs...
});

injectButtons(sock);
```

Também funciona assim:

``` js
const inject = require("./inject");
inject.injectButtons(sock);
```

Para mídia local usada dentro de `richResponse`/Rich UI, o arquivo usa
upload temporário e precisa destas dependências:

``` bash
npm i axios form-data
```

O restante depende do Baileys instalado no projeto.

------------------------------------------------------------------------

# 1. `sendMessage()` normal continua funcionando

Depois da injeção, mensagens que não pertencem aos formatos especiais
continuam indo para o `sendMessage` original:

``` js
await sock.sendMessage(jid, {
  text: "Olá!"
});
```

``` js
await sock.sendMessage(jid, {
  image: { url: "https://site.com/imagem.jpg" },
  caption: "Imagem"
});
```

O injector intercepta apenas formatos que ele reconhece.

------------------------------------------------------------------------

# 2. Prefixo automático

Vários componentes aceitam `prefix`.

``` js
await sock.sendMessage(jid, {
  prefix: "/",
  text: "Escolha:",
  buttons: [
    { text: "Menu", id: "menu" },
    { text: "Ping", id: "ping" }
  ]
});
```

Os IDs viram:

``` text
/menu
/ping
```

Para impedir o prefixo em um botão/linha:

``` js
{
  text: "Sem prefixo",
  id: "teste",
  usePrefix: false
}
```

Se o ID já começar com o prefixo, ele não é duplicado.

------------------------------------------------------------------------

# 3. Quoted personalizado

O injector aceita `quoted` normal do Baileys e também atalhos.

## Quoted real

``` js
await sock.sendMessage(jid, {
  text: "Resposta",
  quoted: m
});
```

## Quoted de IA aleatória

``` js
await sock.sendMessage(jid, {
  text: "Olá",
  quoted: "ai"
});
```

Pode gerar quote de Meta IA, ChatGPT, LuzIA, Alice AI ou Microsoft
Copilot.

## Quoted de banco aleatório

``` js
await sock.sendMessage(jid, {
  text: "Pagamento",
  quoted: "bank"
});
```

## Quoted de texto falso

``` js
await sock.sendMessage(jid, {
  text: "Resposta",
  quoted: "Mensagem original falsa"
});
```

Também:

``` js
quoted: {
  person: "Texto da pessoa"
}
```

ou:

``` js
quoted: {
  type: "person",
  text: "Texto da pessoa"
}
```

## Quoted de contato

``` js
quoted: {
  type: "contact",
  number: "5511999999999",
  name: "Pedrozz"
}
```

Aliases aceitos para número:

``` text
personNum
number
phone
```

------------------------------------------------------------------------

# 4. Native Flow / botões interativos

Você pode usar diretamente:

``` js
await sock.sendNativeInteractive(jid, {
  text: "Escolha uma opção",
  footer: "Pedrozz Mods",
  title: "Menu",
  subtitle: "Selecione",
  prefix: "/",
  buttons: [
    {
      text: "Ping",
      id: "ping"
    }
  ]
});
```

Ou pelo `sendMessage`:

``` js
await sock.sendMessage(jid, {
  text: "Escolha uma opção",
  footer: "Pedrozz Mods",
  prefix: "/",
  buttons: [
    { text: "Ping", id: "ping" }
  ]
});
```

Campos úteis:

``` js
{
  text,
  caption,
  footer,
  title,
  subtitle,
  image,
  video,
  document,
  fileName,
  mimetype,
  jpegThumbnail,
  buttons,
  interactiveButtons,
  sections,
  prefix,
  mentions,
  contextInfo,
  externalAdReply,
  messageParamsJson,
  limited_time_offer
}
```

Native Flow pode ser enviado até com `buttons: []`.

------------------------------------------------------------------------

# 5. Tipos de botão

## Quick reply

Formato simplificado:

``` js
{
  type: "reply",
  text: "Ping",
  id: "ping"
}
```

Aliases de tipo:

``` text
reply
quick_reply
button
```

Formato Native Flow bruto:

``` js
{
  name: "quick_reply",
  buttonParamsJson: {
    display_text: "Ping",
    id: "ping"
  }
}
```

------------------------------------------------------------------------

## Copiar texto

``` js
{
  type: "copy",
  text: "Copiar código",
  code: "PEDROZZ123"
}
```

Aliases:

``` text
copy
cta_copy
```

Formato bruto:

``` js
{
  name: "cta_copy",
  buttonParamsJson: {
    display_text: "Copiar",
    copy_code: "PEDROZZ123"
  }
}
```

------------------------------------------------------------------------

## Abrir URL

``` js
{
  type: "url",
  text: "Abrir site",
  url: "https://example.com"
}
```

Aliases:

``` text
url
link
cta_url
```

Formato bruto:

``` js
{
  name: "cta_url",
  buttonParamsJson: {
    display_text: "Abrir",
    url: "https://example.com",
    merchant_url: "https://example.com"
  }
}
```

------------------------------------------------------------------------

## Ligação

``` js
{
  type: "call",
  text: "Ligar",
  phone: "+5561999999999"
}
```

Aliases:

``` text
call
phone
cta_call
```

------------------------------------------------------------------------

## Lista / single select

``` js
{
  type: "list",
  text: "Abrir menu",
  title: "Escolha",
  sections: [
    {
      title: "Principal",
      rows: [
        {
          title: "Ping",
          description: "Testar bot",
          id: "ping"
        },
        {
          title: "Menu",
          description: "Abrir menu",
          id: "menu"
        }
      ]
    }
  ]
}
```

Aliases:

``` text
list
select
single_select
```

------------------------------------------------------------------------

# 6. Lista diretamente por `sections`

Se `sections` existir no Native Flow, o injector cria automaticamente um
`single_select`.

``` js
await sock.sendMessage(jid, {
  text: "Menu principal",
  buttonText: "ABRIR",
  prefix: ".",
  sections: [
    {
      title: "Comandos",
      rows: [
        {
          title: "Ping",
          description: "Ver latência",
          id: "ping"
        },
        {
          title: "Perfil",
          id: "perfil"
        }
      ]
    }
  ]
});
```

Uma seção também aceita `options`, `items` ou `buttons` no lugar de
`rows`.

Uma linha aceita:

``` text
title / text / name
description / desc
rowId / id / buttonId
usePrefix
```

------------------------------------------------------------------------

# 7. Limited Time Offer

Helper direto:

``` js
await sock.sendLimitedTimeOffer(jid, {
  text: "Oferta especial",
  url: "https://example.com",
  copyCode: "PROMO10",
  expiresInMs: 15 * 60 * 1000
});
```

Também pelo `sendMessage`:

``` js
await sock.sendMessage(jid, {
  text: "Promoção",
  limited_time_offer: {
    text: "Só por 15 minutos",
    url: "https://example.com",
    copy_code: "PROMO10",
    expiresInMs: 900000
  }
});
```

Expiração aceita:

``` text
expiration_time
expirationTime
expires_at
expiresAt
expiresInMs
expires_in_ms
```

Por padrão a oferta **não cria botão**.

Para criar CTA de URL automaticamente:

``` js
limited_time_offer: {
  text: "Comprar agora",
  url: "https://example.com",
  button: true,
  buttonText: "ABRIR"
}
```

Aliases para ativar:

``` text
button: true
autoButton: true
auto_button: true
```

------------------------------------------------------------------------

# 8. `sendMultiButton()`

Feito para misturar botões simplificados.

``` js
await sock.sendMultiButton(jid, {
  text: "Central",
  footer: "Pedrozz Mods",
  headerTitle: "Menu",
  subtitle: "Escolha",
  prefix: "/",
  image: "./menu.jpg",
  buttons: [
    {
      type: "reply",
      text: "Ping",
      id: "ping"
    },
    {
      type: "copy",
      text: "Copiar PIX",
      code: "123456"
    },
    {
      type: "url",
      text: "Site",
      url: "https://example.com"
    },
    {
      type: "call",
      text: "Ligar",
      phone: "+5561999999999"
    }
  ]
});
```

Também:

``` js
await sock.sendMessage(jid, {
  multiButton: {
    text: "Menu",
    buttons: [
      { type: "reply", text: "Ping", id: "ping" }
    ]
  },
  prefix: "/"
});
```

Alias legado:

``` js
await sock.sendTButton2(jid, {
  text: "Menu",
  buttons: [...]
});
```

------------------------------------------------------------------------

# 9. Carousel

``` js
await sock.sendCarousel(jid, {
  text: "Produtos",
  footer: "Escolha um item",
  prefix: "/",
  cards: [
    {
      title: "Produto 1",
      subtitle: "Card 1",
      body: "Descrição do produto",
      footer: "R$ 10",
      image: "./produto1.jpg",
      buttons: [
        {
          type: "reply",
          text: "Comprar",
          id: "comprar_1"
        },
        {
          type: "url",
          text: "Ver site",
          url: "https://example.com/1"
        }
      ]
    },
    {
      title: "Produto 2",
      body: "Outro produto",
      image: "https://example.com/produto2.jpg",
      buttons: [
        {
          type: "reply",
          text: "Comprar",
          id: "comprar_2"
        }
      ]
    }
  ]
});
```

Também pelo `sendMessage`:

``` js
await sock.sendMessage(jid, {
  carousel: {
    text: "Meu carousel",
    cards: [...]
  },
  prefix: "/"
});
```

Ou simplesmente:

``` js
await sock.sendMessage(jid, {
  text: "Carousel",
  cards: [...]
});
```

Cada card aceita imagem ou vídeo:

``` js
{
  video: "./video.mp4",
  mimetype: "video/mp4",
  jpegThumbnail: buffer
}
```

Por padrão cada card precisa ter botão. Para permitir cards sem botão:

``` js
{
  requireButtons: false,
  cards: [...]
}
```

O máximo padrão é 10 cards e `maxCards` é limitado a 10.

------------------------------------------------------------------------

# 10. Poll com comandos

``` js
await sock.sendPollCommand(jid, {
  name: "Escolha",
  prefix: "/",
  values: [
    {
      text: "Ping",
      id: "ping"
    },
    {
      text: "Menu",
      id: "menu"
    }
  ],
  selectableCount: 1,
  cmd: true
});
```

Também:

``` js
await sock.sendMessage(jid, {
  prefix: "/",
  poll: {
    name: "O que deseja?",
    values: [
      { text: "Ping", id: "ping" },
      { text: "Menu", id: "menu" }
    ],
    cmd: true
  }
});
```

Formato simples:

``` js
poll: {
  name: "Escolha",
  values: ["Ping", "Menu"],
  actions: {
    ping: "ping",
    menu: "menu"
  },
  prefix: "/",
  cmd: true
}
```

Quando o voto é descriptografado, o injector pode emitir eventos
próprios e, com `cmd: true`, gera um `messages.upsert` sintético
contendo o comando escolhido.

O poll precisa de pelo menos 2 opções.

------------------------------------------------------------------------

# 11. Form / Galaxy Form

## Form

``` js
await sock.sendForm(jid, {
  text: "Preencha seus dados",
  fields: [
    {
      type: "TEXT_INPUT",
      label: "Nome"
    },
    {
      type: "TEXT_INPUT",
      label: "Cidade"
    }
  ]
});
```

Forma curta:

``` js
await sock.sendForm(jid, {
  text: "Digite seu nome",
  label: "Nome"
});
```

## Form executando comando

``` js
await sock.sendForm(jid, {
  text: "Configurar perfil",
  prefix: "/",
  command: "perfil",
  cmd: true,
  separator: " ",
  fields: [
    { type: "TEXT_INPUT", label: "Nome" },
    { type: "TEXT_INPUT", label: "Idade" }
  ]
});
```

Ao receber a resposta, o injector pode gerar algo equivalente a:

``` text
/perfil Pedrozz 20
```

em um `messages.upsert` sintético.

## Form2

``` js
await sock.sendForm2(jid, {
  text: "Enviar proposta",
  title: "Plano Premium",
  description: "Dados para continuar",
  fields: [
    { type: "TEXT_INPUT", label: "Nome" }
  ]
});
```

`form2` adiciona `offer_name` e `offer_description`.

## Campos de visibilidade

``` js
await sock.sendForm(jid, {
  text: "Cadastro",
  fullNameVisible: true,
  emailVisible: true,
  phoneNumberVisible: true,
  deliveryAddressVisible: true,
  cpfOrCnpjVisible: true,
  citizenshipCardVisible: false,
  fields: []
});
```

Aliases snake_case também são aceitos:

``` text
full_name_visible
email_visible
phone_number_visible
delivery_address_visible
cpf_or_cnpj_visible
citizenship_card_visible
```

Campos avançados:

``` text
id / flow_id
token / flow_token
flow_message_version / flowMessageVersion
screen
well_version / wellVersion
flow_cta / flowCta
flow_action / flowAction
form_type / formType
data
```

Também é possível:

``` js
await sock.sendMessage(jid, {
  form: {
    text: "Formulário",
    fields: [...]
  }
});
```

ou:

``` js
await sock.sendMessage(jid, {
  form2: {
    text: "Formulário 2",
    fields: [...]
  }
});
```

------------------------------------------------------------------------

# 12. Rich Response

É a parte mais extensa do injector.

``` js
await sock.sendRichResponse(jid, {
  title: "Luminus",
  subtitle: "Sistema online",
  text: "Olá, **mundo**!",
  mutedText: "By Pedrozz Mods"
});
```

Também:

``` js
await sock.sendMessage(jid, {
  richResponse: {
    title: "Luminus",
    text: "Teste"
  }
});
```

Principais campos:

``` js
{
  disclaimer,
  title,
  subtitle,
  image,
  text,
  mutedText,
  code,
  codeLanguage,
  citations,
  links,
  table,
  math,
  html,
  htmlTrustedSources,
  htmlType,
  logo,
  cards,
  reels,
  posts,
  products,
  product,
  productScroll,
  productsScroll,
  markdown,
  blocks,
  scrolls,
  actions,
  footer,
  components,
  sections,
  submessages,
  contextInfo,
  prefix,
  responseId,
  aiForwarded,
  botJid,
  bizNode
}
```

------------------------------------------------------------------------

# 13. Rich title, subtitle, imagem e Markdown

``` js
await sock.sendRichResponse(jid, {
  title: "Título",
  subtitle: "Subtítulo",
  image: "./banner.jpg",
  text: "# Olá\n\nTexto em **Markdown**.",
  mutedText: "Informação secundária"
});
```

Mídia Rich aceita URL, caminho local, `Buffer` ou objeto:

``` js
image: "https://example.com/a.jpg"
```

``` js
image: "./a.jpg"
```

``` js
image: buffer
```

``` js
image: {
  path: "./a.jpg",
  mimetype: "image/jpeg"
}
```

Arquivos locais de Rich Response são enviados temporariamente para o
serviço de upload configurado no injector.

------------------------------------------------------------------------

# 14. Rich Markdown adicional

``` js
await sock.sendRichResponse(jid, {
  markdown: [
    "Primeiro bloco",
    "Segundo bloco"
  ]
});
```

Também:

``` js
markdown: [
  {
    text: "**Markdown customizado**",
    inline_entities: []
  }
]
```

------------------------------------------------------------------------

# 15. Rich Code

Helper:

``` js
await sock.sendCode(jid, `
console.log("Pedrozz Mods");
`, {
  quoted: m
});
```

Pelo `sendMessage`:

``` js
await sock.sendMessage(jid, {
  code: {
    code: `console.log("oi")`,
    language: "javascript",
    title: "Exemplo",
    text: "Código abaixo:"
  }
});
```

Direto no Rich:

``` js
await sock.sendRichResponse(jid, {
  codeLanguage: "javascript",
  code: [
    {
      content: `const x = 10;`,
      type: "DEFAULT"
    },
    {
      content: `console.log(x);`,
      type: "DEFAULT"
    }
  ]
});
```

`code` também pode ser apenas uma string.

------------------------------------------------------------------------

# 16. Rich Table

Helper:

``` js
await sock.sendTable(jid, {
  title: "Usuários",
  headers: ["Nome", "Cargo"],
  rows: [
    ["Pedrozz", "Owner"],
    ["Luminus", "IA"]
  ]
});
```

Ou:

``` js
await sock.sendRichResponse(jid, {
  table: {
    title: "Tabela",
    headers: ["A", "B"],
    rows: [
      ["1", "2"],
      ["3", "4"]
    ]
  }
});
```

------------------------------------------------------------------------

# 17. Rich Math / LaTeX

``` js
await sock.sendMath(jid, {
  expression: "E = mc^2"
});
```

Ou:

``` js
await sock.sendRichResponse(jid, {
  math: [
    {
      expression: "\\frac{a}{b}",
      fontHeight: 28,
      padding: 2
    },
    {
      latex: "x^2 + y^2 = z^2"
    }
  ]
});
```

Aliases da expressão:

``` text
expression
latex
formula
text
```

------------------------------------------------------------------------

# 18. Rich HTML

Helper:

``` js
await sock.sendRichHtml(jid, {
  html: "<h1>Pedrozz Mods</h1>"
});
```

Também aceita arquivo:

``` js
await sock.sendRichHtml(jid, {
  path: "./pagina.html"
});
```

Ou:

``` js
await sock.sendMessage(jid, {
  richHtml: {
    html: "<b>Teste</b>",
    trustedSources: ["nixel.dev"]
  }
});
```

Direto no Rich Response:

``` js
await sock.sendRichResponse(jid, {
  html: {
    payload: "<h1>Hello</h1>",
    trusted_sources: ["nixel.dev"]
  }
});
```

------------------------------------------------------------------------

# 19. Rich Links inline

``` js
await sock.sendRichResponse(jid, {
  text: "Acesse {{SITE}}.{{/SITE}}",
  links: [
    {
      key: "SITE",
      displayName: "Meu site",
      url: "https://example.com",
      isTrusted: true
    }
  ]
});
```

------------------------------------------------------------------------

# 20. Rich Citations

``` js
await sock.sendRichResponse(jid, {
  text: "Resultado {{REF}}.{{/REF}}",
  citations: [
    {
      key: "REF",
      reference_id: 1,
      url: "https://example.com",
      title: "Fonte",
      displayName: "Example",
      sources: []
    }
  ]
});
```

------------------------------------------------------------------------

# 21. Rich Logo

``` js
await sock.sendRichResponse(jid, {
  title: "Sistema",
  logo: {
    path: "./logo.png",
    width: 100,
    height: 100,
    font_height: 24,
    padding: 4
  }
});
```

Também pode ser URL ou `Buffer`.

------------------------------------------------------------------------

# 22. Rich Product individual

`product` gera produto grande/individual:

``` js
await sock.sendRichResponse(jid, {
  product: {
    title: "Produto Premium",
    brand: "Pedrozz Mods",
    price: "R$ 100",
    salePrice: "R$ 79",
    url: "https://example.com/produto",
    image: "./produto.jpg",
    additionalImages: [
      "./produto2.jpg",
      "https://example.com/produto3.jpg"
    ]
  }
});
```

Pode mandar vários:

``` js
product: [
  { title: "A", image: "./a.jpg" },
  { title: "B", image: "./b.jpg" }
]
```

Aliases:

``` text
title / name
brand / marca
salePrice / sale_price / sale
url / product_url / link
additionalImages / additional_images
```

------------------------------------------------------------------------

# 23. Rich Product Scroll

Carrossel horizontal de produtos:

``` js
await sock.sendRichResponse(jid, {
  productScroll: [
    {
      title: "Produto A",
      brand: "Pedrozz",
      price: "R$ 10",
      image: "./a.jpg",
      url: "https://example.com/a"
    },
    {
      title: "Produto B",
      price: "R$ 20",
      image: "./b.jpg"
    }
  ]
});
```

Aliases de carrossel:

``` text
productScroll
productsScroll
products
```

`products` é mantido como alias legado de carrossel.

------------------------------------------------------------------------

# 24. `richProduct` legado/específico

Existe ainda `sendRichProduct()`:

``` js
await sock.sendMessage(jid, {
  richProduct: {
    titulo: "Produto",
    marca: "Pedrozz Mods",
    texto: "Descrição",
    imagem: "./produto.jpg",
    links: [
      {
        nome: "Comprar",
        subtitulo: "Loja",
        link: "https://example.com",
        favicon: "https://example.com/favicon.png"
      }
    ]
  }
});
```

Ou diretamente:

``` js
const { sendRichProduct } = require("./inject");

await sendRichProduct(sock, jid, {
  titulo: "Produto",
  marca: "Marca",
  texto: "Descrição",
  imagem: "./produto.jpg",
  links: []
});
```

------------------------------------------------------------------------

# 25. Rich Reels

``` js
await sock.sendRichResponse(jid, {
  reels: [
    {
      url: "https://instagram.com/reel/...",
      thumbnail: "./thumb.jpg",
      avatar: "./avatar.jpg",
      creator: "@pedrozz",
      title: "Meu reel",
      likes: 100,
      shares: 20,
      views: 5000,
      source: "IG",
      verified: true
    }
  ]
});
```

Aliases suportados incluem:

``` text
thumbnail / thumbnail_url / image
avatar / avatar_url / profilePicture
creator / username / author
title / reels_title
likes / likes_count
shares / shares_count
views / view_count
source / reel_source
verified / is_verified
```

------------------------------------------------------------------------

# 26. Rich Posts

``` js
await sock.sendRichResponse(jid, {
  posts: [
    {
      username: "@pedrozz",
      avatar: "./avatar.jpg",
      thumbnail: "./post.jpg",
      caption: "Meu post",
      url: "https://instagram.com/p/...",
      source: "IG",
      verified: true,
      carousel: false,
      orientation: "portrait",
      type: "IMAGE"
    }
  ]
});
```

Aliases relevantes:

``` text
username / creator / author
avatar / avatar_url / profile_picture_url
thumbnail / thumbnail_url / image
caption / post_caption
url / post_url / link
deeplink / post_deeplink
source / source_app / app
verified / is_verified
carousel / is_carousel
type / post_type
```

------------------------------------------------------------------------

# 27. Rich Cards com CTAs

``` js
await sock.sendRichResponse(jid, {
  prefix: "/",
  cards: [
    {
      title: "Card 1",
      buttons: [
        {
          text: "Executar",
          id: "teste"
        }
      ],
      sections: []
    }
  ]
});
```

Os botões dos Rich Cards aceitam principalmente:

``` js
{
  text: "Menu",
  id: "menu",
  kind: "OTHER",
  toast: "Abrindo menu",
  usePrefix: true
}
```

------------------------------------------------------------------------

# 28. Rich Actions

``` js
await sock.sendRichResponse(jid, {
  prefix: "/",
  actions: {
    title: "O que deseja fazer?",
    cardTitle: "Ações",
    buttons: [
      {
        text: "Menu",
        id: "menu"
      },
      {
        text: "Ping",
        command: "ping"
      }
    ]
  }
});
```

------------------------------------------------------------------------

# 29. Rich Footer

Footer com link:

``` js
await sock.sendRichResponse(jid, {
  text: "Conteúdo",
  footer: {
    text: "Abrir site",
    url: "https://example.com",
    cta_type: "OPEN_URL"
  }
});
```

Footer com imagem:

``` js
await sock.sendRichResponse(jid, {
  footer: {
    image: "./footer.jpg",
    width: 120,
    height: 120,
    font_height: 24,
    padding: -5
  }
});
```

Os dois podem ser usados juntos.

------------------------------------------------------------------------

# 30. Rich Blocks ordenados

`blocks` serve para controlar a ordem e intercalar conteúdo.

``` js
await sock.sendRichResponse(jid, {
  blocks: [
    {
      markdown: "Primeiro texto"
    },
    {
      product: {
        title: "Produto",
        image: "./produto.jpg"
      }
    },
    {
      markdown: "Texto depois do produto"
    },
    {
      productScroll: [
        { title: "A", image: "./a.jpg" },
        { title: "B", image: "./b.jpg" }
      ]
    }
  ]
});
```

Dentro de um block existem suportes para:

``` text
markdown
product
productScroll
productsScroll
products
reels
posts
cards
```

------------------------------------------------------------------------

# 31. Rich Scroll customizado

Para primitives já montadas:

``` js
await sock.sendRichResponse(jid, {
  scrolls: [
    {
      primitives: [
        {
          __typename: "SuaPrimitive",
          // ...
        }
      ]
    }
  ]
});
```

Também aceita diretamente um array de primitives.

------------------------------------------------------------------------

# 32. Rich Components customizados

Escape hatch para estruturas GenAI próprias:

``` js
await sock.sendRichResponse(jid, {
  components: [
    {
      type: "GenAIMarkdownTextUXPrimitive",
      data: {
        text: "Componente customizado"
      }
    }
  ]
});
```

Também aceita:

``` js
{
  primitive: {
    __typename: "...",
    // ...
  },
  viewModelType: "GenAISingleLayoutViewModel"
}
```

ou uma seção completa em `view_model`.

Para controle ainda mais baixo:

``` js
await sock.sendRichResponse(jid, {
  sections: [
    {
      __typename: "GenAIUnifiedResponseSection",
      view_model: {
        // ...
      }
    }
  ]
});
```

------------------------------------------------------------------------

# 33. Imagem com `imageSourceType`

``` js
await sock.sendMessage(jid, {
  image: "./foto.jpg",
  caption: "Imagem",
  imageSourceType: 1
});
```

O injector prepara o `imageMessage` e injeta `imageSourceType`
numericamente.

Também suporta `mentions`, `contextInfo` e `externalAdReply`.

------------------------------------------------------------------------

# 34. Split Payment

**Só funciona em grupos.**

``` js
await sock.sendSplitPayment(jid, {
  total: 100,
  count: 4,
  currency: "BRL",
  pix: {
    key: "+5561999999999",
    keyType: "PHONE",
    merchantName: "Pedrozz Mods"
  }
});
```

O valor é dividido entre participantes LID do grupo.

Também:

``` js
await sock.sendMessage(jid, {
  splitPayment: {
    total: 150,
    participants: [
      "123@lid",
      "456@lid"
    ],
    pixKey: "+5561999999999",
    pixKeyType: "PHONE",
    merchantName: "Pedrozz Mods"
  }
});
```

Aliases do total:

``` text
total
amount
totalAmount
```

Aliases de quantidade:

``` text
count
participantCount
```

O injector limita a divisão a no máximo 8 participantes e exige pelo
menos 2 LIDs.

Campos extras:

``` text
currency
referenceId / reference_id
splitId / split_id
pix / payment
pixKey
pixKeyType
merchantName
```

------------------------------------------------------------------------

# 35. Helpers simples adicionados ao socket

## Texto

``` js
await sock.sendText(jid, {
  text: "Olá",
  quoted: m,
  mentions: ["5511999999999@s.whatsapp.net"]
});
```

## Imagem

``` js
await sock.sendImage(jid, {
  image: "./foto.jpg",
  caption: "Foto",
  quoted: m,
  mentions: []
});
```

Também aceita `file`.

## Vídeo

``` js
await sock.sendVideo(jid, {
  video: "./video.mp4",
  caption: "Vídeo",
  quoted: m
});
```

Também aceita `file`.

## GIF

``` js
await sock.sendGif(jid, {
  gif: "./animacao.mp4",
  caption: "GIF",
  quoted: m
});
```

Também aceita `video` ou `file`. Internamente envia vídeo com
`gifPlayback: true`.

## Áudio

``` js
await sock.sendAudio(jid, {
  audio: "./audio.mp3",
  ptt: false,
  mimetype: "audio/mpeg",
  quoted: m
});
```

Para voz/PTT:

``` js
await sock.sendAudio(jid, {
  audio: "./voz.ogg",
  ptt: true,
  mimetype: "audio/ogg; codecs=opus"
});
```

## Reação

``` js
await sock.sendReact(jid, {
  emoji: "🔥",
  quoted: m
});
```

`quoted` precisa possuir `key`.

------------------------------------------------------------------------

# 36. Mídia aceita

Nos helpers comuns, a mídia pode ser:

## Caminho local

``` js
image: "./media/foto.jpg"
```

## URL

``` js
image: "https://example.com/foto.jpg"
```

## Buffer

``` js
image: fs.readFileSync("./foto.jpg")
```

## Objeto

``` js
image: {
  url: "https://example.com/foto.jpg"
}
```

ou, em partes do Rich:

``` js
image: {
  path: "./foto.jpg"
}
```

------------------------------------------------------------------------

# 37. Mentions

``` js
await sock.sendMessage(jid, {
  text: "Olá @usuario",
  mentions: [
    "5511999999999@s.whatsapp.net"
  ]
});
```

Também funciona nos formatos interativos que passam por
`createContextInfo`.

------------------------------------------------------------------------

# 38. External Ad Reply

``` js
await sock.sendMessage(jid, {
  text: "Teste",
  buttons: [],
  externalAdReply: {
    title: "Pedrozz Mods",
    body: "Sistema",
    mediaType: 1,
    sourceUrl: "https://example.com"
  }
});
```

O objeto é colocado em `contextInfo.externalAdReply`.

------------------------------------------------------------------------

# 39. Options suportadas pelos envios especiais

A maioria dos helpers aceita um quarto/terceiro argumento `options`:

``` js
{
  quoted,
  messageId,
  timestamp,
  ephemeralExpiration,
  participant,
  additionalNodes,
  additionalAttributes,
  statusJidList
}
```

Algumas funções usam campos adicionais, como:

``` text
userJid
useCachedGroupMetadata
bizNode
```

Exemplo:

``` js
await sock.sendMultiButton(
  jid,
  {
    text: "Menu",
    buttons: [
      { type: "reply", text: "Ping", id: "ping" }
    ]
  },
  {
    quoted: m
  }
);
```

------------------------------------------------------------------------

# 40. Exemplo completo

``` js
const { injectButtons } = require("./inject");

injectButtons(sock);

const jid = "5511999999999@s.whatsapp.net";

await sock.sendMultiButton(jid, {
  headerTitle: "Luminus",
  subtitle: "Pedrozz Mods",
  text: "Escolha uma opção:",
  footer: "Sistema online",
  prefix: "/",
  image: "./media/menu.jpg",

  buttons: [
    {
      type: "reply",
      text: "Ping",
      id: "ping"
    },
    {
      type: "copy",
      text: "Copiar código",
      code: "PEDROZZ"
    },
    {
      type: "url",
      text: "Site",
      url: "https://example.com"
    },
    {
      type: "list",
      text: "Mais opções",
      title: "Comandos",
      sections: [
        {
          title: "Sistema",
          rows: [
            {
              title: "Menu",
              description: "Abrir menu",
              id: "menu"
            },
            {
              title: "Status",
              id: "status"
            }
          ]
        }
      ]
    }
  ]
});
```

------------------------------------------------------------------------

# 41. API adicionada pelo `injectButtons`

Depois da injeção, o socket ganha:

``` text
socket.sendNativeInteractive()
socket.sendLimitedTimeOffer()
socket.sendSplitPayment()
socket.sendMultiButton()
socket.sendTButton2()
socket.sendCarousel()
socket.sendRichResponse()
socket.sendRichHtml()
socket.sendCode()
socket.sendTable()
socket.sendMath()
socket.sendPollCommand()
socket.sendForm()
socket.sendForm2()
socket.sendText()
socket.sendImage()
socket.sendVideo()
socket.sendGif()
socket.sendAudio()
socket.sendReact()
```

E o próprio:

``` text
socket.sendMessage()
```

passa a reconhecer automaticamente:

``` text
splitPayment
image + imageSourceType
form
form2
poll
carousel
cards
multiButton
tbutton2
richProduct
code
table
math
richHtml
richResponse
buttons
interactiveButtons
sections
limited_time_offer
limitedTimeOffer
nativeFlowMessage
interactiveMessage
```

------------------------------------------------------------------------

# 42. Exports do módulo

O arquivo exporta:

``` js
const {
  injectButtons,
  sendNativeInteractive,
  sendSplitPayment,
  sendImageWithSourceType,
  sendMultiButton,
  sendTButton2,
  sendCarousel,
  buildCarouselMessage,
  buildInteractiveContent,
  buildOurinSingleSelect,
  normalizeNativeFlowButton,
  normalizeSections,
  resolveQuoted,
  createNativeFlowBizNode,
  normalizeLimitedTimeOffer,
  mergeNativeFlowMessageParams,
  sendRichProduct,
  sendRichResponse,
  sendRichHtml,
  richResponseBizNode,
  richResponseAdditionalNodes,
  normalizePollForSend,
  decryptBotPollVote,
  saveBotPoll,
  emitPollMessage,
  normalizeFormForSend,
  sendGalaxyForm,
  saveBotForm,
  extractGalaxyFormResponse,
  emitFormMessage
} = require("./inject");
```

Na prática, para uso normal do bot, basta importar `injectButtons` e
trabalhar pelos métodos adicionados ao socket.

------------------------------------------------------------------------

# 43. Observações importantes

-   Chame `injectButtons(sock)` apenas depois que o socket existir.
-   O injector evita ser aplicado duas vezes no mesmo socket.
-   `splitPayment` exige grupo e participantes com JID `@lid`.
-   Poll exige pelo menos duas opções.
-   Carousel exige pelo menos um card.
-   Por padrão, cada card do carousel exige pelo menos um botão.
-   `sendReact` exige uma mensagem `quoted` com `key`.
-   Rich media local requer `axios` e `form-data`.
-   URLs temporárias de mídia Rich são cacheadas por um período curto.
-   O injector possui listeners próprios para respostas de Poll e Galaxy
    Form.
-   `products` no Rich Response é alias legado de carrossel; para
    produto individual use `product`.
-   Alguns formatos usados são estruturas internas/experimentais do
    WhatsApp e podem depender da versão/fork do Baileys e do cliente do
    WhatsApp.

------------------------------------------------------------------------

## Resumo rápido

Se você só quer começar:

``` js
const { injectButtons } = require("./inject");
injectButtons(sock);
```

Depois:

``` js
await sock.sendText(jid, { text: "Olá" });

await sock.sendMultiButton(jid, {
  text: "Menu",
  prefix: "/",
  buttons: [
    { type: "reply", text: "Ping", id: "ping" }
  ]
});

await sock.sendCarousel(jid, {
  cards: [
    {
      body: "Card",
      image: "./foto.jpg",
      buttons: [
        { type: "reply", text: "Abrir", id: "abrir" }
      ]
    }
  ]
});

await sock.sendRichResponse(jid, {
  title: "Luminus",
  text: "**Online.**"
});
```

Pronto. O resto é escolher qual formato do injector faz sentido para
cada mensagem.
