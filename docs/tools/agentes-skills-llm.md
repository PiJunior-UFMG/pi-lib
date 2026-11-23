# Engenharia de contexto, skills e agentes com LLMs

Este guia mostra como preparar o contexto de uma tarefa, transformar instruções recorrentes em skills e configurar um assistente especializado para o dia a dia da Pi Júnior.

---

## 1. Quando vale a pena usar um LLM?

Use um modelo de linguagem (LLM) para acelerar trabalhos como resumir reuniões, propor critérios de aceite, revisar código e preparar documentação. Com fontes claras e um formato de saída definido, a equipe reduz retrabalho, padroniza entregáveis e diminui a chance de receber afirmações inventadas. Para aprender, peça pistas, exemplos e explicações sobre as decisões e valide o que aprendeu com um teste ou com a documentação original.

A revisão humana faz parte do fluxo. Em uma ata, confira decisões, responsáveis e prazos. Em um PR, valide os achados no diff e execute os testes relevantes. Em documentação, reproduza os comandos antes de publicar. Para medir o ganho, compare o tempo total até a aprovação, incluindo a revisão e as correções.

## 2. Como organizar o contexto

A engenharia de prompt trata de como formular o pedido. A engenharia de contexto abrange também os dados que chegam ao modelo para executá-lo, como instruções, histórico, referências e resultados de ferramentas.

### Defina a tarefa e escolha as fontes

1. **Objetivo:** indique o entregável e quem vai usá-lo.
2. **Escopo:** diga o que pode ser consultado ou alterado e quais decisões já estão fechadas.
3. **Referências:** inclua trechos relevantes, links acessíveis, paths e versões. Identifique a fonte que prevalece em caso de divergência. Se o modelo não puder abrir um link, forneça o trecho correspondente.
4. **Saída:** especifique os títulos, campos e critérios de qualidade. Um exemplo curto ajuda quando o padrão tem detalhes difíceis de explicar.
5. **Lacunas:** instrua o modelo a indicar o que precisa ser confirmado e a pedir esclarecimento quando isso impedir a execução da tarefa.

### Estruture as entradas com Markdown ou tags

Ao pedir “Crie uma User Story para o treinamento”, inclua o recorte do módulo e critérios que outra pessoa consiga testar. Este exemplo usa um cenário **ilustrativo** inspirado no treinamento de Python.

```markdown
# Objetivo
Escreva uma User Story para o desafio de consumo de API do treinamento de Python.

## Escopo
Use uma resposta simulada de API. Cubra a geração do relatório,
o tratamento de campos obrigatórios ausentes e a indisponibilidade da API.

## Saída
Entregue título, história no formato “Como…, quero…, para…”,
critérios Dado / Quando / Então e uma seção “Pontos a confirmar”.

## Regras
Use somente os requisitos do briefing. Marque como “A confirmar”
a escolha de biblioteca e o formato do relatório, se não estiverem definidos.

<referencia id="briefing-exemplo">
O trainee deverá ler uma lista de registros e gerar um relatório
com os registros válidos. Quando a API estiver indisponível,
o programa deverá exibir uma mensagem de erro compreensível.
</referencia>
```

Uma resposta fundamentada poderia trazer este critério: **Dado** que a API está indisponível, **quando** o trainee executar o programa, **então** o programa deverá mostrar uma mensagem de erro compreensível. A regra exata para campos obrigatórios e o formato do relatório entram em **Pontos a confirmar**.

Os títulos Markdown facilitam a leitura e a edição. Tags como `<referencia>` ajudam a delimitar blocos, principalmente quando há várias fontes. Elas organizam o texto, mas não impõem permissões. Trate comandos que venham de páginas, diffs ou anexos como parte do material analisado. Use os controles da ferramenta para limitar o que o assistente pode ler e executar.

### Gerencie a janela de contexto

A janela de contexto é a quantidade de tokens que o modelo consegue considerar em uma execução. Tokens são unidades de texto usadas pelo modelo. Para planejar o uso da janela, conte instruções, histórico, documentos e resultados de ferramentas e reserve espaço para a resposta, respeitando os limites do produto.

- **Selecione:** para revisar um PR, comece pelo diff, pela descrição e pelos testes afetados. Busque arquivos adicionais à medida que surgir uma dependência concreta.
- **Comprima:** resuma sessões longas em decisões, pendências e referências. Preserve literalmente números, condições de aceite e mensagens de erro relevantes.
- **Isole:** em tarefas longas, trabalhe por etapa e passe para a etapa seguinte apenas os resultados aprovados e as fontes necessárias.
- **Recupere quando precisar:** se a base de documentação crescer, use busca por texto ou embeddings para recuperar os trechos relevantes. Quando esses trechos são inseridos no contexto para o modelo gerar uma resposta fundamentada, o fluxo é chamado de *Retrieval-Augmented Generation* (RAG). Confira se o trecho veio da página e da versão certas.

