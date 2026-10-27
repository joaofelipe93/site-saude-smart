# Site The Smart Corretora — Planos de Saúde

Página única e estática. Sem build, sem Node, sem banco. Para publicar,
copie `index.html` e a pasta `imagens/` para a raiz do site no servidor
Apache, substituindo os arquivos atuais.

```
index.html
imagens/
  logo-thesmart.png
  operadora-amil.svg
  operadora-porto.svg
  operadora-bradesco-seguros.svg
  operadora-sulamerica.png
  operadora-hapvida.png
  operadora-seguros-unimed.png
```

Guarde uma cópia do `index.html` antigo antes de sobrescrever.
O arquivo `teste-widget.html` **não precisa ser publicado** — era só para
inspecionar o formulário do CRM. Pode apagar.

---

## 1. Como o lead chega no CRM

O widget do Funil de Vendas já está embutido na página, **oculto**, no fim
do `index.html`:

```html
<div id="FormFunildeVendas" aria-hidden="true" style="position:absolute;left:-9999px;...">
<script src="https://leads.funildevendas.com.br/Forms/Forms.js"
        id="KeyFormsFunil" data-name="XCSF720261022180737"></script>
</div>
```

O visitante preenche o formulário bonito da página. No envio, o site
copia nome, e-mail e telefone para o formulário do widget e dispara esse
formulário dentro de um iframe invisível. Em seguida abre o WhatsApp.

**Por que assim:** o widget do CRM tem só três campos e um visual próprio.
Embuti-lo visível destruiria o design da seção e perderia a qualificação
de saúde. Desse jeito você fica com os dois.

### Onde foi parar a qualificação de saúde

O formulário do CRM tem três campos obrigatórios — `Name`, `eMail` e
`WhatsApp` — mais três personalizados que você criou: `Idades`, `cep` e
`Hospital que você não quer perder`. Todos são preenchidos.

Além deles, o campo `Tag` recebe um resumo que o CRM usa como **título da
oportunidade**, para o consultor ver a qualificação no card do funil sem
precisar abrir:

```
Site - Planos de Saúde · PME · 3 vidas · CEP 01310-100
```

Se você apagar algum campo personalizado no CRM, o preenchimento daquele
campo é simplesmente ignorado — nada quebra.

> **Campos confirmados em 22/10/2026**, inspecionando o formulário
> publicado. Se você mudar os campos no CRM, me avise para eu refazer o
> mapeamento.

### Trocar de conta do CRM

Mude a chave **nos dois lugares**: no `data-name` do widget e em
`CONFIG.crm.chave`.

### Se o CRM falhar

O lead **nunca se perde por causa do CRM**. Se o widget não carregar,
estiver fora do ar ou vier sem `action`, o código registra um aviso no
console e segue direto para o WhatsApp.

---

## 2. Campanhas: Tag e canal de venda

A página lê da URL, guarda na sessão do visitante e repassa ao CRM:

- `Tag` → entra no começo do título da oportunidade
- `ForceCodeChannel` → define o **canal de venda**
  (código em *Configurações › Integrações › Entrada de dados › Canal de Venda*)
- `utm_source`, `utm_campaign`, `utm_term`

Exemplo de link para anúncio:

```
https://www.thesmart.com.br/?Tag=Meta-PME&ForceCodeChannel=123&utm_source=facebook&utm_campaign=saude-pme
```

Como os valores ficam guardados na sessão, eles sobrevivem à navegação
dentro do site até o envio do formulário — a atribuição não se perde se a
pessoa passear pela página antes de converter.

Para as UTMs aparecerem em toda oportunidade, crie campos personalizados
com **exatamente** estes nomes em *Configurações › Personalizar campos ›
Aba oportunidade*: `utm_source`, `utm_campaign`, `utm_term`, `url`.

---

## 3. Medição

No bloco `CONFIG`, no fim do `index.html`:

```javascript
ga4: "G-XXXXXXXXXX",
metaPixel: "123456789012345"
```

