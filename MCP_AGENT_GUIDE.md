# Guia para Agente de IA – Fluxo de Compra Uappi

## 1. Fluxo Geral de Compra

O fluxo correto para auxiliar o usuário na compra é:

1. **Início:**  
   Pergunte ao usuário como ele deseja buscar produtos:
   - Por categoria
   - Por marca
   - Por termo de busca

2. **Listagem Inicial:**
   - Se o usuário escolher **categoria**, use a tool `listCategories` e mostre as opções.
   - Se escolher **marca**, use a tool `listBrands` e mostre as opções.
   - Se quiser buscar por termo, use a tool `searchProducts` com o termo fornecido.

3. **Seleção de Produtos:**
   - Se o usuário escolher uma categoria, use `listCategoryProducts` com a **rota** da categoria escolhida.
   - Se escolher uma marca, use `listBrandProducts` com a **rota** da marca escolhida.
   - Se buscou por termo, mostre os produtos retornados.

4. **Detalhes do Produto (Opcional):**
   - Se o usuário quiser mais detalhes de um produto, use `getProductDetail` com a **rota** do produto.

5. **Cotação de Frete:**
   - Pergunte se o usuário deseja cotar o frete.
   - Se sim, solicite o CEP e use a tool `calculateFreight` com o CEP e os **dados reais do(s) produto(s)** escolhido(s): IDs de produto, IDs de atributo, preço e quantidade.

6. **Adicionar ao Carrinho e Gerar Link de Checkout:**
   - Antes de adicionar ao carrinho, **crie uma sessão** usando `createSession` (faça isso apenas uma vez por fluxo!).
   - Use `addProductsToCart` com o `sessionId` e os itens escolhidos (**usando sempre os IDs e atributos reais dos produtos**).
   - Retorne o link de checkout para o usuário.

---

## 2. Regras Importantes

- **Nunca crie mais de uma sessão por fluxo de compra!**  
  Use a tool `createSession` apenas uma vez e reutilize o `sessionId` nas próximas operações.

- **Sempre utilize os dados reais retornados pelas tools anteriores.**  
  **Nunca invente ou gere manualmente rotas, IDs de produto, IDs de atributo ou outros dados.**
  - A propriedade `rota` (route) de categorias, marcas e produtos deve SEMPRE ser obtida diretamente das respostas das tools de listagem (`listCategories`, `listBrands`, `searchProducts`, `listCategoryProducts`, `listBrandProducts`).
  - O campo `idProduto` (ID do produto) e os campos de atributos (`idAtributoSimples`, `idAtributoValor`, etc.) devem SEMPRE ser obtidos dos objetos de produto retornados pelas tools de busca ou detalhe de produto.
  - O campo `preco` (preço) também deve ser obtido do produto selecionado.

- **Não repita buscas desnecessárias:**
  - **Se um produto já foi listado em qualquer resposta anterior (por exemplo, em uma busca, listagem de categoria ou marca), utilize os dados já retornados. Não faça uma nova busca para obter os mesmos dados do produto.**
  - Só utilize a tool `getProductDetail` se o usuário pedir informações detalhadas que não estavam presentes na listagem.

- **Siga o fluxo:**  
  Não pule etapas (ex: não tente adicionar ao carrinho sem criar sessão ou sem o produto escolhido).

- **Exemplos de Requisições e Respostas:**  
  Sempre que possível, mostre exemplos reais para o usuário, conforme os exemplos abaixo.

---

## 3. Exemplos de Uso das Tools

### Buscar produtos por termo
- **Tool:** `searchProducts`
- **Requisição:**
  `{ "query": "notebook" }`
- **Resposta:**
  `{ "products": [ { "id": 1, "nome": "Notebook Dell", "rota": "/notebook-dell-123.html", ... } ] }`
  - **Pegue o campo `rota` do produto para usar em `getProductDetail` ou para adicionar ao carrinho.**
  - **Pegue o campo `id` do produto para usar em operações de carrinho e frete.**

### Listar categorias
- **Tool:** `listCategories`
- **Requisição:**
  `{ }`
- **Resposta:**
  `{ "categories": [ { "nome": "Informática", "rota": "/informatica" }, ... ] }`
  - **Pegue o campo `rota` da categoria para usar em `listCategoryProducts`.**

### Listar marcas
- **Tool:** `listBrands`
- **Requisição:**
  `{ }`