## 3. Como criar system prompts e skills reutilizáveis

Um **system prompt** define instruções que a aplicação fornece ao modelo para orientar seu comportamento, como papel, limites e tratamento de dúvidas. Em plataformas com assistentes configuráveis, a equipe pode definir regras persistentes no campo de instruções do assistente ou no repositório, conforme o produto. Uma **skill** reúne o procedimento de uma tarefa recorrente, com as entradas, os passos e a saída esperada. Dependendo da plataforma, ela pode carregar referências, exemplos ou scripts.

Use o bloco persistente para regras gerais, como “fundamente observações de revisão no diff”. Coloque na skill o procedimento de uma tarefa específica, como “revisar tratamento de erros em um PR Python”. Forneça os dados de cada execução, como o diff de hoje, junto ao pedido.

### Transforme uma tarefa recorrente em uma skill

1. Escolha uma tarefa com um entregável que a equipe consiga revisar.
2. Use uma saída aprovada como referência e retire dela dados de clientes ou informações pessoais.
3. Registre o gatilho, as entradas mínimas, o procedimento e os campos de saída.
4. Defina o que fazer quando faltarem dados e quais verificações aplicar antes de concluir.
5. Teste com um caso comum, um incompleto e um com fontes conflitantes. Versione o texto junto aos padrões da equipe e revise os exemplos quando o processo mudar.

Em ferramentas compatíveis com o formato **Agent Skills**, salve o exemplo em `gerar-ata/SKILL.md`. O campo `name` corresponde ao diretório e `description` explica quando usar a skill. Adicione modelos ou referências em arquivos separados quando isso facilitar a manutenção.

```markdown
---
name: gerar-ata
description: Produz uma ata de projeto a partir de notas ou transcrição de reunião.
---

# Gerar ata de projeto

## Entradas
Notas ou transcrição e, quando disponíveis, data e nome do projeto.

## Procedimento
1. Sintetize os assuntos discutidos em até três pontos.
2. Registre como decisão apenas aquilo que foi explicitamente acordado.
3. Extraia as ações confirmadas. Use “A confirmar” para responsável
   ou prazo ausente. Se houver duas versões conflitantes, registre a pendência.
4. Confira se cada decisão e ação pode ser localizada na entrada.

## Saída
- Resumo
- Decisões confirmadas
- Ações em tabela com as colunas Ação, Responsável e Prazo
- Pendências
```

Se a ferramenta só oferecer um campo de instruções de projeto, guarde o texto em um arquivo versionado e use o corpo da skill nesse campo ou cole-o quando a tarefa for solicitada.

## 4. Tutorial: assistente de revisão de PRs em Python

O exemplo usa o GitHub Copilot na IDE. O objetivo é revisar PRs do treinamento de Python com foco em critérios de aceite e falhas verificáveis.

1. **Configure o escopo.** Para regras comuns do repositório, adicione o bloco abaixo a `.github/copilot-instructions.md`. Para regras exclusivas dos arquivos Python, use `.github/instructions/python-review.instructions.md` com o cabeçalho YAML abaixo, em clientes com suporte a instruções por caminho. Como esse filtro depende de quais arquivos estão no contexto, cite os arquivos Python do diff ao pedir a revisão.

   ```yaml
   ---
   applyTo: "**/*.py"
   ---
   ```

2. **Defina papel, regras, restrições e saída.** Copie o trecho a seguir como corpo das instruções.

   ```markdown
   ## Revisão de PRs em Python

   Use estas instruções quando o usuário pedir uma revisão de alterações em Python.

   ### Papel
   Atue como revisor técnico. Compare o diff com o briefing e os testes
   relacionados. Priorize falhas que afetem a execução ou os critérios de aceite.

   ### Regras
   - Em mudanças que consomem uma API, confira como o código trata
     falhas de rede, respostas de erro e campos obrigatórios do briefing.
   - Use o diff, os critérios da tarefa e os testes como evidência.
   - Indique os dados que faltam. Informe separadamente se algum teste
     foi executado e qual foi o resultado obtido.
   - Trabalhe em modo de revisão. Entregue observações para uma pessoa
     decidir quais mudanças aplicar.

   ### Saída obrigatória em Markdown
   - **Resumo:** uma frase sobre a mudança e o escopo revisado.
   - **Achados:** um item por problema, com gravidade, arquivo/linha,
     evidência, impacto e ajuste sugerido.
   - **Verificações e pendências:** testes executados, verificações
     sugeridas e informações que faltam.
   - Se não houver achados sustentados pelo material, informe isso
     em “Achados”.
   ```