Enquanto vazios, **nenhum script de terceiro é carregado por nossa conta**.
Eventos instrumentados:

| Evento | Quando dispara |
|---|---|
| `whatsapp_click` | qualquer abertura do WhatsApp, com `origem` (`botao_flutuante`, `formulario_estudo`, `busca_rede`) |
| `generate_lead` / `Lead` | envio válido do formulário, com `perfil` |

### ⚠️ Cuidado com Pixel duplicado

O `Forms.js` do Funil de Vendas **instala um Pixel do Facebook por conta
própria** se você tiver configurado um Pixel dentro do CRM. Se você
também preencher `metaPixel` aqui com o mesmo ID, o `PageView` será
contado duas vezes e o seu custo por resultado vai aparecer errado.

Escolha **um dos dois**:

- Pixel configurado no CRM → deixe `metaPixel: ""` aqui.
- Pixel configurado aqui → remova o Pixel de dentro do CRM.

---

## 4. Como testar depois de publicar

1. Abra o site e envie o formulário com os seus próprios dados.
2. Confira se a oportunidade apareceu no funil, com o título qualificado.
3. Abra o console do navegador (F12) e veja se há aviso
   `[The Smart] Formulário do CRM sem action`. Se houver, o widget mudou
   de formato e o envio ao CRM precisa de ajuste — me chame.
4. Teste um link com `?Tag=Teste&utm_source=teste` e veja se chega no CRM.

O e-mail é **obrigatório** no formulário porque o widget do CRM pede esse
campo. Se você confirmar que o CRM aceita lead sem e-mail, dá para voltar
a ser opcional — é uma linha.

---

## 4.1 Número de WhatsApp

O número está em **três** lugares do `index.html` e os três foram
atualizados para **(11) 95262-7883**: o `CONFIG.whatsapp` (que monta o
link do botão flutuante e do envio do formulário) e os dois links
`tel:` — o botão da seção final e o do rodapé. Se a linha de telefone
for diferente do WhatsApp, me avise que eu separo.

---

## 5. O que ainda falta de conteúdo

### Fotos da equipe

As três estão no ar:

| Consultor | Arquivo | Observação |
|---|---|---|
| Cleiton Teixeira | `cleitonteixeira.jpeg` | ampliada no rosto |
| Cesar Nieto | `cesarnieto.jpeg` | ampliada no rosto |
| Rodrigo Francheli | `rodrigofrancheli.jpeg` | entra direto |

**Duas delas precisaram de recorte.** A do Rodrigo já é um retrato de
rosto e entrou como estava. As do Cleiton e do Cesar não são: a do
Cleiton é de meio corpo e a do Cesar é um card de marca, com fundo roxo
e o logotipo do lado. Como esses arquivos são quadrados e o círculo
também, a imagem inteira entraria no avatar de 72px e a pessoa sairia
minúscula. As duas entram ampliadas e deslocadas, enquadrando a cabeça.

Fica funcionando, mas vale saber: **o círculo do Cesar fica roxo e
escuro** ao lado dos outros dois, que têm fundo claro. Se existir um
retrato comum dele, como o do Rodrigo, me mande que eu troco e os três
ficam iguais.

Trocando qualquer uma dessas fotos, me avise: o recorte é medido no
arquivo e precisa ser refeito.

**Os cargos saíram.** A página mostra só os nomes, acompanhando a
decisão que já estava no repositório. Se quiser os títulos de volta,
me mande como cada um deve ser apresentado.

Esses três arquivos substituem `equipe-1.jpg`, `equipe-2.jpg` e
`equipe-3.jpg`, que estavam no repositório: eram **exatamente as mesmas
fotos** (conferido byte a byte), só sem compressão e com nome que não
dizia de quem era. As versões aqui pesam um terço.

### Fotos em uso

| Arquivo | Onde aparece |
|---|---|
| `hero2.jpg` | metade esquerda da primeira tela |
| `maos.jpg` | faixa embaixo dos cartões em "Nossas soluções em saúde" |