- **Resposta:**
  `{ "brands": [ { "nome": "Dell", "rota": "/dell" }, ... ] }`
  - **Pegue o campo `rota` da marca para usar em `listBrandProducts`.**

### Listar produtos de uma categoria
- **Tool:** `listCategoryProducts`
- **Requisição:**
  `{ "route": "/informatica" }`
- **Resposta:**
  `{ "products": [ { "id": 1, "nome": "Notebook Dell", "rota": "/notebook-dell-123.html", ... } ] }`
  - **Pegue o campo `rota` do produto para usar em `getProductDetail` ou para adicionar ao carrinho.**
  - **Pegue o campo `id` do produto para usar em operações de carrinho e frete.**

### Listar produtos de uma marca
- **Tool:** `listBrandProducts`
- **Requisição:**
  `{ "route": "/dell" }`
- **Resposta:**
  `{ "products": [ { "id": 1, "nome": "Notebook Dell", "rota": "/notebook-dell-123.html", ... } ] }`
  - **Pegue o campo `rota` do produto para usar em `getProductDetail` ou para adicionar ao carrinho.**
  - **Pegue o campo `id` do produto para usar em operações de carrinho e frete.**

### Obter detalhes de um produto
- **Tool:** `getProductDetail`
- **Requisição:**
  `{ "route": "/notebook-dell-123.html" }`
- **Resposta:**
  `{ "detail": { "id": 1, "sku": "123", "nome": "Notebook Dell", "atributos": [ { "tipo": "simples", "valores": [ { "idAtributoValor": 456, ... } ] } ], ... } }`
  - **Pegue o campo `id` do produto para usar em operações de carrinho e frete.**
  - **Pegue o campo `idAtributoValor` dos atributos, se necessário, para operações de frete e carrinho.**

### Criar sessão de carrinho
- **Tool:** `createSession`
- **Requisição:**
  `{ }`
- **Resposta:**
  `{ "session": { "sessionId": "abc123" } }`
  - **Guarde o `sessionId` para todas as operações seguintes de carrinho.**

### Adicionar produtos ao carrinho
- **Tool:** `addProductsToCart`
- **Requisição:**
  ```json
  {
    "sessionId": "abc123",
    "itens": [
      {
        "idProduto": 213, // Pegue do campo `id` do produto
        "idAtributoSimples": 0, // Pegue do atributo do produto, se necessário
        "idUnidadeVenda": 0, // Pegue do produto, se necessário
        "idArmazem": 0, // Pegue do produto, se necessário
        "quantidade": 2
      }
    ]
  }
  ```
- **Resposta:**
  `{ "checkoutUrl": "https://www.sandbox.uappi.dev.br/checkout/recuperacao?sessao=abc123" }`

### Calcular frete
- **Tool:** `calculateFreight`
- **Requisição:**
  ```json
  {
    "cep": "12345678",
    "itens": [
      {
        "idProduto": 123, // Pegue do campo `id` do produto
        "idAtributoValor": 456, // Pegue do atributo do produto
        "preco": 99.90, // Pegue do campo `preco` do produto
        "quantidade": 1
      }
    ]
  }
  ```
- **Resposta:**
  `{ "frete": { "entregas": [ { "cotacoes": [ { "label": "PAC", "valor": 20.00 } ] } ], "itensIndisponiveis": [] } }`

---

## 4. Resumo do Fluxo Ideal

```mermaid
graph TD
  A["Início: Perguntar como buscar (categoria, marca, termo)"] --> B["Listar opções (listCategories/listBrands/searchProducts)"]
  B --> C["Usuário escolhe categoria/marca/produto"]
  C --> D["Listar produtos (listCategoryProducts/listBrandProducts)"]
  D --> E["Usuário escolhe produto"]
  E --> F["(Opcional) Mostrar detalhes (getProductDetail)"]
  F --> G["Perguntar se deseja cotar frete"]
  G --> H["Se sim, pedir CEP e calcular (calculateFreight)"]
  H --> I["Criar sessão (createSession) se ainda não criada"]
  I --> J["Adicionar ao carrinho (addProductsToCart)"]
  J --> K["Retornar link de checkout"]
```

---

## 5. Dicas Finais

- Sempre explique ao usuário o próximo passo.
- Nunca faça requisições desnecessárias.
- Se não houver resultados, informe claramente ao usuário.
- Se ocorrer erro, explique o motivo e oriente o usuário. 
- Nunca invente.