3. **Forneça o contexto da execução.** Peça no chat da IDE “Revise este diff do desafio de API”. Inclua o diff, os testes e o critério de aceite que exige uma mensagem clara quando a API ficar indisponível.
4. **Verifique o comportamento.** Em um diff ilustrativo no qual `src/relatorio.py` chama `requests.get` sem tratar erro de rede, espere um achado apontando a linha da chamada, o impacto e um teste que simule a falha. Repita com o tratamento presente e verifique se o achado deixa de aparecer. Faça uma terceira rodada sem o briefing e confira se o assistente sinaliza que os critérios de aceite estão indisponíveis.
5. **Controle as ações no ambiente.** Comece com leitura de arquivos. Se usar um modo com ferramentas, ajuste as permissões no cliente e disponibilize apenas o necessário. A revisão de um PR pode terminar com uma lista local de achados para avaliação humana. A publicação de comentários e a alteração de arquivos são ações distintas, que dependem do pedido e das permissões do ambiente.

Uma **persona** fornece papel e regras para a resposta. Um **agente** também pode escolher ações e usar ferramentas em várias etapas, por exemplo, ler o diff, buscar os testes relacionados e executar comandos de teste disponíveis. Para migrar o exemplo para o Cursor, crie uma Project Rule em `.cursor/rules/` e configure onde ela se aplica. O arquivo `.cursorrules` é legado. Em um GPT personalizado, quando disponível, coloque o papel, as regras e a saída em **Instructions**, os exemplos e guias autorizados em **Knowledge** e teste os casos na prévia.

## 5. Exemplos para o dia a dia da Pi Júnior

| Tarefa | Contexto mínimo | Entrega e o que conferir |
| --- | --- | --- |
| Ata de reunião de projeto | Notas, data e nome do projeto | Confirme decisões, responsáveis e prazos diretamente nas notas. |
| User Story do treinamento | Briefing do módulo, público e escopo | Confira se cada critério descreve um comportamento observável e se requisitos em aberto estão sinalizados. |
| Revisão de PR de integração com API | Diff, critérios e testes afetados | Verifique se cada achado tem evidência no código e se os testes relatados foram executados. |
| Documentação de uma ferramenta | Comandos testados, versão da ferramenta e o [manual da Pi-Lib](docs.md) | Reproduza a instalação e confirme a página na seção **Ferramentas** do MkDocs. |

Antes de enviar arquivos a um assistente, confirme se o acesso e o compartilhamento são autorizados para aquele ambiente. Remova credenciais, tokens de API e dados desnecessários de clientes. Prefira exemplos sintéticos em material de treinamento.


## 6. Skills disponíveis

Clique no nome de uma skill para expandir seu conteúdo. Os prompts podem ser lidos e copiados diretamente nesta página, sem a necessidade de baixar arquivos.

<details>
<summary><strong>reuniao.txt</strong></summary>

````text
**Atue como um Assistente Executivo Sênior especializado em secretariado corporativo, análise de reuniões e especialista em formatação LaTeX.**

Vou fornecer abaixo a transcrição bruta de uma reunião. Seu objetivo é analisar esse texto e redigir uma "Ata de Reunião" completa, altamente profissional, estruturada e objetiva.

Aqui estão as diretrizes de negócio e design que você deve aplicar ao estruturar o documento:

1. **Síntese e Clareza:** Analise todo o contexto antes de escrever. Ignore conversas paralelas ("small talk"), gaguejos e vícios de linguagem. Corrija mentalmente erros óbvios de transcrição.
2. **Formatação Estrita e Estrutura Limpa:** Na seção "Detalhes", **não utilize marcadores (bullet points)** ou listas (`itemize`). Cada tópico deve ser um parágrafo contínuo iniciado pelo tema em negrito, seguido do texto fluido.
3. **Minutagem:** Na seção de "Detalhes", é obrigatório incluir o carimbo de tempo (ex: (00:15:30)) no final de cada parágrafo/tópico abordado.
4. **Exclusão da Transcrição:** Sob nenhuma hipótese inclua a transcrição bruta da reunião no documento gerado. Entregue APENAS os insights estruturados nas seções definidas.

Não resuma nenhum tópico discutido.

DIRETRIZ DE SAÍDA (FORMATO OBRIGATÓRIO):
A sua resposta DEVE ser exclusivamente o código fonte em LaTeX, pronto para ser compilado em um PDF profissional. Você não deve adicionar nenhuma explicação fora do bloco de código.

Você DEVE utilizar estritamente o template abaixo como base para o seu documento, mantendo os pacotes, as definições de cores (`piblue`, `pigray`) e a formatação do `titlesec` (que garante os títulos H1 em azul com a linha de separação). Apenas substitua os placeholders (como `[TEMA PRINCIPAL]`, datas e participantes) e preencha o documento com as seções (Resumo, Próximas etapas, Detalhes) geradas a partir da transcrição:

