Objetivo: 
automatizar o processo de desenvolvimento de projetos por meio de agentes funcionando em loop, de forma praticamente "ilimitada", utilizando somente ferramentas gratuitas.

Ferramentas:

Metodo:
ao iniciar o atalho para o arquivo executavel do nosso proprio projeto (vou chamar de Factory), executar verificacao se todas as ferramentas estao devidamente disponiveis (python, docker, git, wifi, etc).
ao abrir o "aplicativo" da Factory, a primeira tela deve mostrar as API Keys para o usuario informar (se ja nao tivermos essa informacao antes - ai ja poderia pular essa fase).
com as API Keys configuradas, permissoes concedidas, ferramentas disponiveis, a Factory tem tudo para funcionar.
devemos ter a pagina inicial como "Projetos", que mostre e de a opção do usuário entrar em um dos projetos desenvolvidos e concluidos / em desenvolvimento / criar novo.
Se o usuario criar novo projeto / utilizar projeto ja existente mas não desenvolvido pela Factory, ele deve indicar o repositorio junto de uma forma de conexão.
Compreensão do estado atual do projeto "importado". (se necessário fazemos adaptações/revisões/adicoes/pulamos etapas, a depender do estado do projeto).
Ai sim abrindo uma pagina do próprio desenvolvimento do projeto, com chat / parte visual dos agentes em execucao / fase atual / estimativa de tempo / custo evitado / barra de progresso do projeto completo

Criterios:
1) uso do projeto compreendido por meio de contexto + interrogatorio.
2) requisitos do projeto compreendido.
3) alinhamento de resultados.
4) producao de um prototipo para garantirmos o alinhamento
5) validação humana explicita de "contexto" para iniciar a próxima fase.
6) apresentação de arquitetura + ferramentas indicadas (sempre custo zero).
7) validação de arquitetura.
8) criação dos contratos do projeto e seu fatiamento do escopo.
9) separação das ferramentas e tarefas.
10) validação humana.
11) Start na parte do Loop Engineering de criação.
12) A cada "grande bloco" completo ou 12 horas de execução do projeto realizamos uma grande auditoria para confirmar a integridade e coerência do projeto.
13) com base na auditoria do passo anterior, se encontrados erros trabalhar primeiro nas respectivas correções, até que o controle do projeto seja reestabelecido por completo.
14) fazer uma breve estimativa de tempo para concluir o projeto (com base nos limites de API, taxa de erro, etc). E uma estimativa de custo "evitado" por uso de tokens.
15) quando o software for considerado totalmente pronto, o usuario deve avaliar (e se quiser, realizar solicitações de modificações ou coisas novas - possivelmente voltando alguns passos atras no processo de criação).
16) ate que ele passe na aprovação final do usuario e possa ser dado como "encerrado".

Sempre fazer uso de arquivos .md no repositório do projeto a ser elaborado para deixar os "contextos"/"informações"/"specs"/etc nos diretórios correspondentes visando facilitar a arquitetura do desenvolvimento.
todas as funcoes devem estar corretamente idenficiadas e explicadas (da forma mais sucinta possivel).
a documentação do projeto deve estar organizada e intuitivamente posicionada nos diretorios respectivos.
vamos preferencialmente criar os codigos de backend completo antes do frontend.


Fases:
1) Pré-Alinhamento e Especificação (O "Cérebro")
Nesta etapa, o objetivo é proteger o processo de desenvolvimento contra ambiguidades e "alucinações" dos modelos de linguagem. O sistema atua de forma consultiva, garantindo que o escopo esteja perfeito antes de qualquer implementação.
Interrogatório de Arquitetura: Guiado por uma máquina de estados, um agente com persona de Arquiteto Sênior recusa-se a codificar imediatamente. Ele entrevista você, fazendo perguntas críticas sobre banco de dados, regras de negócio e segurança para eliminar pontas soltas.
Contrato do Projeto (Spec): Assim que você aprova o escopo, as regras são consolidadas em documentos textuais definitivos (como arquivos CONTEXT.md, em cada parte). A partir daqui, os agentes de programação perdem a liberdade criativa e seguem estritamente este contrato.
Fatiamento do Escopo: O Agente Planejador quebra o projeto em pequenos "tickets" independentes, marcados por bloco (segurança, frontend, backend, db, ...). Isso garante que as requisições enviadas à IA permaneçam curtas e dentro da janela de contexto ideal (evitando que o modelo perca o foco).
Interpretação de Ferramentas: Um agente analisa os blocos dos tickets e separa "Ferramentas/Skills" que podem ser utilizadas por cada bloco, visando otimizar o processo.

Fase 2: Desenvolvimento Agêntico (A "Fábrica")
Com as regras do jogo definidas, o sistema de roteamento é ativado para implementar os tickets de forma autônoma, delegando papéis e mantendo o processamento isolado do seu computador pessoal.
Motor de Inteligência na Nuvem: A lógica local envia requisições para o OpenRouter, consumindo modelos de ponta gratuitos (Llama, Mistral). O 9Router opera como um proxy, comprimindo o contexto e gerenciando chaves para contornar limites de taxa de uso.
Os agentes já providos de "contexto" para conseguirem executar suas ações começam a desenvolver mais a fundo as Specs e os Testes (harness) para as tarefas, focando em principalmente dar os meios de identificar quando um código nao esta "suficiente".
Test-Driven Development (TDD): Usando o OpenCode ou Aider, o Agente Programador começa lendo os testes/specs (harness) para o ticket atual. Ele usa o contexto dos testes como guia para iterar/guiar/desenvolver o código real até alcançar o sucesso.
Isolamento de Execução (Sandbox): Todo script gerado pelo Agente Programador é executado em um ambiente de air gap local através de um contêiner Docker descartável. O código nunca acessa o seu sistema operacional hospedeiro, e tem limitações de ações a serem executadas (comandos).
Revisão em Contexto Limpo: Os logs de execução gerados no Docker são recolhidos e enviados juntos do codigo e o requisitos iniciais para o Agente Revisor. Este agente audita o resultado de forma imparcial comparando-o apenas com o contrato definido anteriormente.
