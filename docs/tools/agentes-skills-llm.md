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

## 6. Referências

- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)
- [Agent Skills — Especificação do formato](https://agentskills.io/specification)
- [GitHub — Instruções de repositório para o Copilot na IDE](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/customize-copilot/configure-custom-instructions/add-repository-instructions-in-your-ide)
- [Cursor — Regras de contexto](https://docs.cursor.com/context/rules-for-ai)
- [OpenAI — Criação e edição de GPTs](https://help.openai.com/en/articles/8554397-creating-and-editing-gpts)