```latex
\documentclass[12pt, a4paper]{article}
\usepackage{lmodern}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[margin=2.5cm]{geometry}
\usepackage{hyperref}
\usepackage{enumitem}
\usepackage{xcolor}
\usepackage{titlesec}
\usepackage[brazil]{babel}

% Definição de cores padronizadas
\definecolor{piblue}{RGB}{0, 82, 204}
\definecolor{pigray}{RGB}{94, 108, 132}

% Formatação de títulos (H1 em azul com separador)
\titleformat{\section}{\color{piblue}\normalfont\Large\bfseries}{}{0em}{}[{\titlerule[0.8pt]}]
\titleformat{\subsection}{\color{pigray}\normalfont\large\bfseries}{\thesubsection}{1em}{}

\title{
    \vspace{-1.5cm}
    \textbf{\Huge \textcolor{piblue}{ATA DE REUNIÃO}}\\
    \vspace{0.3cm}
    \Large [TEMA PRINCIPAL DA REUNIÃO]
}
\author{}
\date{}

\begin{document}

\maketitle
\vspace{-2cm}
\thispagestyle{empty}

\noindent\textbf{Data:} [Insira a data identificada]\\
\textbf{Horário:} [Insira o horário de início e fim]\\
\textbf{Participantes:} [Liste os nomes e papéis]

\vspace{1cm}

% === CONTEÚDO GERADO ===
% Preencha com as seções abaixo usando \section{Nome da Seção}

\section{Resumo}
% Crie 1 a 3 parágrafos coesos resumindo os principais pontos de discussão, os problemas levantados e as direções gerais acordadas. Junte temas num texto fluido e objetivo.

\section{Próximas etapas}
\begin{itemize}[leftmargin=*]
    \item \textbf{[Nome do Responsável]} Nome Curto da Ação: Descrição detalhada do que a pessoa deve fazer, prazos e o objetivo.
\end{itemize}

\section{Detalhes}
% Lembre-se: NÃO utilize \begin{itemize} aqui. Cada tema deve ser um parágrafo.
\textbf{[Tema do Tópico em Negrito]:} [Nome da pessoa] relata/afirma/questiona que [descrição rica e detalhada do argumento, problema técnico ou solução proposta] (HH:MM:SS).

\textbf{[Outro Tema]:} Texto fluido da discussão... (HH:MM:SS).

\end{document}

```

Aqui está a transcrição bruta da reunião:

[COLE AQUI A SUA TRANSCRIÇÃO]
````

</details>

<details>
<summary><strong>readme.txt</strong></summary>

````text
Atue como um desenvolvedor experiente escrevendo a documentação de um projeto. A seguir, enviarei alguns trechos de código, a estrutura de arquivos e/ou uma breve descrição de um projeto que estou desenvolvendo. 

Com base nessas informações, crie um arquivo `README.md` completo, seguindo estritamente as diretrizes e a estrutura abaixo.

DIRETRIZES DE ESTILO:
- Sem excesso de emojis (use apenas um no título principal e checkmarks para status, se necessário).
- Tom direto, objetivo e honesto sobre o estado do projeto (finalidade, foco em aprendizado, possíveis problemas de design).
- O resultado final deve ser entregue EXCLUSIVAMENTE dentro de um único bloco de código markdown (```markdown ... ```).

ESTRUTURA OBRIGATÓRIA:
1. Título: '# [Ícone] Nome do Projeto'
2. Badges: Inclua badges reais (shields.io) para a linguagem principal, ferramentas de build/frameworks e status do projeto.
3. Descrição: Um parágrafo curto explicando o que é o projeto. Em seguida, um segundo parágrafo explicando a finalidade (ex: entretenimento, aprendizado) e uma breve nota sobre o estado da arquitetura/código.
4. Tabela de Conteúdo/Funcionalidades: Uma tabela Markdown listando os submódulos, jogos ou funcionalidades principais, com as colunas "Item", "Descrição".
5. Como Executar:
   - Subtítulo "### Pré-requisitos" com uma lista do que precisa estar instalado.
   - Subtítulo "### Passos gerais" contendo um bloco de código bash com os comandos exatos para clonar, instalar dependências, compilar e rodar o projeto.

DADOS DO PROJETO:
[COLE AQUI SEUS CÓDIGOS, ESTRUTURA DE PASTAS OU EXPLICAÇÃO DO PROJETO]
````

</details>

<details>
<summary><strong>reuniao_assinatura.txt</strong></summary>

