# Guia Definitivo: Git e GitHub

Bem-vindo ao nosso guia de versionamento! Este documento é parte fundamental da nossa Gestão de Conhecimento Técnico. Aqui você aprenderá desde os conceitos básicos até as melhores práticas de uso do Git e GitHub no dia a dia dos nossos projetos.

---

## 1. Entendendo os Conceitos Básicos

Antes de colocar a mão na massa, é essencial entender a diferença entre as duas ferramentas:

*   **Git:** É o sistema de controle de versão (a ferramenta em si). Ele roda localmente no seu computador e rastreia o histórico de alterações dos arquivos.
*   **GitHub:** É uma plataforma de hospedagem na nuvem para repositórios Git. Facilita a colaboração, revisão de código (Code Review) e gerenciamento de projetos em equipe.

### Termos Importantes
*   **Repositório (Repo):** A pasta do seu projeto, onde ficam os arquivos e o histórico do Git.
*   **Commit:** Um "ponto de salvamento" do projeto. Um pacote com as alterações feitas.
*   **Branch (Ramificação):** Uma linha independente de desenvolvimento. Permite trabalhar em novas funcionalidades sem quebrar o código principal (geralmente a branch `main` ou `master`).
*   **Merge:** A ação de juntar as alterações de uma branch em outra.
*   **Pull Request (PR):** Um pedido no GitHub para que as suas alterações (em uma branch) sejam revisadas e integradas na branch principal.

---

## 2. Configuração Inicial

Se você acabou de instalar o Git, o primeiro passo é configurar sua identidade. Abra o terminal e digite:

```bash
# Define seu nome de usuário
git config --global user.name "Seu Nome Completo"

# Define o seu e-mail (use o mesmo cadastrado no GitHub)
git config --global user.email "seuemail@exemplo.com"

# Verifica se a configuração foi feita corretamente
git config --list
```

---

## 3. Fluxo de Trabalho (O Dia a Dia)

Aqui está o fluxo básico de trabalho em um projeto existente da nossa organização.

### Clonando o Repositório
Para baixar um projeto do GitHub para sua máquina:
```bash
git clone https://github.com/nossa-organizacao/nome-do-repo.git
cd nome-do-repo
```

### Criando uma Branch para sua Tarefa
Sempre crie uma branch nova antes de começar a trabalhar. Nunca programe direto na `main`!
```bash
# Atualiza sua máquina com os dados da nuvem
git pull origin main

# Cria e já entra na nova branch
git checkout -b feature/minha-nova-funcionalidade
```
> **Dica de Nomenclatura:** Use prefixos para facilitar. Ex: `feature/` (nova funcionalidade), `bugfix/` (correção de erro), `docs/` (documentação).

### Salvando as Alterações (Commit)
Depois de editar ou criar seus arquivos:

```bash
# Mostra quais arquivos foram modificados
git status

# Adiciona todos os arquivos modificados para a área de preparação (staging)
git add .

# Ou adiciona um arquivo específico
git add arquivo.txt

# Cria o ponto de salvamento com uma mensagem descritiva
git commit -m "feat: adiciona botão de login na página inicial"
```

### Enviando para o GitHub (Push)
```bash
# Envia a sua branch com os commits para o repositório remoto no GitHub
git push origin feature/minha-nova-funcionalidade
```

Após o `push`, vá até a página do repositório no GitHub e você verá um botão verde sugerindo a criação de um **Pull Request**.

---

## 4. Trabalhando em Equipe (Colaboração)

### O Ciclo do Pull Request (PR)
1. Após fazer o *Push*, abra um **Pull Request** no GitHub.
2. Adicione uma descrição clara do que foi feito.
3. Peça para pelo menos um outro membro da equipe revisar seu código (*Code Review*).
4. Se houver correções, basta fazer novos commits na sua máquina e dar `git push` novamente. O PR é atualizado automaticamente.
5. Após aprovado, clique em **Merge Pull Request** no GitHub.

### Como lidar com Conflitos (Merge Conflict)?
Às vezes, duas pessoas alteram a mesma linha de código. O Git não sabe qual versão manter.
1. O terminal avisará sobre o conflito no momento do `git merge` ou `git pull`.
2. Abra o arquivo no seu editor de código (como o VS Code). Ele mostrará as duas versões.
3. Escolha a versão correta (ou mescle ambas manualmente).
4. Salve o arquivo, rode `git add .` e `git commit -m "Resolve conflitos de merge"`.

---

## 5. Resumo de Comandos Úteis (Cheat Sheet)

| Comando | Para que serve? |
| :--- | :--- |
| `git init` | Inicia um novo repositório Git em uma pasta local. |
| `git log` | Mostra o histórico de commits do projeto. |
| `git branch` | Lista todas as branches locais. A que tem um `*` é a atual. |
| `git switch <nome>` | Muda para uma branch existente (alternativa moderna ao `checkout`). |
| `git restore <arquivo>`| Desfaz as alterações não commitadas em um arquivo específico. |
| `git reset --hard` | **Cuidado:** Apaga todas as alterações não commitadas e volta ao último commit. |

---

## 6. Materiais de Estudo e Aprofundamento

Para dominar o Git, a prática é o melhor caminho. Aqui estão os melhores recursos para estudar:

### Tutoriais e Jogos Interativos
*  **[Learn Git Branching](https://learngitbranching.js.org/?locale=pt_BR):** O melhor recurso interativo (e gamificado) para entender graficamente como branches e merges funcionam. Altamente recomendado para todos os novos membros!
*  **[GitHub Skills](https://skills.github.com/):** Repositórios de treinamento oficiais do próprio GitHub, onde você aprende fazendo.
*  **[Git e Github para Iniciantes (Curso em Vídeo / YouTube)](https://www.youtube.com/results?search_query=git+e+github+curso+em+video):** Excelente para quem prefere vídeo-aulas didáticas e passo a passo.

### Documentação e Convenções
*  **[Git Documentação Oficial (Livro Pro Git)](https://git-scm.com/book/pt-br/v2):** Referência completa, em português.
*   **[Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/):** Convenção para escrever mensagens de commit claras e padronizadas (ex: `feat:`, `fix:`, `chore:`).

---
*Documentação mantida pela Diretoria de Projetos/TI.*