`imagens/hero1.jpg` **não é mais usada** e ocupa 842 KB. Pode apagar —
não comprimi essa porque não vai para a tela. Se quiser que eu apague,
é só dizer.

### Compressão: já feita

As imagens já foram comprimidas. A pasta `imagens/` saiu de **4,4 MB
para 433 KB** nas imagens que a página carrega — 90% a menos, sem
diferença visível na tela.

| Arquivo | Antes | Agora |
|---|---|---|
| `hero2.jpg` | 3,1 MB, 5363x3575 | **164 KB**, 1600x1067 |
| `maos.jpg` | 737 KB, 3000x2000 | **86 KB**, 1200x800 |
| `operadora-sulamerica.png` | 268 KB, 2560x590 | **50 KB**, 572x132 |
| `logo-thesmart.png` | 57 KB, 1261x1561 | **33 KB**, 414x512 |
| `rodrigofrancheli.jpeg` | 55 KB | **35 KB**, 555x557 |
| `cleitonteixeira.jpeg` | 43 KB | **28 KB**, 551x559 |
| `cesarnieto.jpeg` | 43 KB | **27 KB**, 548x548 |
| `travus-capital.png` | 41 KB, 2000x1125 | **23 KB**, 364x110 |

Cada arquivo foi reduzido para pouco mais que o dobro do tamanho em que
aparece na tela, que é o que uma tela de celular de alta densidade
precisa. Formato e nome continuam os mesmos, então nada mudou na
publicação.

As três fotos da equipe foram **refeitas maiores** quando os retratos
cresceram na página: saíram de 400px para o tamanho do original,
~555px. Em tela de celular de alta densidade, o enquadramento amplia a
imagem e 400px ficava mole.

A logo da Travus era uma peça inteira de fundo preto, 2000x1125, com o
lockup no meio. Ela foi recortada e **recolorida para fundo claro**: o
dourado da marca ficou, o branco virou o verde-petróleo do site e o
preto virou transparente. Virou `travus-capital.png` (PNG por causa da
transparência). Chegando a versão oficial para fundo claro, é trocar o
arquivo.

`operadora-seguros-unimed.png` ficou como estava: recomprimir deixava o
arquivo **maior** que o original.

**Se um dia você trocar alguma dessas imagens**, mande o arquivo
original e me avise, em vez de subir direto: o arquivo que sai da
câmera ou do banco de imagens costuma ter vários megabytes e é isso que
derruba a nota do site no Google.

Dá para cortar mais uns 40% trocando para o formato **WebP**
(a `hero2.jpg` iria de 164 KB para 92 KB), mas aí os nomes dos arquivos
mudam de extensão. Se quiser, me peça.

### Um produto saiu da seção de planos

A pedido seu, "Nossas soluções em saúde" ficou com **três cartões**:
"PME · 2 a 29 vidas", "Individual e familiar" e "Empresarial 30+".
Saiu só o cartão de **"Adesão por entidade"**.

⚠️ **Atenção, isso mudou.** Quando eu tirei o cartão, deixei o produto
no rodapé e no formulário de propósito, para o lead desse perfil
continuar chegando. Mas no `main` do repositório ele já tinha sido
removido **também do formulário** e do rodapé, e eu segui essa decisão
para não desfazer o que estava lá.

Na prática: o formulário tem **duas** opções de perfil, "Eu ou família"
e "Minha empresa". Ninguém consegue mais se identificar como adesão por
entidade, e o CRM não recebe mais esse perfil. Se isso não era para ter
acontecido, me peça que eu devolvo as três opções — é uma linha.

### Depoimentos: confira a autorização

A página tem três: o do **Carlos Santos** como citação em destaque e,
abaixo de um fio, o do **João Felipe** (PME, 5 vidas) e o do **Ricardo
Melo** (troca de plano, PME 85 vidas).

