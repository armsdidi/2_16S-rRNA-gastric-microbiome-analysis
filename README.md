# Pipeline de Análise de Dados de Amplicons do Gene 16S rRNA

Scripts em **Shell e R Markdown** para processamento de dados paired-end de amplicons do gene 16S rRNA e caracterização de comunidades bacterianas gástricas. O fluxo inclui download de dados públicos, controle de qualidade, inferência de variantes de sequência de amplicon (**ASVs**), atribuição taxonômica, reconstrução filogenética, composição, diversidade e abundância diferencial.

**Palavras-chave:** microbioma gástrico; 16S rRNA; ASVs; DADA2; SILVA; câncer gástrico; diversidade microbiana; bioinformática.

## Identificadores e recursos associados

| Recurso | Identificador ou localização |
| --- | --- |
| Código-fonte | [Repositório no GitHub](https://github.com/armsdidi/2_16S-rRNA-gastric-microbiome-analysis) |
| Estudo indicado no script de download | [PRJEB21497 — ENA](https://www.ebi.ac.uk/ena/browser/view/PRJEB21497) |
| Corridas selecionadas no script | 36 acessos: `ERR2014724` a `ERR2014759`, listados individualmente na etapa 1 |
| Referência taxonômica indicada no projeto | SILVA 138.2, com arquivos obtidos no [registro Zenodo 20955974](https://zenodo.org/records/20955974) |
| Licença do código | [MIT](LICENSE) |
| Autor do pipeline | [Diego Pereira](https://github.com/armsdidi) |

Os identificadores do ENA correspondem aos dados de origem. O registro do banco taxonômico não identifica uma versão arquivada deste pipeline. Um DOI específico de versão do software ainda não está documentado.

## Fluxo de trabalho e estrutura

Execute as etapas na ordem numérica, verificando as saídas antes de prosseguir.

| Etapa | Script | Procedimento e principais saídas |
| --- | --- | --- |
| 1 | [1_Download_do_dataset_PRJEB21497.sh](1_Download_do_dataset_PRJEB21497.sh) | Consulta à API do ENA e download dos FASTQ para `16S_FASTQ/RAW_FASTQ` |
| 2 | [2_Controle_de_qualidade.sh](2_Controle_de_qualidade.sh) | FastQC, MultiQC e Cutadapt; relatórios e FASTQ em `TRIMMED_FASTQ` |
| 3 | [3_Inferência_de_ASVs.Rmd](3_Inferência_de_ASVs.Rmd) | DADA2, remoção de quimeras, classificação SILVA e filtragem taxonômica; objeto `phyloseq`, sequências das ASVs e acompanhamento das leituras |
| 4 | [4_Análise_filogenética.Rmd](4_Análise_filogenética.Rmd) | Alinhamento com DECIPHER, árvore inicial por Neighbor Joining e otimização com phangorn; objeto `phyloseq` com árvore |
| 5 | [5_Caracterização_da_comunidade_microbiana.Rmd](5_Caracterização_da_comunidade_microbiana.Rmd) | Composição, prevalência, diversidade, PCoA, PERMANOVA e LEfSe |

**Conexão entre as etapas 2 e 3:** o script de DADA2 está configurado para ler `RAW_FASTQ`, e não os arquivos produzidos pelo Cutadapt. A documentação da etapa 3 registra que os primers não foram detectados neste conjunto de dados e que as leituras brutas foram utilizadas sem corte inicial. Para outro conjunto de dados, avalie os relatórios e adapte os caminhos e padrões dos nomes antes de utilizar leituras com primers removidos.

## Disponibilidade e acesso aos dados

O código e a documentação estão disponíveis publicamente no GitHub. Os arquivos FASTQ não são armazenados neste repositório: a etapa 1 solicita os endereços ao ENA e baixa os arquivos associados aos 36 acessos configurados. A lista representa a seleção do script; não se deve presumir que corresponda a todas as corridas do estudo.

Os arquivos do banco SILVA precisam ser obtidos separadamente. O arquivo **`metadata.xlsx` não está incluído**, portanto o download dos FASTQ, sozinho, não permite reproduzir todas as análises entre grupos.

Para completar a documentação dos dados, disponibilize ou indique:

- A correspondência entre estudo, corrida, amostra biológica e grupo clínico.
- A origem e o procedimento de curadoria dos metadados.
- O dicionário de variáveis, valores ausentes e critérios de inclusão/exclusão.
- As condições de acesso e reutilização dos metadados e dados derivados.
- As somas de verificação dos FASTQ e arquivos de referência.

Caso algum dado exija acesso controlado, documente a restrição e o procedimento de solicitação. A licença do código não substitui os termos de uso dos dados de origem.

## Formatos de entrada e metadados

| Entrada | Formato e estrutura esperados |
| --- | --- |
| Leituras brutas | FASTQ comprimido: `<acesso>_1.fastq.gz` e `<acesso>_2.fastq.gz` |
| Leituras do Cutadapt | `<acesso>_1_trimmed.fastq.gz` e `<acesso>_2_trimmed.fastq.gz`; não são a entrada padrão da etapa 3 |
| Referência para classificação até gênero | `DATABASE/SILVA_138.2/silva_nr99_v138.2_toGenus_trainset.fa.gz` |
| Referência para atribuição de espécies | `DATABASE/SILVA_138.2/silva_v138.2_assignSpecies.fa.gz` |
| Metadados | `metadata.xlsx`, uma linha por amostra e coluna `ID` correspondente ao identificador extraído dos FASTQ |

### Campos essenciais dos metadados

| Campo | Uso e valores esperados |
| --- | --- |
| `ID` | Identificador único; na etapa 3 é convertido em nome de linha e deve coincidir com as amostras da tabela de ASVs |
| `Disease` | Grupo biológico; a etapa 5 utiliza `FD`, `GC` e `GU` |

Documente a definição dos códigos `FD`, `GC` e `GU` no dicionário de dados, com base nos metadados do estudo. Eles não devem ser interpretados como códigos de tipo de tecido sem verificar sua origem. Preserve os identificadores do ENA para permitir rastrear as amostras aos dados originais.

O script verifica igualdade e ordem dos identificadores ao construir o objeto `phyloseq`. A etapa filogenética também verifica a correspondência entre ASVs, sequências e pontas da árvore.

## Saídas e interoperabilidade

| Arquivo ou diretório | Conteúdo |
| --- | --- |
| `16S_FASTQ/QUALITY_CONTROL/` | Relatórios FastQC, MultiQC e Cutadapt |
| `16S_FASTQ/FILT_FASTQ/` | Leituras filtradas pelo DADA2 |
| `16S_DADA2_OUTPUT/dada2_read_tracking.xlsx` | Contagem de leituras retidas em cada etapa |
| `16S_DADA2_OUTPUT/ASV_sequences.xlsx` | Correspondência entre as colunas `ASV` e `Sequence` |
| `16S_DADA2_OUTPUT/16S_phyloseq.RData` | Objeto `ps` com abundâncias, taxonomia e metadados após filtragem |
| `16S_DADA2_OUTPUT/16S_phyloseq_tree.RData` | Objeto `ps` com a árvore filogenética integrada |

O código exporta a correspondência ASV–sequência em **XLSX**, apesar de um trecho explicativo da etapa 3 mencionar CSV. Use o nome realmente gerado pelo código.

Os identificadores `ASV1`, `ASV2`, etc. são locais à análise. Preserve suas sequências completas para comparação com outros conjuntos de dados. Para facilitar reutilização fora do R, recomenda-se distribuir também tabelas TSV/CSV, sequências FASTA e árvore Newick, acompanhadas de um dicionário de dados. Essas exportações adicionais ainda não são fornecidas pelo fluxo atual.

## Requisitos e dependências

### Linha de comando

- Bash e utilitários de linha de comando usados nos scripts.
- **curl** para consulta à API e download; o script atual utiliza curl, não wget.
- **FastQC**, **MultiQC** e **Cutadapt**.

A etapa de controle de qualidade configura 8 threads. A variável `THREADS` da etapa de download não implementa downloads paralelos. Tempo e memória necessários dependem do volume de leituras e do número de ASVs; não há estimativa de requisitos mínimos validada no repositório.

### R

Os scripts carregam os seguintes pacotes:

`dada2`, `phyloseq`, `ggplot2`, `openxlsx`, `tibble`, `Biostrings`, `DECIPHER`, `phangorn`, `dplyr`, `tidyr`, `reshape2`, `readxl`, `vegan`, `ggpubr`, `colorspace`, `gridExtra`, `microbiome`, `microbiomeMarker`, `circlize`, `scales`, `patchwork`, `cowplot`, `ggvenn`, `ComplexHeatmap`, `ade4`, `cluster`, `philentropy`, `factoextra`, `xgboost`, `SHAPforxgboost`, `caret`, `pROC`, `survival`, `survminer`, `glmnet`, `SpiecEasi`, `igraph` e `ggdist`.

A lista reflete chamadas `library()` existentes. O carregamento de um pacote não significa que sua respectiva análise esteja implementada: a etapa 5 não documenta, por exemplo, um modelo de aprendizado de máquina ou análise de sobrevida apenas por carregar esses pacotes.

Use RStudio para executar os chunks ou configure um renderizador compatível com os documentos. Os cabeçalhos contêm opções de formato; revise a configuração antes de renderizar o documento inteiro.

**As versões exatas do R, dos pacotes e das ferramentas de linha de comando não estão registradas.** O projeto não fornece ambiente bloqueado ou contêiner. A versão SILVA e os nomes dos arquivos estão indicados, mas devem ser acompanhados de checksums e informações de obtenção.

## Como executar

### 1. Obter o código

```bash
git clone https://github.com/armsdidi/2_16S-rRNA-gastric-microbiome-analysis.git
cd 2_16S-rRNA-gastric-microbiome-analysis
git rev-parse HEAD
```

Registre o commit utilizado com os resultados. Execute os comandos a partir da raiz do projeto, pois os scripts Shell usam o diretório corrente como base.

### 2. Baixar os dados e executar o controle de qualidade

```bash
bash 1_Download_do_dataset_PRJEB21497.sh
bash 2_Controle_de_qualidade.sh
```

Confirme a presença dos arquivos paired-end esperados e examine os relatórios. O script de download pode continuar quando um acesso não retorna FASTQ; a mensagem final, sozinha, não confirma que todos os dados foram obtidos. Verifique a lista de arquivos e, preferencialmente, compare checksums com o arquivo de origem.

### 3. Preparar referência e metadados

Obtenha os dois arquivos SILVA indicados e coloque-os nos caminhos configurados, ou altere os caminhos na etapa 3. Prepare `metadata.xlsx` com `ID` e `Disease`, mantendo a correspondência com as corridas selecionadas.

### 4. Inferir ASVs e construir a árvore

Execute a etapa 3 em R, com o diretório de trabalho na raiz do projeto. Avalie os perfis de qualidade antes de aplicar os parâmetros de truncamento e confirme qual conjunto de FASTQ será utilizado.

Após gerar `16S_phyloseq.RData` e `ASV_sequences.xlsx`, execute a etapa 4. Ela adiciona a árvore ao objeto e salva `16S_phyloseq_tree.RData`.

### 5. Caracterizar a comunidade

Execute a etapa 5 usando o objeto com árvore. Confirme os grupos de `Disease`, a disponibilidade de amostras e a adequação dos testes. Acompanhe os chunks em ordem e salve explicitamente os resultados de interesse; nem todas as tabelas e figuras possuem comandos de exportação.

## Parâmetros implementados

| Componente | Configuração visível nos scripts |
| --- | --- |
| Cutadapt | Primers ancorados no início; erro máximo `0.1`; comprimento mínimo 50; 8 núcleos |
| Primer forward | `GTGCCAGCMGCCGCGGTAA` |
| Primer reverse | `GGACTACHVGGGTWTCTAAT` |
| DADA2 | `truncLen = c(245, 245)`, `trimLeft = c(0, 0)`, `maxN = 0`, `maxEE = c(2, 5)`, `truncQ = 2`, `rm.phix = TRUE` |
| Filtragem taxonômica | Manter `Kingdom == "Bacteria"`; remover cloroplastos, mitocôndrias e ASVs sem leituras |
| Filogenia | DECIPHER; árvore inicial NJ; otimização GTR com `optInv = TRUE`, `optGamma = TRUE` e rearranjo estocástico |
| Composição | Agregação por gênero e abundância relativa; prevalência mínima de 50% para a seleção de gêneros do núcleo no trecho correspondente |
| Diversidade alfa | Indicadores calculados com `microbiome::alpha(index = "all")`; gráficos de riqueza observada e Shannon; comparações por Wilcoxon |
| Diversidade beta | Bray–Curtis, Jaccard binário, UniFrac ponderado e não ponderado; PCoA e PERMANOVA por `Disease` |
| PERMANOVA | `set.seed(123)`; número de permutações não especificado explicitamente nas chamadas `adonis2` |
| LEfSe | Objeto `ps`, nível `Genus`, normalização CPM, `kw_cutoff = 0.05`, `lda_cutoff = 3` |

Consulte os scripts para os demais argumentos. O README anterior mencionava PERMDISP, mas não há chamada a `betadisper` no script atual de caracterização; não se deve apresentar esse teste como executado sem acrescentar sua implementação.

Os parâmetros de truncamento são específicos deste conjunto de dados e devem preservar sobreposição suficiente para unir os pares. Não os transfira automaticamente para outras regiões do 16S, comprimentos de leitura ou plataformas.

## Reprodutibilidade e proveniência

Para cada execução, registre:

- Commit ou versão do pipeline e quaisquer alterações locais.
- Acessos utilizados, relação amostra–grupo e critérios de inclusão.
- Versões das ferramentas, R e pacotes, com `sessionInfo()`.
- Arquivos e versão da referência, checksums e data de obtenção.
- Parâmetros, sementes aleatórias aplicáveis, relatórios de qualidade e retenção de leituras.
- Transformações das abundâncias e critérios de seleção dos táxons.

A inferência de ASVs, a atribuição taxonômica e as comparações de grupos dependem dessas escolhas. Um perfil de abundância de amplicons não representa diretamente carga bacteriana absoluta, e diferenças entre grupos não estabelecem causalidade.

## Alinhamento aos princípios FAIR

Este README documenta identificadores, acesso, formatos, proveniência e condições de reutilização. **Melhorar o README não estabelece, por si só, conformidade integral com FAIR.** Dados, metadados e versões do software também precisam ser documentados e preservados.

| Princípio | Elementos documentados | Pendências |
| --- | --- | --- |
| **Findable — Encontrável** | Título, palavras-chave, repositório, estudo ENA e lista de corridas | Arquivar uma versão do código com identificador persistente e vincular dados derivados |
| **Accessible — Acessível** | Código público, download via ENA e localização indicada do banco | Disponibilizar metadados ou informar condições de acesso; documentar preservação das saídas |
| **Interoperable — Interoperável** | FASTQ, campos de metadados, identificadores de ASVs e integração por `phyloseq` | Publicar dicionário e esquema de dados; exportar tabelas, sequências e árvore em formatos de intercâmbio |
| **Reusable — Reutilizável** | Licença MIT, ordem de execução, parâmetros, referência SILVA e limitações | Registrar versões/checksums, fornecer ambiente e exemplo de metadados, documentar termos dos dados |

## Como citar

Ao utilizar este pipeline, cite o repositório e informe o commit ou versão utilizada:

> Pereira, Diego. *Pipeline de Análise de Dados de Amplicons do Gene 16S rRNA*. Código-fonte: https://github.com/armsdidi/2_16S-rRNA-gastric-microbiome-analysis. Informar commit/versão e data de acesso.

Cite também o estudo de origem dos dados, os acessos do ENA, DADA2, SILVA e demais ferramentas conforme suas orientações. A autoria do pipeline não implica autoria dos dados públicos reutilizados. Uma versão arquivada e um arquivo `CITATION.cff` permitiriam uma citação mais precisa do software.

## Licença

Código e documentação são distribuídos sob a [Licença MIT](LICENSE), copyright © 2026 Diego Pereira. Preserve o aviso de licença e autoria ao redistribuir o material abrangido.

A licença do repositório não se aplica automaticamente aos dados do ENA, bancos de referência ou software de terceiros; consulte seus próprios termos.

## Autor e suporte

**Diego Pereira**

Bioinformatics Scientist | PhD in Genetics & Molecular Biology | NGS | R | Linux/Bash | Metagenomics & Metatranscriptomics.

Para dúvidas, problemas ou sugestões, utilize as [Issues do repositório](https://github.com/armsdidi/2_16S-rRNA-gastric-microbiome-analysis/issues). Informe script, commit, versões e descrição reproduzível do problema, sem dados pessoais identificáveis.
