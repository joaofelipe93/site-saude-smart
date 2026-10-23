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

## 5. O que ainda falta

### Três logos de operadora

A parede de operadoras tem oito posições. Cinco já têm arquivo oficial;
**três estão como cartão tracejado** esperando o arquivo:

| Operadora | Situação |
|---|---|
| Amil, Porto, SulAmérica, Seguros Unimed, Alice | arquivo oficial no repositório |
| **Bradesco Saúde** | falta — o site público da Bradesco só publica o lockup "Bradesco Seguros" |
| **Omint** | falta — o servidor deles bloqueia download direto (WAF) |
| **NotreDame Saúde** | falta — a marca foi absorvida pelo grupo Hapvida e não há lockup próprio público |

Pegue os três no **portal do corretor** de cada operadora, que é a fonte
certa de qualquer jeito: lá vem a versão aprovada para uso por
corretoras, com as regras de área de respiro e tamanho mínimo. Salve em
`imagens/` como `operadora-bradesco-saude.*`, `operadora-omint.*` e
`operadora-notredame.*`, e me avise que eu troco os cartões.

Preferência de formato: SVG > PNG com fundo transparente > PNG comum.

### Dois nomes de depoimento

Os três depoimentos têm texto pronto, mas **dois estão com
`[NOME DO CLIENTE]`** no lugar da assinatura.

⚠️ **Esses textos são rascunhos para você validar com clientes reais,
não para publicar como estão.** Depoimento inventado atribuído a pessoa
inventada é propaganda enganosa e o cliente é quem responde por isso.
Use o texto como roteiro: mostre ao cliente, ajuste com as palavras
dele, peça autorização e só então troque o `[NOME DO CLIENTE]`.

O mesmo vale para o depoimento do **Carlos Santos**: o texto original
falava de seguro de vida e foi reescrito para falar só de plano de
saúde. Confirme com ele antes de publicar — é o nome dele que está ali.