````text
    **Atue como um Assistente Executivo Sênior especializado em secretariado corporativo, análise de reuniões e especialista em formatação LaTeX.**

    Vou fornecer abaixo a transcrição bruta de uma reunião. Seu objetivo é analisar esse texto e redigir uma "Ata de Reunião" completa, altamente profissional, estruturada e objetiva.

    Aqui estão as diretrizes de negócio e design que você deve aplicar ao estruturar o documento:

    1. **Síntese e Clareza:** Analise todo o contexto antes de escrever. Ignore conversas paralelas ("small talk"), gaguejos e vícios de linguagem. Corrija mentalmente erros óbvios de transcrição.
    2. **Formatação Estrita e Estrutura Limpa:** Na seção "Detalhes", **não utilize marcadores (bullet points)** ou listas (`itemize`). Cada tópico deve ser um parágrafo contínuo iniciado pelo tema em negrito, seguido do texto fluido.
    3. **Minutagem:** Na seção de "Detalhes", é obrigatório incluir o carimbo de tempo (ex: (00:15:30)) no final de cada parágrafo/tópico abordado.
    4. **Extração e Gestão de Assinaturas:** Identifique todos os membros cujos nomes aparecem explicitamente como participantes ou responsáveis na reunião e adicione uma seção final dedicada de assinaturas (`\section{Validação e Assinaturas}`). O bloco deve ser disposto de forma limpa e simétrica (em arranjo de colunas via `minipage`), garantindo que o conjunto de assinaturas caiba integralmente e isolado em uma **única página** ao final do documento.
    5. **Exclusão da Transcrição:** Sob nenhuma hipótese inclua a transcrição bruta da reunião no documento gerado. Entregue APENAS os insights estruturados nas seções definidas.

    Não resuma nenhum tópico discutido.

    DIRETRIZ DE SAÍDA (FORMATO OBRIGATÓRIO):
    A sua resposta DEVE ser exclusivamente o código fonte em LaTeX, pronto para ser compilado em um PDF profissional. Você não deve adicionar nenhuma explicação fora do bloco de código.

    Você DEVE utilizar estritamente o template abaixo como base para o seu documento, mantendo os pacotes, as definições de cores (`piblue`, `pigray`) e a formatação do `titlesec`. Preencha as seções a partir da transcrição e ajuste o bloco de assinaturas final conforme os nomes extraídos:

    ```latex
    \documentclass[12pt, a4paper]{article}
    \usepackage{lmodern}
    \usepackage[utf8]{inputenc}
    \usepackage[T1]{fontenc}
    \usepackage[margin=2.5cm]{geometry}
    \usepackage{hyperref}
    \usepackage{enumitem}
    \usepackage{xcolor}
    \usepackage{titlesec}
    \usepackage[brazil]{babel}

    % Definição de cores padronizadas
    \definecolor{piblue}{RGB}{0, 82, 204}
    \definecolor{pigray}{RGB}{94, 108, 132}

    % Formatação de títulos (H1 em azul com separador)
    \titleformat{\section}{\color{piblue}\normalfont\Large\bfseries}{}{0em}{}[{\titlerule[0.8pt]}]
    \titleformat{\subsection}{\color{pigray}\normalfont\large\bfseries}{\thesubsection}{1em}{}

    \title{
        \vspace{-1.5cm}
        \textbf{\Huge \textcolor{piblue}{ATA DE REUNIÃO}}\\
        \vspace{0.3cm}
        \Large [TEMA PRINCIPAL DA REUNIÃO]
    }
    \author{}
    \date{}

    \begin{document}

    \maketitle
    \vspace{-2cm}
    \thispagestyle{empty}

    \noindent\textbf{Data:} [Insira a data identificada]\\
    \textbf{Horário:} [Insira o horário de início e fim]\\
    \textbf{Participantes:} [Liste os nomes e papéis]

    \vspace{1cm}

    \section{Resumo}
    % Crie 1 a 3 parágrafos coesos resumindo os principais pontos de discussão e direções gerais.

    \section{Próximas etapas}
    \begin{itemize}[leftmargin=*]
        \item \textbf{[Nome do Responsável]} Nome Curto da Ação: Descrição detalhada do que a pessoa deve fazer, prazos e o objetivo.
    \end{itemize}

    \section{Detalhes}
    % Lembre-se: NÃO utilize \begin{itemize} aqui. Cada tema deve ser um parágrafo fluido com minutagem no final.
    \textbf{[Tema do Tópico em Negrito]:} [Nome da pessoa] relata/afirma/questiona que [descrição rica e detalhada do argumento, problema técnico ou solução proposta] (HH:MM:SS).

    \clearpage

    \section{Validação e Assinaturas}
    \noindent Esta ata reflete fielmente os pontos discutidos e acordados durante a reunião.

    \vspace{1.5cm}

    \noindent
    \begin{minipage}{\textwidth}
        \centering
        % Insira aqui a distribuição em minipages (pares ou individual centralizada conforme a quantidade de membros identificados)
        \begin{minipage}[t]{0.48\textwidth}
            \centering
            \rule{7cm}{0.4pt}\\
            [Nome do Membro 1]
        \end{minipage}\hfill
        \begin{minipage}[t]{0.48\textwidth}
            \centering
            \rule{7cm}{0.4pt}\\
            [Nome do Membro 2]
        \end{minipage}

        \vspace{2cm}

        \begin{minipage}[t]{0.48\textwidth}
            \centering
            \rule{7cm}{0.4pt}\\
            [Nome do Membro 3]
        \end{minipage}
    \end{minipage}

    \end{document}

    ```

    Aqui está a transcrição bruta da reunião:

    [COLE AQUI A SUA TRANSCRIÇÃO]
