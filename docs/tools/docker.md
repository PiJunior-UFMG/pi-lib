# Docker

O Docker é uma plataforma de virtualização de nível de sistema
operacional projetada para criar, implantar e gerenciar aplicações
dentro de ambientes isolados conhecidos como containers. Seu principal
objetivo é resolver o clássico problema de inconsistência entre
ambientes ("funciona na minha máquina"), garantindo que uma aplicação
rode da mesma forma no ambiente de desenvolvimento, teste e produção.

Diferente das máquinas virtuais (VMs) tradicionais, que exigem a
emulação de um sistema operacional completo para cada aplicação, os
containers Docker compartilham o kernel do sistema operacional
hospedeiro. Isso torna o processo de virtualização muito mais leve,
rápido e eficiente em termos de consumo de recursos de infraestrutura.

------------------------------------------------------------------------

## 1. Introdução

Para compreender o Docker de forma estruturada, é necessário dominar
seus componentes fundamentais. A arquitetura baseia-se em um modelo
cliente-servidor onde o Docker Engine atua como o núcleo, gerenciando a
construção e execução dos containers.

Os pilares da tecnologia incluem:

-   **Imagem:** Um pacote estático e de leitura (*read-only*) que contém
    o código-fonte, bibliotecas, dependências e ferramentas necessárias
    para a aplicação funcionar. Funciona como um "molde" ou fotografia
    do ambiente.
-   **Container:** A instância em execução de uma imagem. É o ambiente
    isolado onde o processo da aplicação efetivamente ocorre. Múltiplos
    containers podem ser gerados a partir de uma única imagem. Uma regra
    importante: um container só continua ativo enquanto existir um
    processo rodando dentro dele; quando o processo termina, o container
    para.
-   **Dockerfile:** Um arquivo de texto puro contendo a "receita"
    (instruções passo a passo) para automatizar a criação de uma imagem.
-   **Registry:** Um repositório centralizado de imagens (ex.: Docker
    Hub). Permite o armazenamento e o compartilhamento de imagens
    prontas com a equipe ou comunidade.
-   **Docker Engine:** Parte do Docker que executa e gerencia os
    containers.
-   **Docker Desktop:** Aplicativo com interface gráfica que já inclui o
    Docker Engine. Útil principalmente no Windows (integrado via WSL 2)
    e no macOS.

``` mermaid
flowchart TD
    A[Cliente Docker / CLI] -->|Comunica via API| B(Docker Engine / Daemon)
    B -->|Baixa imagens| C[(Docker Registry / Hub)]
    B -->|Constrói| D[Imagens]
    D -->|Instancia| E[Containers]
    E -->|Executa| F[Processos da Aplicação]
```

------------------------------------------------------------------------

## 2. Pré-requisitos e Contexto

Antes de iniciar a estruturação de um ambiente Dockerizado, os seguintes
requisitos e conhecimentos devem estar estabelecidos:

### Infraestrutura e Software

-   Docker Engine instalado na máquina hospedeira (recomenda-se a
    utilização nativa via terminal para servidores Linux, ou Docker
    Desktop para Windows via WSL 2 e macOS).
-   Acesso ao terminal do sistema operacional (Bash, PowerShell ou Zsh).

### Acessos e Permissões

-   Privilégios de administrador (`root` ou `sudo`) na máquina para
    execução inicial e configuração de grupos de usuário.
-   Conta ativa em um Registry de imagens, como o Docker Hub, para
    versionamento e armazenamento em nuvem.

### Conhecimento Prévio Necessário

-   Noções básicas de navegação e manipulação de arquivos via linha de
    comando (CLI).
-   Entendimento básico sobre o funcionamento de redes e portas em
    sistemas operacionais.

### Comparativo: Máquina Virtual vs. Container

  | **Critério** | **Máquina Virtual** | **Container** |
|---|---|---|
| **Sistema Operacional** | Tem um sistema operacional completo próprio | Usa o sistema operacional da máquina hospedeira |
| **Peso** | Mais pesada | Mais leve |
| **Isolamento** | Total | Isolado dos outros processos, mas compartilha o kernel do sistema |

## 3. Estruturação e Setup

O processo de implementação inicia-se com a validação do ambiente e a
posterior criação da primeira "receita" de infraestrutura (o
Dockerfile).

### Passo 1: Instalação

Existem duas formas de usar o Docker:

-   Pelo terminal, instalando apenas o Docker Engine. É a opção
    recomendada pela documentação oficial para quem pretende usar
    somente a linha de comando.
-   Pelo Docker Desktop, com interface gráfica.

O guia oficial de instalação está em
<https://docs.docker.com/get-started/>.

### Passo 2: Validação da Instalação

Para garantir que o Docker Engine está operando corretamente e capaz de
se comunicar com o Registry padrão (Docker Hub), execute o comando de
teste oficial:

``` bash
docker run hello-world
```

Se a instalação estiver correta, este comando fará o download de uma
imagem de teste, criará um container, exibirá uma mensagem de sucesso no
terminal e será encerrado.

### Passo 3: Criação do Dockerfile

Na raiz do seu projeto, crie um arquivo com o nome exato de `Dockerfile`
(sem extensão). Este arquivo ditará como a imagem da sua aplicação será
montada.

``` dockerfile
# Define a imagem base a ser utilizada
FROM ubuntu:latest

# Executa comandos de atualização no sistema isolado
RUN apt-get update && apt-get install -y curl

# Define o diretório de trabalho dentro do container
WORKDIR /app

# Comando padrão que manterá o container em execução
CMD ["bash"]
```

### Caminho de estudo recomendado