⚠️ **Antes de publicar, confirme que os três autorizaram.** Depoimento
é fala atribuída a uma pessoa real, com nome e sobrenome visíveis. Se
a pessoa não disse aquilo, ou disse e não autorizou a publicação, quem
responde na reclamação é a corretora. O caminho é simples: mostre o
texto, ajuste com as palavras dela se precisar, peça o ok.

Os textos do João Felipe e do Ricardo Melo foram escritos antes dos
nomes chegarem. Vale a pena ler junto com cada um e ajustar para soar
como a pessoa fala.

Sobre o **Carlos Santos**: o texto dele perdeu a menção a seguro de
vida, a pedido seu. O resto é o original do site antigo.

Trocando um nome, me avise — as iniciais do círculo ao lado mudam
junto.

### Logo da The Smart para fundo claro

O arquivo `imagens/logo-thesmart.png` que temos hoje foi feito para
**fundo escuro**: quase dois terços dele são branco. Sobre a barra do
topo, que é clara, o símbolo praticamente desaparecia — sobravam só os
traços verdes.

Por enquanto ele aparece dentro de uma plaquinha verde-escura, que
devolve o fundo para o qual ele foi desenhado. Funciona e fica bonito,
mas é um remendo.

Se você tiver no material da marca a **versão para fundo claro** (a que
tem o símbolo em verde-escuro ou preto, sem branco), me mande: eu troco
o arquivo e tiro a plaquinha.

### Dois registros ANS

O **registro na ANS** aparece no rodapé, embaixo do nome de cada
operadora, na **tarja** preta e branca que as próprias operadoras usam.
Vale saber: a exigência legal é exibir o número — o retângulo escuro é
convenção de mercado, não formato obrigatório. Seguimos a convenção
porque quem já viu um contrato reconhece o dado de imediato.

Seis números saíram de documento oficial da própria operadora ou de
lista da ANS:

| Operadora | Registro ANS |
|---|---|
| Amil Assistência Médica Internacional | 326305 |
| Bradesco Saúde | 005711 |
| SulAmérica Cia. de Seguro Saúde | 006246 |
| Unimed Seguros Saúde | 000701 |
| Alice | 421928 |
| Notre Dame Intermédica Saúde | 359017 |

⚠️ **Confira os seis no seu credenciamento antes de publicar.** Eu os
levantei em documentos públicos, não no seu contrato. Registro ANS é
identificação regulatória: número errado na página é problema seu, não
meu.

Os outros três vieram de você, do credenciamento: **Omint** 359661,
**Qualicorp** 417173 e **Porto** 000582.

O da Porto bate com o que eu tinha encontrado: o relatório IDSS da
própria Porto traz `00058-2` para a **Porto Seguro - Seguro Saúde**.
Fica o registro de que o grupo tem uma segunda operadora de saúde, a
**Portomed**, com número diferente — se um dia o produto representado
mudar, o número muda junto.

**A lista está completa: os nove têm número.**

### Três logos de operadora

A parede tem **nove** posições, em grade de 3x3. Oito têm arquivo;
**só a Omint** está como cartão tracejado:

| Logo | Situação |
|---|---|
| Amil, Porto, SulAmérica, Seguros Unimed, Alice | arquivo oficial no repositório |
| Qualicorp | arquivo oficial, pego no site deles |
| Hapvida NotreDame Intermédica | logo da **Hapvida**, com o nome do grupo (veja abaixo) |
| Bradesco Seguros | logo do **grupo**, com o nome do grupo (veja abaixo) |
| **Omint** | falta — o servidor deles bloqueia download automatizado |

**Atenção à Qualicorp:** ela não é operadora, é **administradora de
benefícios** — administra planos coletivos por adesão, e quem assume o
risco é a Amil, a SulAmérica e as demais.

No rodapé ela aparece com **duas** linhas: "Administradora de
benefícios" e a tarja `ANS nº 417173`. As duas juntas é que contam a
história certa — administradora também é registrada na ANS, mas como
administradora, não como operadora. Se a linha descritiva sumir, o
número passa a parecer registro de operadora.

