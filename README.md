
# Citizen Assist AI – Primeiro Emprego

Assistente de terminal, feito em Python, que orienta **jovens que estão procurando o primeiro emprego** sobre documentos, trabalho e direitos básicos, sempre mostrando a fonte oficial, a data de verificação e o aviso de que não é um canal oficial.

Global Solution 2026.2 – Pensamento Computacional e Automação com Python (FIAP – 1CCPK)

## Integrantes

| Nome | RM |
|---|---|
| Gabriel Rodrigues Zappelloni | 572060 |
| Felipe Pereira Restivo | 570712 |
| Maykon de Lima Silva | 574022 |

## Público e problema

**Público:** jovens de 14 a 24 anos buscando o primeiro emprego, estágio ou vaga de Jovem Aprendiz.

**Problema:** quem vai trabalhar pela primeira vez não sabe por onde começar: como tirar o CPF, se ainda precisa "fazer" a carteira de trabalho, como funciona o estágio, onde ver o PIS ou o que fazer se for demitido. A informação existe, mas está espalhada em vários sites do governo, com linguagem técnica e no meio de páginas que parecem oficiais e não são. O assistente reúne essas respostas em linguagem simples, sempre com o link oficial.

## ODS relacionados

- **ODS 8 – Trabalho decente e crescimento econômico:** ajuda jovens a entrar no mercado formal conhecendo seus direitos.
- **ODS 4 – Educação de qualidade:** orienta sobre estágio e aprendizagem profissional.
- **ODS 10 – Redução das desigualdades:** facilita o acesso a serviços públicos para quem tem menos familiaridade com eles.
- **ODS 16 – Paz, justiça e instituições eficazes:** acesso a documentos e informação confiável.

## Como executar

Requisitos: Python 3.8 ou mais novo. Não usa bibliotecas externas.

```bash
git clone <link-do-repositorio>
cd citizen-assist-ai
python main.py
```

No Windows também dá para dar duplo clique em `Iniciar Citizen Assist.bat`.

Menu principal:

```
1. Fazer uma pergunta
2. Navegar por categorias
3. Ver indicadores
4. Exportar atendimentos (CSV)
0. Sair   (pede a nota de facilidade e exporta o CSV)
```

## Estrutura

```
citizen-assist-ai/
├── README.md
├── main.py                  # menu e ligação entre as classes
├── orientacao.py            # classe Orientacao
├── base_conhecimento.py     # classe BaseConhecimento + normalizar()
├── assistente.py            # classe Assistente
└── data/
    ├── base_conhecimento.json   # 13 orientações, 3 categorias
    ├── historico.json           # atendimentos registrados
    ├── avaliacoes.json          # notas de facilidade (1 a 5), criado ao sair
    └── atendimentos.csv         # exportação com csv.DictWriter
```

### Classes (composição)

```
Assistente
 └── base: BaseConhecimento
      └── orientacoes: [Orientacao, Orientacao, ...]
```

| Classe | Atributos | Métodos |
|---|---|---|
| `Orientacao` | id, categoria, titulo, palavras_chave, resposta, documentos, onde, fonte, verificado_em | `exibir()` |
| `BaseConhecimento` | caminho, orientacoes | `carregar()`, `categorias()`, `por_categoria()`, `pontuar()`, `buscar()` |
| `Assistente` | nome, base, historico, avaliacoes | `perguntar()`, `navegar()`, `registrar()`, `indicadores()`, `exportar_csv()`, `avaliar()`, `ler_opcao()`, `ler_sim_nao()`, `tem_dado_pessoal()` |

## Regra de pontuação

A pergunta, as palavras-chave e o título são comparados depois de passar por `normalizar()` (minúsculas, sem acentos e sem pontuação). Em relação ao modelo do enunciado, a equipe incluiu também `-` e `/` na lista de sinais removidos, para que "seguro-desemprego" e "seguro desemprego" sejam tratados igual.

| Regra | Pontos |
|---|---|
| Palavra-chave da orientação que aparece na pergunta | +3 |
| Palavra da pergunta (4 letras ou mais) que aparece no título (cada palavra conta uma vez) | +1 |
| Melhor orientação com 6 pontos ou mais | Confiança **ALTA** |
| Melhor orientação com 3 a 5 pontos | Confiança **MÉDIA** |
| Nenhuma orientação com 3 pontos | "Não encontrei" + sugestão de palavras e categorias |

`buscar()` devolve uma lista de tuplas `(pontos, orientacao)` da maior para a menor pontuação. O assistente mostra a melhor e até duas alternativas com 3 pontos ou mais; se a primeira não ajudar, a pessoa pode abrir uma alternativa.

**Exemplo:** "Como faço a carteira de trabalho digital?"
- palavras-chave `carteira de trabalho` (+3) e `carteira de trabalho digital` (+3)
- palavras do título "Ter a Carteira de Trabalho Digital": `carteira`, `trabalho`, `digital` (+1 cada)
- total: **9 pontos → confiança ALTA**

Os pesos e limites ficam em constantes no topo de `base_conhecimento.py`.

## Regras de atendimento