A equipe sugere seguir esta ordem no site oficial
(<https://docs.docker.com/get-started/>):

1.  Ler a página **What is Docker?**. As primeiras partes bastam. A
    parte sobre a arquitetura do Docker pode ficar para depois.
2.  Estudar a seção **Docker concepts → The basics**. Ela explica
    container, imagem, registry e Docker Compose, e ensina a criar um
    repositório no Docker Hub e enviar uma imagem. Cada página tem um
    vídeo, caso você prefira assistir a ler. Esta parte é essencial.
3.  Depois, você pode aprofundar-se nos outros tópicos disponíveis nesse
    link.

------------------------------------------------------------------------

## 4. Aplicação Prática

No fluxo de trabalho diário de desenvolvimento e operações, a interação
com o Docker ocorre primordialmente através da Interface de Linha de
Comando (CLI). O ciclo de vida de um container depende de processos
ativos; se o processo principal de um container for concluído ou
interrompido, o container será desligado.

``` mermaid
sequenceDiagram
    participant Dev as Desenvolvedor
    participant CLI as Docker CLI
    participant Engine as Docker Engine

    Dev->>CLI: docker build -t .
    CLI->>Engine: Processa Dockerfile
    Engine-->>Dev: Imagem Criada

    Dev->>CLI: docker run
    CLI->>Engine: Inicia Container isolado
    Engine-->>Dev: Container em execução
```

### Comandos Essenciais de Gestão Diária

  -----------------------------------------------------------------------
  Comando                             O que faz
  ----------------------------------- -----------------------------------
  `docker run hello-world`            Cria e executa um container de
                                      teste para confirmar a instalação.

  `docker run -it ubuntu bash`        Cria e executa um container Ubuntu
                                      interativo, abrindo o terminal
                                      (bash).

  `docker ps`                         Lista os containers que estão
                                      atualmente em execução.

  `docker ps -a`                      Lista todos os containers da
                                      máquina, incluindo os parados ou
                                      que falharam.

  `docker stop <ID>`                  Interrompe graciosamente a execução
                                      de um container ativo.

  `docker start <ID>`                 Reinicia a execução de um container
                                      que estava parado.

  `docker exec -it <ID> bash`         Abre um terminal interativo dentro
                                      de um container que já está
                                      rodando, permitindo investigar o
                                      ambiente interno sem pará-lo.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 5. Casos de Uso e Exemplos

Abaixo estão cenários práticos de como o Docker resolve problemas
operacionais de forma isolada e previsível.

### Exemplo 1: Ambiente de testes limpo com Ubuntu interativo

Muitas vezes é necessário testar comandos em um sistema operacional
específico sem afetar a máquina física:

``` bash
# A flag -it (interativo e alocação de TTY) permite que o terminal não feche.
docker run -it ubuntu bash
```

### Exemplo 2: Execução de um banco de dados temporário

Para desenvolver aplicações sem instalações complexas na máquina host:

``` bash
# Executa um container do PostgreSQL em segundo plano (-d) e expõe a porta 5432.
docker run -d --name meu-postgres -p 5432:5432 -e POSTGRES_PASSWORD=senha_segura postgres
```

### Exemplo 3: Envio da imagem para um Registry (Docker Hub)

Após construir a aplicação, o deploy exige enviar a imagem final para um
registro:

``` bash
# 1. Autenticar no Docker Hub
docker login

# 2. Renomear (tagear) a imagem com o padrão do usuário
docker tag minha-app:latest meu-usuario/minha-app:v1.0

# 3. Enviar para a nuvem
docker push meu-usuario/minha-app:v1.0
```

------------------------------------------------------------------------

## 6. Desafios Comuns e Soluções

### Problema 1: O Container inicia e para (morre) imediatamente

**Causa:** Um container Docker só permanece ativo enquanto houver um
processo rodando em primeiro plano (*foreground*) dentro dele. Se o
comando padrão da imagem for apenas um script que termina rapidamente, o
container é encerrado.

**Solução:** Garanta que o `CMD` no seu Dockerfile chame um serviço
contínuo (como um servidor web) ou execute o container com uma flag
interativa (`-it`) caso precise que ele aguarde comandos.

### Problema 2: Erro "Permission denied" ao executar comandos do Docker

**Causa:** O Docker Daemon (Engine) se comunica via soquetes do Unix que
exigem privilégios de administrador (`root`).

**Solução:** Execute utilizando `sudo` ou adicione seu usuário padrão ao
grupo `docker`:

``` bash
sudo usermod -aG docker $USER
```

Reinicie a sessão em seguida.

### Problema 3: Erro "Port is already allocated" ou "Address already in use"

**Causa:** A porta da máquina hospedeira mapeada com a flag `-p` já está
sendo utilizada por outra aplicação ou container.

**Solução:** Pare a aplicação concorrente ou altere o mapeamento (ex.:
de `-p 80:80` para `-p 8080:80`).

------------------------------------------------------------------------

## 7. Glossário de Termos

-   **Container:** Ambiente isolado que executa uma aplicação a partir
    de uma imagem.
-   **Docker Compose:** Ferramenta para gerenciar múltiplos containers
    simultaneamente.
-   **Docker Hub:** Registry público oficial para armazenamento de
    imagens Docker.
-   **Kernel:** Núcleo do sistema operacional compartilhado entre o
    container e a máquina hospedeira.
-   **Registry:** Serviço de distribuição e armazenamento de imagens.
-   **WSL 2:** Subsistema do Windows para Linux utilizado para executar
    o Docker Desktop de forma nativa.

------------------------------------------------------------------------

## 8. Referências Principais

-   **Documentação Oficial - Guia Inicial:**
    <https://docs.docker.com/get-started/>
-   **Manuais e Conceitos Docker:** <https://docs.docker.com/>
-   **Guia prático da DIO:** <https://www.dio.me/>