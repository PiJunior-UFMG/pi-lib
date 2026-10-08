# Chrome DevTools

O Chrome DevTools é um conjunto de utilitários para desenvolvedores
integrado diretamente ao navegador Google Chrome. Seu objetivo principal
é fornecer recursos avançados para inspeção estrutural, depuração de
código e análise de performance e tráfego de rede em aplicações web.

------------------------------------------------------------------------

## 1. Introdução

O ecossistema do DevTools é segmentado em diferentes painéis:

-   **Elements:** Vê o esqueleto da página. Permite a manipulação e
    visualização em tempo real do DOM (*Document Object Model*) e das
    regras CSS aplicadas.
-   **Console:** Ambiente de execução de scripts JavaScript da página,
    exibindo logs, *warnings* e erros críticos.
-   **Network (Rede):** Monitoramento de todas as requisições HTTP e
    conexões de rede, como WebSockets e WebRTC.

``` mermaid
mindmap
  root((Chrome DevTools))
    Elements
      Estrutura HTML
      Regras CSS
      Box Model
    Console
      Execucao JS
      Logs de Erro
      Avisos
    Network
      Metodos HTTP
        GET
        POST
      Status Codes
        2xx Sucesso
        4xx Erro Cliente
        5xx Erro Servidor
      Headers
      Filtros e Timings
```

------------------------------------------------------------------------

## 2. Pré-requisitos e Contexto

Antes de utilizar o Chrome DevTools para inspeção e análise de tráfego
web, recomenda-se possuir:

-   Navegador Google Chrome atualizado (versão estável mais recente).
-   Compreensão fundamental do protocolo HTTP (métodos, status codes e
    headers).
-   Conhecimento básico sobre a estrutura de dados JSON (pares de
    "chave" e "valor").
-   Acesso ao ambiente de testes padrão [HTTPBin](https://httpbin.org/)
    ou à aplicação alvo.

------------------------------------------------------------------------

## 3. Estruturação e Setup

### Acesso à Ferramenta

O Chrome DevTools pode ser aberto de diferentes formas:

-   **F12**
-   **Ctrl + Shift + I** no Windows/Linux
-   **Cmd + Option + I** no macOS
-   Botão direito na página → **Inspecionar**

### Configuração do Painel Network

1.  Vá até a aba **Network**.
2.  Marque **Disable cache**: obriga o Chrome a buscar tudo no servidor,
    evitando versões antigas guardadas em cache.
3.  Marque **Preserve log**: mantém o histórico mesmo se a página
    recarregar ou redirecionar.
4.  Selecione o filtro **Fetch/XHR**: exibe apenas o tráfego de dados
    assíncrono, como chamadas de API.

------------------------------------------------------------------------

## 4. Aplicação Prática

A aba **Network** expõe detalhes críticos da comunicação
cliente-servidor:

-   **Método HTTP:** Define a intenção da requisição, como `GET` para
    buscar dados e `POST` para enviar novos dados.
-   **Status Code:** Indica o resultado final:
    -   **Faixa 2xx** (ex.: `200 OK`): Sucesso.
    -   **Faixa 4xx** (ex.: `404 Not Found`): Erro do cliente.
    -   **Faixa 5xx** (ex.: `500 Internal Server Error`): Erro no
        servidor.
-   **Headers:** Metadados transacionais divididos em:
    -   **Request Headers:** Dados enviados pelo navegador, como o
        `User-Agent`.
    -   **Response Headers:** Dados devolvidos pelo servidor.

``` mermaid
sequenceDiagram
    participant U as Usuário
    participant B as Navegador (Chrome)
    participant D as DevTools (Network)
    participant S as Servidor (API)

    U->>B: Interage com a página (ex: preenche e envia um form)
    B->>D: Registra início da requisição ("pending")
    B->>S: Envia Requisição HTTP (Método, Headers, Body)
    S-->>B: Retorna Resposta HTTP (Status, Headers, Resposta final)
    B->>D: Atualiza registro com dados da Resposta
    D-->>U: Exibe Status Code, Payload e Timings
```

------------------------------------------------------------------------

## 5. Casos de Uso e Exemplos

### 5.1. Análise de Requisição GET com Parâmetros (Query Strings)

1.  Abra o DevTools na aba **Network**.

2.  Acesse a URL de testes:

    <https://httpbin.org/get?mes=setembro&dia=segunda>

3.  Clique na requisição `get` na listagem.

4.  Analise: o painel exibirá o método `GET`, status `200 OK` e, na
    resposta, o objeto `args` demonstrando os parâmetros ecoados pelo
    servidor.

#### Exemplo de resposta JSON

``` json
{
  "args": {
    "dia": "segunda",
    "mes": "setembro"
  },
  "headers": {
    "Accept": "text/html...",
    "Host": "httpbin.org",
    "User-Agent": "Mozilla/5.0..."
  },
  "origin": "192.168.0.1",
  "url": "https://httpbin.org/get?mes=setembro&dia=segunda"
}
```

> **Observação:** O valor de `origin` é apenas um exemplo. Na prática, o
> endereço exibido será o endereço de origem identificado pelo servidor.

### 5.2. Teste Rápido via Console

Você pode simular requisições diretamente na aba **Console** usando a
Fetch API:

``` javascript
fetch('https://httpbin.org/post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ usuario: 'admin', acao: 'login' })
})
.then(response => response.json())
.then(data => console.log('Sucesso:', data))
.catch(error => console.error('Erro:', error));
```

------------------------------------------------------------------------

## 6. Desafios Comuns e Soluções

### Problema 1: A requisição foi realizada, mas não aparece na aba Network

**Solução:** Certifique-se de que a gravação está ativa (botão vermelho)
e que o filtro correto está selecionado. Use **All** ou **Fetch/XHR**.
Se necessário, abra o DevTools antes de carregar a página e pressione
`F5`.

### Problema 2: O navegador executa versões antigas ou exibe status `304 Not Modified`

**Solução:** Marque a opção **Disable cache** no painel Network com o
DevTools aberto.

### Problema 3: O JSON de resposta está compactado em uma única linha ilegível

**Solução:** Utilize a sub-aba **Preview** para visualizar o JSON em uma
árvore expansível ou clique no botão de formatação `{}` (*Pretty
print*).

------------------------------------------------------------------------

## 7. Glossário de Termos

-   **Depurar:** Procurar e entender erros em um programa ou página web.
-   **Header:** Informação extra enviada junto com uma requisição ou
    resposta HTTP.
-   **Query string:** Parte da URL que se sucede ao `?`, usada para
    passar parâmetros.

------------------------------------------------------------------------

## 8. Referências Principais

-   **Documentação Oficial Chrome DevTools:**
    <https://developer.chrome.com/docs/devtools/>
-   **HTTPBin --- Teste de Requisições HTTP:** <https://httpbin.org/>