- **Navegação:** categorias e orientações numeradas, com "0 para voltar" em todas as telas.
- **Entradas inválidas:** letras, opção fora da faixa, resposta vazia e s/n inválido são tratados em laço (`ler_opcao()` e `ler_sim_nao()`), sem encerrar o programa.
- **Dados pessoais:** perguntas com 8 ou mais dígitos ou com `@` são recusadas e **não** são registradas. O assistente nunca pede nome, CPF, telefone ou e-mail.
- **Transparência:** toda resposta mostra fonte, data de verificação e "este assistente NÃO é um canal oficial".
- **Arquivos:** JSON ausente, corrompido ou com registros incompletos não trava o programa (`try/except` em `carregar()` e na leitura do histórico).

## Histórico, indicadores e CSV

Cada atendimento é salvo em `data/historico.json` com: `data`, `origem` (pergunta ou categoria), `pergunta`, `id_orientacao`, `encontrou`, `ajudou`, `tentativas` e `tempo_segundos` (medido com `time.time()`).

- Um atendimento começa na primeira pergunta e termina quando a pessoa diz que a orientação ajudou. Cada pergunta feita nesse intervalo soma uma tentativa.
- Ao sair, o assistente pede uma nota de facilidade de 1 a 5 (salva em `data/avaliacoes.json`).
- Indicadores: resposta encontrada (%), taxa de acerto (%), tempo médio para encontrar, tentativas por atendimento, facilidade de uso e lista de dúvidas sem solução.
- Exportação para `data/atendimentos.csv` com `csv.DictWriter` (separador `;` e codificação `utf-8-sig` para abrir certo no Excel).

## Integrações com outras disciplinas

Nenhuma nesta versão.

## Fontes consultadas

Todas as orientações foram conferidas em 09/11/2026:

- Receita Federal – Inscrição no CPF: https://www.gov.br/pt-br/servicos/inscrever-no-cpf
- Carteira de Identidade Nacional: https://www.gov.br/pt-br/servicos/obter-a-carteira-de-identidade-nacional
- TSE – Autoatendimento Eleitoral: https://www.tse.jus.br/servicos-eleitorais/autoatendimento-eleitoral
- Conta gov.br: https://www.gov.br/governodigital/pt-br/identidade/conta-gov.br
- Carteira de Trabalho Digital: https://www.gov.br/pt-br/servicos/obter-a-carteira-de-trabalho
- Aprendizagem profissional (MTE): https://www.gov.br/trabalho-e-emprego/pt-br/assuntos/aprendizagem-profissional
- Lei do Estágio (Lei 11.788/2008): https://www.planalto.gov.br/ccivil_03/_ato2007-2010/2008/lei/l11788.htm
- SINE: https://www.gov.br/trabalho-e-emprego/pt-br/servicos/trabalhador/sine
- Seguro-desemprego: https://www.gov.br/pt-br/servicos/solicitar-o-seguro-desemprego
- Constituição Federal, art. 7º: https://www.planalto.gov.br/ccivil_03/constituicao/constituicao.htm
- Alistamento militar: https://alistamento.eb.mil.br
- MEI / Portal do Empreendedor: https://www.gov.br/empresas-e-negocios/pt-br/empreendedor

## Resultado do teste com usuários

Teste feito em 09/11/2026 com **3 pessoas de fora da equipe**, somando **14 atendimentos** (4 por pergunta em texto livre e 10 pela navegação por categorias). Nenhum nome ou dado pessoal foi registrado. Os números abaixo são os da opção 3 do menu (Indicadores), calculados a partir de `data/historico.json` e `data/avaliacoes.json`.

| Indicador | Resultado |
|---|---|
| Pessoas testadas | 3 |
| Atendimentos | 14 |
| Resposta encontrada | 85,7% (12 de 14) |
| Taxa de acerto (ajudou) | 85,7% (12 de 14) |
| Tempo médio para encontrar | 17,2 s |
| Tentativas por atendimento | 1,1 |
| Facilidade de uso (1 a 5) | 4,7 (notas 5, 4 e 5) |

**Dúvidas sem solução:**
- "como eu faço um suco de laranja": fora do tema do assistente, o "não encontrei" foi o comportamento correto.
- "como faço pra tirar um cnpj": dúvida real do público que não estava na base.

**Melhoria feita a partir do teste:** foi criada a orientação 13, "Abrir um CNPJ como MEI", na categoria Trabalho, com as palavras-chave `cnpj`, `mei`, `microempreendedor`, `abrir empresa` e `trabalhar por conta`. A mesma pergunta, que antes não era encontrada, agora recebe 5 pontos (confiança MÉDIA) e mostra a orientação certa. No vídeo, a equipe também adicionou ao vivo a palavra-chave `curriculo` na orientação 8 (SINE).

**Observação:** a navegação por categorias foi o caminho mais usado e o mais rápido. Nas perguntas em texto livre, o tempo foi maior porque a pessoa precisa pensar em como escrever a dúvida.

## Limitações

- A busca é por palavras: sinônimos que não estão nas palavras-chave não são entendidos (ex.: "trampo" no lugar de "emprego") e erros de digitação também não.
- Só responde o que está na base. As regras mudam e a base precisa ser revisada manualmente (campo `verificado_em`).
- Respostas são gerais e não substituem o atendimento oficial; casos específicos (ex.: valor e número de parcelas do seguro-desemprego) dependem do órgão responsável.
- A regra de dados pessoais por número de dígitos pode recusar perguntas legítimas com números longos.

## Vídeo técnico

Link (YouTube, não listado):(https://youtu.be/DKIdrAa9U8A?si=xwbXtMW8lfsGnacF)