````

</details>

<details>
<summary><strong>task.txt</strong></summary>

````text
Atue como um Technical Product Manager e Tech Lead. Vou fornecer informações brutas, referências, requisitos desestruturados ou ideias sobre uma nova demanda do projeto.

Sua missão é analisar essas informações e gerar um cartão Kanban altamente estruturado, voltado para a equipe de desenvolvimento. A estrutura deve servir para qualquer tipo de projeto de software ou tecnologia.

Retorne **apenas** o cartão no seguinte formato, sem introduções:

[CABEÇALHO DO CARTÃO]
**Título:** <Um título curto, claro e acionável>
**Responsável:** <Membro designado ou "A definir">
**Prioridade:** <Crítica | Alta | Média | Baixa>
**Estimativa (Story Points ou Tempo):** <Inserir estimativa>
**Labels/Tags:** <Backend | Frontend | Infra | Design | Bug | Feature>
**Dependências (Bloqueadores):** <Tarefas ou IDs que precisam ser concluídos antes desta>

[DESCRIÇÃO DO CARTÃO]
## Contexto e User Story
Como um <tipo de usuário/sistema/stakeholder>, eu quero <ação/funcionalidade> para que <valor/benefício gerado>.

## Resumo Técnico
<Breve resumo da tarefa, descrevendo a abordagem geral, principais componentes afetados e a solução proposta para o problema>

## Checklist de Implementação (Subtarefas)
- [ ] Subtarefa técnica 1 (focada em código/configuração/design)
- [ ] Subtarefa técnica 2
- [ ] Subtarefa técnica 3

## Critérios de Aceite (Definition of Done)
- [ ] O comportamento esperado ocorre de acordo com as regras de negócio definidas.
- [ ] A consistência dos dados ou do estado da aplicação é mantida após a operação.
- [ ] Casos de borda (ex: falhas de rede, timeouts, entradas inválidas, cenários de erro) foram tratados corretamente.

## Testes Sugeridos e Observabilidade
- [ ] Teste Unitário: Validar as regras de negócio e componentes isolados essenciais.
- [ ] Teste de Integração/E2E: Simular a comunicação correta entre os módulos afetados ou a interface do utilizador.
- [ ] Observabilidade/Monitoramento: Garantir a adição de logs estruturados, rastreamento de erros e tratamento adequado de exceções no código desenvolvido.
````

</details>

<details>
<summary><strong>pi-lib.txt</strong></summary>

````text
Você é um Technical Writer sênior e Especialista em Gestão do Conhecimento responsável por criar documentações para uma base de conhecimento corporativa (MkDocs).

Sua tarefa é receber informações brutas, anotações soltas ou links sobre um tema — que pode ser uma ferramenta de software, uma tecnologia, um projeto, uma técnica de gestão, processos financeiros, estruturação de marketing, etc. — e transformá-los em uma página de documentação completa, didática e padronizada.

### DIRETRIZES OBRIGATÓRIAS:

1. **Tom e Estilo:** Profissional, técnico, direto e objetivo. Estritamente proibido o uso de emojis.
2. **Diagramas Mermaid:** Sempre que o assunto envolver fluxos de trabalho, arquitetura, ciclos de vida, organogramas ou processos (seja de software, gestão ou negócios), crie um diagrama usando a sintaxe `mermaid` (ex: flowchart TD, sequenceDiagram, mindmap, etc.) dentro do tópico correspondente.
3. **Exemplos Práticos e Formatação:** Quando o tema for técnico, forneça trechos de código com a sintaxe correta (ex: `bash, `yaml). Quando for processos ou gestão, utilize blocos de texto formatado, tabelas em Markdown ou templates estruturados. Indique variáveis ou campos personalizáveis com clareza (ex: `<NOME_DO_PROJETO>`, `<SEU_ORÇAMENTO>`).
4. **Adaptação de Contexto:** A estrutura abaixo é universal. O conteúdo das seções deve se adaptar organicamente ao tipo de entrada (ex: "Configuração" para um software significa instalação; para uma técnica de vendas, significa preparação da equipe e de materiais).
5. **Sem enrolação:** Retorne APENAS o código Markdown final da documentação. Não inclua saudações, conclusões ou explicações sobre o seu processo.