Se você quiser separá-la das operadoras de forma visível no muro
também, é só pedir.

**Sobre a NotreDame e a Bradesco:** a seu pedido, os dois cartões
passaram a exibir o logo do **grupo**, com o rótulo acompanhando a
arte. A NotreDame mostra a marca Hapvida e virou "Hapvida NotreDame
Intermédica" no muro, no rodapé e na descrição do site; a Bradesco
Saúde mostra o lockup "Bradesco Seguros", que é como a arte se
apresenta. Assim o logo corresponde ao nome escrito, que é o problema
que existia antes — logo de uma empresa embaixo do nome de outra.

No **rodapé**, a Bradesco continua como "Bradesco Saúde": lá a lista é
de operadoras com registro na ANS, e 005711 é o registro da Bradesco
Saúde S.A. Pôr "Bradesco Seguros" ali colaria o número ao nome errado.

Procurei os três na internet antes disso, e vale registrar o resultado:
o site da Bradesco serve só o SVG "Bradesco Seguros"; o domínio
`notredameintermedica.com.br` não existe mais e o do grupo serve a
marca Hapvida; o da Omint bloqueia qualquer cliente que não seja um
navegador. Os agregadores de logo (seeklogo, cdnlogo, vectorseek,
iconape) têm os arquivos, mas a licença deles é **não comercial** ou
exige autorização por escrito — e um site de corretora é uso
comercial.

Falta só a **Omint**. Pegue no **portal do corretor**, que é a fonte
certa de qualquer jeito: lá vem a versão aprovada para uso por
corretoras, com as regras de área de respiro e tamanho mínimo. Salve em
`imagens/` como `operadora-omint.*` e me avise que eu troco o cartão.
Baixar pelo seu navegador funciona — o bloqueio deles é contra
automação, não contra você.

Se um dia chegar o lockup próprio da **Bradesco Saúde** ou da
**NotreDame**, melhor ainda: aí o logo e o nome da operadora voltam a
ser os mesmos, e é só trocar arquivo e rótulo.

Preferência de formato: SVG > PNG com fundo transparente > PNG comum.

O cartão tracejado é melhor que logo errado: ele registra que a
operadora é representada sem exibir a marca de outra empresa debaixo do
nome errado.

---

## 6. Tema claro, sempre

A página é de **tema claro** do começo ao fim: fundo branco-menta,
texto em verde-escuro e o verde da marca como único acento. **Não há
nenhum bloco escuro.** O verde-petróleo ficou só em detalhes pequenos:
a fita "MAIS PROCURADO", os círculos de iniciais e o texto dos botões
verdes.

O cartão PME continua sendo o destaque, mas por tinta de menta mais
forte, fio verde e a fita, não por peso. O rodapé é branco com um fio
em cima; a faixa do "Pronto para tomar a decisão certa?" é de menta,
para os dois não virarem uma mancha só.

A primeira tela é dividida ao meio: `imagens/hero2.jpg` ocupa a metade
esquerda inteira até a borda da tela e a mensagem fica à direita, sobre
um degradê de menta. No celular o texto vem primeiro e a foto entra
logo abaixo dos botões, em 3:2.

**A página ignora o modo escuro do aparelho.** Quem abrir o site com o
celular ou o Chrome no modo escuro vê exatamente a mesma página clara.
Antes ela acompanhava o aparelho; isso saiu a pedido seu.

Isso vale também para as partes que o navegador pinta sozinho — os
campos do formulário, a lista suspensa, a barra de rolagem e o
preenchimento automático. Sem isso, o visitante no modo escuro veria
caixas pretas no meio de uma página branca justamente no formulário,
que é onde o lead acontece.

São dois pontos no `index.html`, caso um dia queira o modo escuro de
volta: a linha `color-scheme:light` no `:root` e os blocos
`@media (prefers-color-scheme:dark)`, que foram removidos.