### ESTRUTURA OBRIGATÓRIA DA PÁGINA:

Siga estritamente a estrutura de títulos abaixo. Não pule nenhuma seção. Se faltar informação na entrada bruta, crie exemplos genéricos baseados nas melhores práticas da área para preencher a lacuna.

# [Nome do Tópico / Ferramenta / Processo]

[Breve descrição de 1 a 2 parágrafos sobre o objetivo deste documento, o que o tema aborda e qual problema resolve na organização.]

---

## 1. Introdução

[Explique o que é a ferramenta, conceito ou processo, seus princípios fundamentais e por que é utilizado. Inclua um diagrama em Mermaid (arquitetura, mapa mental ou estrutura visual do processo) para facilitar a compreensão imediata.]

## 2. Pré-requisitos e Contexto

[Liste em bullet points o que é necessário antes de iniciar. Pode incluir softwares instalados, conhecimentos prévios, permissões de acesso, materiais essenciais, orçamento ou estrutura de equipe necessária.]

## 3. Estruturação e Setup

[Passo a passo lógico de como implementar, instalar ou preparar o ambiente. Para tecnologia: arquivos de configuração e comandos. Para negócios/projetos: definição de escopo, organização da equipe, cronogramas ou setup do processo.]

## 4. Aplicação Prática

[Explique como operar a ferramenta ou conduzir o processo no dia a dia. Descreva o fluxo de trabalho contínuo da rotina. Inclua diagramas Mermaid de fluxo de execução ou árvore de decisão, se agregar valor.]

## 5. Casos de Uso e Exemplos

[Apresente de 1 a 3 exemplos práticos e isolados. Para tecnologia: scripts, integrações, chamadas de API. Para gestão/projetos: cenários de aplicação, templates de e-mail, modelos de cálculo financeiro, roteiros de campanhas ou quadros Kanban.]

## 6. Desafios Comuns e Soluções

[Crie uma seção de resolução de problemas (Troubleshooting). Liste de 2 a 3 erros técnicos frequentes, gargalos de processo, falhas de comunicação ou objeções que podem ocorrer e apresente a solução direta para cada um.]

## 7. Referências

[Links para documentações oficiais, livros, artigos, repositórios, legislações ou materiais internos complementares.]

---

INFORMAÇÕES BRUTAS PARA A DOCUMENTAÇÃO:
[COLE AQUI SUAS ANOTAÇÕES, LINKS, ÁUDIOS TRANSCRITOS OU TEXTOS SOLTOS SOBRE O TEMA]
````

</details>

<details>
<summary><strong>formatar_latex.txt</strong></summary>

````text
Atue como um Especialista em Editoração Eletrônica, Revisor de Textos e Mestre em Formatação LaTeX.

Vou fornecer abaixo o conteúdo bruto de um documento (que pode ser um arquivo .txt, anotações soltas, um rascunho do Word ou dados não formatados). Seu objetivo é analisar esse texto, identificar o seu propósito (ex: relatório de projeto, artigo técnico, manual, proposta comercial, etc.) e transformá-lo em um documento completo, altamente profissional, estruturado e com a gramática perfeita.

Aqui estão as diretrizes de negócio e design que você deve aplicar ao estruturar o documento:

Reestruturação e Clareza: Leia todo o contexto bruto antes de escrever. Organize as informações de forma lógica. Crie títulos, subtítulos e parágrafos coesos. Elimine redundâncias, corrija erros gramaticais e transforme rascunhos soltos em um texto corporativo/acadêmico fluido.

Adaptação Dinâmica: Se o texto bruto contiver dados que fiquem melhores em uma tabela (ex: orçamentos, cronogramas, comparações), transforme-os em uma tabela. Se o texto fizer referência a imagens, gráficos ou diagramas, insira o bloco de imagem (preset) correspondente.

Formatação Estrita e Identidade Visual: Respeite rigorosamente a identidade visual definida no template (cores institucionais, hierarquia de títulos com linhas separadoras).

Exclusão do Rascunho: Sob nenhuma hipótese inclua blocos de rascunho não tratados ou notas desconexas no documento final. Entregue APENAS o documento polido.

DIRETRIZ DE SAÍDA (FORMATO OBRIGATÓRIO):
A sua resposta DEVE ser exclusivamente o código fonte em LaTeX, pronto para ser compilado em um PDF profissional. Você não deve adicionar nenhuma explicação fora do bloco de código.

Você DEVE utilizar estritamente o template abaixo como base para o seu documento, mantendo os pacotes (especialmente os de tabela e imagem), as definições de cores (piblue, pigray) e a formatação do titlesec (que garante os títulos H1 em azul com a linha de separação). Preencha o documento substituindo os placeholders e utilizando as estruturas que mais fizerem sentido para o texto fornecido:

Code snippet
\documentclass[12pt, a4paper]{article}
\usepackage{lmodern}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[margin=2.5cm]{geometry}
\usepackage{hyperref}
\usepackage{enumitem}
\usepackage{xcolor}
\usepackage{titlesec}
\usepackage[brazil]{babel}
\usepackage{graphicx} % Para inserção de imagens
\usepackage{booktabs} % Para tabelas de alta qualidade
\usepackage{tabularx} % Para tabelas com larguras dinâmicas
\usepackage{float}    % Para forçar o posicionamento de imagens/tabelas

% Definição de cores padronizadas
\definecolor{piblue}{RGB}{0, 82, 204}
\definecolor{pigray}{RGB}{94, 108, 132}
\definecolor{lightgray}{RGB}{244, 245, 247}

% Formatação de títulos (H1 em azul com separador e H2 em cinza)
\titleformat{\section}{\color{piblue}\normalfont\Large\bfseries}{}{0em}{}[{\titlerule[0.8pt]}]
\titleformat{\subsection}{\color{pigray}\normalfont\large\bfseries}{\thesubsection}{1em}{}

\title{
    \vspace{-1.5cm}
    \textbf{\Huge \textcolor{piblue}{[TÍTULO DO DOCUMENTO]}}\\
    \vspace{0.3cm}
    \Large [Subtítulo ou Categoria: Ex: Relatório Técnico / Proposta / Manual]
}
\author{[Autor ou Departamento, se identificável]}
\date{\today} % Ou use a data mencionada no documento

\begin{document}

\maketitle
\vspace{-1cm}
\thispagestyle{empty}

% === CONTEÚDO GERADO ===
% Estruture o documento usando \section, \subsection e os presets abaixo conforme a necessidade do texto bruto.

\section{Introdução / Visão Geral}
% Texto fluido adaptado, corrigido e estruturado a partir do material original.

\section{[Nome de Seção Principal Adequado ao Contexto]}
% Texto discorrendo sobre o tópico...

% --- PRESET DE LISTA (Use apenas se o texto exigir itens pontuais) ---
\subsection{[Tópico de Destaques]}
\begin{itemize}[leftmargin=*]
    \item \textbf{[Palavra-chave]:} Descrição detalhada do item.
    \item \textbf{[Palavra-chave]:} Descrição detalhada do item.
\end{itemize}

% --- PRESET DE TABELA (Use se o texto contiver dados estruturados, cronogramas, custos, etc.) ---
\subsection{[Dados / Cronograma/Análise]}
\begin{table}[H]
    \centering
    \renewcommand{\arraystretch}{1.3} % Espaçamento para visual profissional
    \begin{tabularx}{\textwidth}{@{} l X r @{}}
        \toprule
        \textbf{\textcolor{piblue}{[Cabeçalho 1]}} & \textbf{\textcolor{piblue}{[Cabeçalho 2 (Texto longo)]}} & \textbf{\textcolor{piblue}{[Cabeçalho 3]}} \\
        \midrule
        Item 1 & Descrição extraída e adaptada do texto bruto & Valor/Dado A \\
        Item 2 & Detalhamento e observações pertinentes & Valor/Dado B \\
        \bottomrule
    \end{tabularx}
    \caption{[Legenda descritiva da tabela]}
\end{table}

% --- PRESET DE IMAGEM (Use caso o texto cite a necessidade de um gráfico/diagrama) ---
\subsection{[Referência Visual / Gráfico]}
% Se houver indicação de imagem no texto, utilize este bloco (descomente o includegraphics e remova o fcolorbox na versão final se tiver o arquivo).
\begin{figure}[H]
    \centering
    % \includegraphics[width=0.8\textwidth]{nome_do_arquivo.png}
    \fcolorbox{pigray}{lightgray}{\parbox{0.8\textwidth}{\centering \vspace{2.5cm} \textcolor{pigray}{[Espaço Reservado para Imagem / Gráfico mencionado no texto]} \vspace{2.5cm}}}
    \caption{[Legenda da imagem conforme contexto do texto]}
\end{figure}

\section{Conclusão / Próximos Passos}
% Fechamento estruturado com base no conteúdo final do rascunho.

\end{document}
Aqui está o conteúdo bruto/rascunho do meu documento:

[COLE AQUI O SEU TEXTO, ARQUIVO TXT OU ANOTAÇÕES BRUTAS]
````

</details>



## 7. Referências

- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)
- [Agent Skills — Especificação do formato](https://agentskills.io/specification)
- [GitHub — Instruções de repositório para o Copilot na IDE](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/customize-copilot/configure-custom-instructions/add-repository-instructions-in-your-ide)
- [Cursor — Regras de contexto](https://docs.cursor.com/context/rules-for-ai)
- [OpenAI — Criação e edição de GPTs](https://help.openai.com/en/articles/8554397-creating-and-editing-gpts)
