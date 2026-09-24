---
title: "AI-in-RStudio-airway-package-API-pathway"
output: html_document
date: "2026-09-24"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Table of Contents

00. About this Course
  - Learning goals
  - Abstract
  - Introduction
  - Ethical use of AI for data analysis
  - Methods
  - Troubleshooting Tips
01. Setup
02. Load and qc with gander
03. Deseq2 with gander
04. Plot with gander
05. Annotate with gander

---

# 00 About this Course

## Learning Goals

## Abstract

This course introduces gander and ellmer, two modern R packages designed to embed context-aware AI assistants directly inside the RStudio.

To make these concepts concrete, we use a canonical bulk RNA-seq pipeline as a comprehensive working example.Readers work end-to-end through the classic airway dataset, learning how to inspect and filter raw counts, test for differential expression using DESeq2, and interpret essential visualizations including PCA, MA, and volcano plots. A core strength of this approach is flexibility in execution: learners can choose between a hosted API or a fully local model served via Ollama. This ensures that no paid service is strictly required and that sensitive datasets never need to leave the researcher's local machine.

The patterns learned here - stating an intent, prompting the assistant, inspecting the generated code, executing it, and verifying the output - transfer well across domains, whether analyzing another omics assay or a standard spreadsheet.

**Keywords**: AI-assisted data analysis, gander, ellmer, large language models, reproducible research, RNA-seq, DESeq2, genomic data science education


## Introduction

Welcome to gander-assisted data analysis in RStudio. This course is designed to show you how to utilize LLMs in the context of your RStudio environment.

**Gander and Ellmer tools**
Gander and ellmer perform different jobs, and neither one is the AI model itself.
- `ellmer` is the underlying client package that connects R to various LLM providers. It knows how to speak to a model and handles the mechanics of communication, including authentication and streaming responses token by token It supports major backend providers such as Anthropic, OpenAI, Google, Ollama, and others.
- `gander` is an R package that integrates an AI assistant in your RStudio session and is aware of the objects in your active session. It gathers context from your R environment - such as the objects in your environment, the column names and types in your data frames, the code around your cursor - and hands that context to an ellmer chat.

**Pick your path: Hosted vs. Local**
You have two primary paths for running models in RStudio in this course. Choose the path that best fits your security, budget, and hardware constraings. You can easily switch between them later with only minor changes to your data processing and analysis scripts.

| Feature | Hosted API| Local (Ollama)|
| :--| :--|:--|
| Cost |Per-token| Free|
| Data| code and data are sent to external cloud | Data never leaves your machine (private) |
| Hardware | None or Minimal | Substantial ~19 GB disk; GPU strongly preferred |
| Model quality | Highest available frontier models| Generally smaller, somewhat weaker models |
| Best for | General public datasets and prototyping | Proprietary, sensitive, or restricted data|

**Picking a model/provider:**
Selecting the right model depends heavily on how it is hosted and what task it needs to perform.
- Hosted APIs: If you select the hosted path, you will configure an API key for a kajor provider through 'ellmer', tapping into state-of-the-art cloud models.
- Local models: For local execution via Ollama, we demonstrate using code-optimized models such as qwen3-coder trained specifically on code; General-purpose models of the same parameter size will often produce code that looks plausible at glance, but fails during execution. 

**Bulk RNA-seq as working example:**
We use bulk RNA-seq analysis for several reasons. 
- Canonical pipeline: RNA-seq follows a well-established, multi-step pipeline including quality control, normalization, differential expression testing, and visualization. Readers work through the airway dataset end to end: inspecting and filtering counts, testing for differential expression with DESeq2, interpreting PCA, MA, and volcano plots, and debugging the errors that arise.
- Airway Dataset: We utilize the 'airway' dataset, an RNA-seq transcriptome profiling study of airway smooth muscle cells responding to dexamethasone treatment (PubMed 24926665). 
- Transferable Patterns: The core interaction loop - stating an intent, prompting your assistant, reading and reviewing the returned code, executing it, and verifying the output — is the same loop whether the analysis is RNA-seq, another omics assay, or a spreadsheet.

## Ethical use of AI for data analysis

Integrating generative AI into scientific research introduces important ethical, security, and other considerations. Using AI effectively requires maintaining scientific rigor and transparency.

**Data privacy and the local model option**
- Data Exposure: When using hosted cloud APIs, parts of your workspace, code snippets, or data summaries may traverse external servers. If your dataset is subject to patient privacy constraints (e.g., human-subject genomic data, HIPAA, or institutional data use agreements), sending raw or processed data matrices to external APIs may violate compliance policies.
- The Local Alternative: For restricted data, local models served via Ollama ensure that your data stays strictly on your local machine, bridging the gap between cutting-edge assistance and strict data governance.

**API Key security and best practices**
- Never hardcode API keys directly into shared scripts or repositories like GitHub.
- Store keys securely

**AI tools and human oversight**
- The Assistant paradigm: AI tools are powerful, but they do not replace domain expertise. You, the researcher, are entirely responsible for the final analytical output, statistical validity, and biological interpretations.
- Think for yourself: Never accept a generated code blindly. Always examine why a particular statistical threshold, normalization method, or filtering step was chosen.

**Developing literacy with AI**
- Ask AI to explain code: When gander suggests unfamiliar functions or complex operations, prompt it to break down the logic line by line.

### Questions

1. **What is the researcher's role when integrating generative AI into scientific analysis?**
      A. You can safely accept generated code blindly if it runs without syntax errors.
     B. The AI tool assumes full scientific responsibility for the validity of the results.
     C. You, the researcher, remain entirely responsible for the final analytical output, statistical validity, and biological interpretations.
     D. Domain expertise is no longer required once a model is successfully integrated into RStudio.

Correct Answer: C (AI tools do not replace domain expertise; researchers must maintain oversight and think critically about every analytical step).

2. **If your dataset is subject to strict patient privacy constraints (such as human-subject genomic data, HIPAA, or institutional data use agreements), why might you choose the local model path via Ollama over a hosted API?**
     A. Hosted APIs are illegal to use for any form of scientific research.
     B. Local models ensure that your data stays strictly on your local machine and never traverses external cloud servers.
     C. Local models provide higher-quality frontier model outputs than any hosted cloud API.
     C. Local models automatically encrypt API keys so you never have to worry about security.

Correct Answer: B (Local models ensure data privacy by keeping sensitive or restricted data entirely on your own machine).

3. **What is the primary difference between ellmer and gander?**
    A. gander handles the underlying cloud authentication, while ellmer is the RStudio add-in.
    B. ellmer connects R to various LLM providers, while gander integrates an AI assistant into your RStudio session and gathers context from your active environment.
    C. Both packages are local AI models that require a minimum of 19 GB of disk space.
    D. gander is used exclusively for Python integration, while ellmer is designed for R.

Correct Answer: B (ellmer manages communication and client connection to LLMs, whereas gander wraps that into an RStudio assistant aware of your active session environment).

---

## Methods 

### Environment Requirements

- The book requires R (>= 4.3) and RStudio. 
- Packages fall into several families: data wrangling (readr, tidyr, dplyr, ggplot2), differential expression (DESeq2, installed via BiocManager),and AI integration (gander, ellmer). The data itself is another package called 'airway', installed also via BiocManager.
- The model is supplied through a provider key set in the session environment, and ellmer is pointed at a cloud model.  

### Workflow

Five stages, one per chapter: environment setup; data import and quality control; differential expression with DESeq2; visualization and interpretation; and debugging. Each stage is taught as a loop rather than a recipe - state an intent, prompt the model, read the returned code, run it, inspect the output, and if the result is wrong, pass the error back verbatim before accepting a fix.The workflow is not particular to RNA-seq; it is the generic shape of an analysis, and the book is written so that this is visible. 

### Dataset

RNA-seq supplies the instance. The airway dataset (dexamethasone-treated versus untreated human airway smooth muscle cells) is used throughout because it is small, freely available, and has well-characterized biology, which lets readers check the model's biological interpretation against what is independently known about the glucocorticoid response — and that check is the point. Where an analysis has no independently known answer, the reader needs to construct one; the book models how.

#### Questions

1. What is the primary rationale for using the airway dataset throughout the book's RNA-seq analysis examples?

A) It is a massive dataset that requires high-end enterprise server clusters to process.
B) It is small, freely available, and has well-characterized biology, which allows readers to check the model's biological interpretations against independently known facts.
C) It requires zero normalization or data filtering steps compared to other omics data.
D) It is the only dataset that can integrate natively with the gander package add-in.

---> Correct Answer: B (The airway dataset's well-characterized biology provides a benchmark to check the model's biological interpretations).

---

## Troubleshooting Tips

### Prompt rule of thumb

- keep prompts short and concrete. Rather than *"help me do RNA-seq"* (which is too broad), ask *"Summarize key counts file statistics in a table"* or *"Filter genes with at least 1 count in all samples"*.

### A Critical tip for Data Analysis with Gander: Minimize Context Noise

Minimize context noise by opening a new .R script containing only what the task needs:
- E.g. If you want to ask about airway dataset, you rnew .R script should contain only word `airway` corresponding to a loaded dataset in your RStudio environment.*Keeping the script lean prevents background code clutter from interfering with Gander's responses. A long script sends the assistant a long pile of unrelated code, and the extra context does not help it answer.*

###  Troubleshooting errors

- **Ask again, verbatum**. A second answer may be different.
- **Ask the same question sing a different prompt**. Ask again, but more specifically, e.g. name the object, the column, and the output format you want.
- **Highlight the error and the code together**.  The pairing of the error with error-producing code provides important context.
- **Remove context**. Use `chat$chat("...")` instead of gander add-in. The difference between using the gander add-in and standard chat (`chat$chat`) is that gander reads your surrounding code and activate environment and sends it along with your prompt. Sometimes, a verbally expanded question with no specific context can help. 
- **Add more context** In opposite approach, expand gander context rather than limit it. E.g., provide additonal objects that may be helpful.

#### Questions

1. Why you may want to keep your script lean and open a new .R script containing only the necessary code when asking Gander for help?

A) To automatically install missing Bioconductor package dependencies
B) To prevent background code clutter from sending unrelated context and confusing the assistant
C) To permanently save your workspace environment variables to disk
D) To ensure your API key remains hidden from the R console history

---> Correct Answer: B (A long script sends a pile of unrelated code, adding unnecessary context noise).

---

## How to use gander shortcut:

> [!IMPORTANT]
> 1. **Highlight** an object: e.g. 'counts' \
> 2. **Evoke gander** with your pre-set shortcut *Shift+Cmd+g* \
> 3. **Enter a prompt** - a short, concrete instruction of what you want to do in plain language \
> 4. **Review the output** before running the generated code

## gander_peek

---


# 01. Setup

Most people meet a language model in a browser tab. They describe their data in prose, paste the code they get back into their editor, run it, watch it fail, and describe the failure in prose again. The model never sees the data. It never sees the error message, the column names, the shape of the object, or the three lines above the cursor that establish what the analyst is actually trying to do. It is asked to reason about files and an analysis it cannot observe.

That is the arrangement this chapter replaces. By the end, you will have a model answering questions from inside your RStudio session, able to see the objects in your environment and the code in your editor.

Everything in this chapter is setup. None of it is analysis.

**Exercises**:

## 1.1 Install packages

Three families for packages, for three different jobs.

```
# Common R tools for data manipulation and analysis
install.packages("tidyverse")

# Bioconductor packages
BiocManager::install("DESeq2")
BiocManager::install("airway") # this takes a while

# AI integration packages
install.packages(c("gander", "ellmer"))
```

Note: 
If you encounter installation or library loading errors, you may need to restart your R session. To restart R session:
- In RStudio, go to the top menu and select Session > Restart R (or use the keyboard shortcut Cmd + Shift + F10 on Mac, or Ctrl + Shift + F10 on Windows/Linux).

## 1.2 Load Libraries

```
library(tidyverse)
library(DESeq2)
library(airway)
library(gander)
library(ellmer)
```

## 1.3 Pick your model - the API path

Pick a provider, get a key, and save it to a safe space on your local computer. Run this line in the Console, never in a saved script, and never commit a key to a repository - keep it to yourself, keep it private.

In R Console pick a provider (here we choose Google Gemini)
```
# Pick ONE provider:
Sys.setenv(GEMINI_API_KEY = "your-key-here")      # Gemini
# Sys.setenv(OPENAI_API_KEY = "your-key-here")    # OpenAI
# Sys.setenv(ANTHROPIC_API_KEY = "your-key-here") # Anthropic
```

Then point gander at a model from that provider
```
options(gander.chat = ellmer::chat_google_gemini())   # Gemini 'default' model (gemini-2.5-flash at this time)
# options(gander.chat = ellmer::chat_openai())        # GPT models
# options(gander.chat = ellmer::chat_anthropic())     # Claude models
```

Initialize the chat
```
chat <- chat_google_gemini()
```

Verify the connection before wiring it into gander
```
chat$chat("Tell me one fact about bacterial genomes")
```

Tip:
You can also explicitly declare a model (e.g., chat <- chat_google_gemini(model = "gemini-2.5-flash").

### Questions:

1. Where is the safest place to run your Sys.setenv() command containing your API key?

A) At the top of an R script that you push to GitHub
B) Interactively in the R Console, or securely stored in your .Renviron file
C) Inside a markdown documentation file (.Rmd or .qmd)
D) Hardcoded directly into a package function call

--> Correct Answer: B. nteractively in the Console or stored securely in a local .Renviron file keeps your API keys out of your code files.
Options A, C, and D are wrong and all risk accidentally exposing your secret keys if you share, publish, or commit your files to a public repository like GitHub.

2. Once you have created your chat object (chat <- chat_google_gemini()), what is the correct syntax to send a question or prompt to the model in R?

A) chat.ask("What is a data frame?")
B) chat$chat("What is a data frame?")
C) send_message(chat, "What is a data frame?")
D) chat <- prompt("What is a data frame?")

--> Correct Answer: B. (chat$chat("...").

---

## 1.4. Make a keyboard shortcut for gander

In RStudio: Navigate to **Tools → Modify Keyboard Shortcuts…** → search for "gander" → assign `Cmd+Shift+G` (Mac) or, [`Ctrl+Alt+G` (Windows/Linux)]

## 1.5. Test gander shortcut - examine 'airway' dataset

Highlight word, text a sentence or an empty space and press a keyboard shortcut (`Cmd+Shift+G`), then a dialog box will pop up where you can type your request (prompt)

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| Describe the airway dataset|


:eyes: **sample gander output**

```
RNA-seq counts for A549 cells treated with dexamethasone, including 8 samples (4 treated, 4 untreated) and associated metadata.
```

## 1.6. Use gander_peek() after gander response

Run gander_peek() in Console right after your prompt/response cycle to see what context 'gander' saw. E.g., run gander_peek() after highlighting 'airway', asking gander to 'Describe the airway dataset' and getting the output.


In Console:

```
gander_peek()
```

---

# 02. Load and qc with gander

This chapter loads the data and examines it.

**Exercises**:

## 1. Load airway dataset into your workspace

AI models often assume loading a library automatically attaches its internal datasets. This is not the case with airway dataset, which needs an additional step to be loaded into your RStudio environment as an object. So, to get an AI assistant like gander to give you the complete and correct code (which is data(airway)) - instead of just stopping at library(airway) - you need to explicitly prompt it for the data loading step.

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| I loaded library(airway), how do I load the actual dataset object?|

:eyes: **sample gander output**

```
data(airway)
```

## 2. Find out what files are available through airway dataset

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| What data and metadata are available through airway package?|

:eyes: **sample gander output**

```
The airway package provides a SummarizedExperiment object named `airway`. This object contains gene expression counts for 8 samples, with associated rowData (gene information) and colData (sample metadata, including cell line, treatment, and run accession).
```

## 3. Exaplore airway dataset

a. Airway dataset

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| what data comes with airway dataset?|

:eyes: **sample gander output**

```
The airway dataset contains:
- Gene expression counts
- Gene annotations, including gene_id, gene_name, and symbol
- Sample metadata, including SampleName, cell type, and dexamethasone treatment status
```

b.Counts and metadata

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| get airway counts and metadata|

:eyes: **sample gander output**

```
assay(airway)
colData(airway)
```

c. Explore further

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| suggest code to explore airway dataset|

```
dim(airway)
colData(airway)
rowData(airway)
assayNames(airway)
head(assay(airway, "counts"))
summary(as.vector(assay(airway, "counts")))
library(ggplot2)
library(dplyr)
as_tibble(colData(airway)) %>%
  ggplot(aes(x = dex, fill = cell)) +
  geom_bar() +
  labs(title = "Sample Distribution by Dexamethasone Treatment and Cell Type")
```

### Questions:


## 4. Filter lowly expressed genes

a. Filter counts to remove zeroes 

| Gander |  |
|:---|:---|
|**Highlight**| airway | 
|**Prompt**| Filter genes with ≥ 1 counts in all samples, create a new counts file, and summarize|

:eyes: **sample gander output**

```
filtered_counts <- assay(airway, "counts")
filtered_counts <- filtered_counts[rowSums(filtered_counts >= 1) == ncol(filtered_counts), ]
dim(filtered_counts)
summary(filtered_counts)
```

b. Ask to explain code

| Gander |  |
|:---|:---|
|**Highlight**| filtered_counts <- filtered_counts[rowSums(filtered_counts >= 1) == ncol(filtered_counts), ] | 
|**Prompt**| Annotate and explain this code|

:eyes: **sample gander output**

```
filtered_counts <-   # Assign the result back to 'filtered_counts'.
  filtered_counts[   # Select rows from 'filtered_counts'.
    rowSums(         # For each row, calculate the sum of...
      filtered_counts >= 1 # ...whether each element is greater than or equal to 1 (TRUE/FALSE treated as 1/0).
    ) == ncol(filtered_counts), # Keep rows where the sum of TRUEs (number of samples with count >= 1) equals the total number of columns (all samples).
  ] # Keep all columns for the selected rows.

Explanation: This code filters the `filtered_counts` matrix, keeping only rows (genes) where all columns (samples) have a count of 1 or greater. Rows that contain any zero counts across the samples are removed.
```

---

# 03. Deseq2 with gander


**Exercises**:

## 1. Perform differential expression analysis with DESeq2

> [!IMPORTANT]
> Highlight both objects - airway and filtered counts - for gander to see both the filtered count matrix and the associated metadata (stored in the airway object) to understand the full analysis context.


| Gander |  |
|:---|:---|
|**Highlight**| airway AND filtered_counts | 
|**Prompt**|  Perform differential expression with filtered counts using DESeq2 and `dex` as the design condition then save the results as a new object |

:eyes: **sample gander output**

```
library(DESeq2)
dds <- DESeqDataSetFromMatrix(countData = filtered_counts,
                              colData = as.data.frame(colData(airway)),
                              design = ~ dex)
dds <- DESeq(dds)
res <- results(dds)
```

## 2. Ask to explain code


| Gander |  |
|:---|:---|
|**Highlight**| *entire code above* | 
|**Prompt**|  Annotate and explain code |

:eyes: **sample gander output**

```
# Load the DESeq2 library, which provides tools for differential expression analysis
library(DESeq2)

# Create a DESeqDataSet object from a count matrix, sample information, and design formula
# countData: A matrix of raw counts (e.g., gene counts per sample)
# colData: A data frame containing sample information (e.g., experimental conditions)
#          Here, it's extracted from an existing 'airway' object's column data.
# design: A formula specifying the experimental design (e.g., ~ factor_of_interest)
#         Here, it indicates 'dex' as the primary variable for comparison.
dds <- DESeqDataSetFromMatrix(countData = filtered_counts,
                              colData = as.data.frame(colData(airway)),
                              design = ~ dex)

# Perform the DESeq2 differential expression analysis
# This function normalizes counts, estimates dispersion, and fits generalized linear models.
dds <- DESeq(dds)

# Extract the results of the differential expression analysis
# This creates a DESeqResults object containing log2 fold changes, p-values, and adjusted p-values.
res <- results(dds)
```


## 3. View results

| Gander |  |
|:---|:---|
|**Highlight**| res | 
|**Prompt**| View results |

:eyes: **sample gander output**

```
res
View(as.data.frame(res))
```

d. Ask a clarifying question

| Gander |  |
|:---|:---|
|**Highlight**| dds AND res | 
|**Prompt**|  What is the difference between dds and res objects |

:eyes: **sample gander output**

```
A DESeqDataSet (dds) object contains the raw count data, experimental design, sample information, and stores all intermediate calculations (e.g., size factors, dispersion estimates, fitted model parameters) during the DESeq2 analysis.

A DESeqResults (res) object is derived from the dds object after the differential expression analysis is complete. It contains the final results of the statistical tests for differential expression, including log2 fold changes, standard errors, p-values, and adjusted p-values for each gene.
```

---

# 0.4 Plot with gander


- PCA - build it, then understand it
- Boxplot of log2-transformed counts - build it, then break it
- MA plot - build it, then ask what it means
- Volcano plot - build it, then interpret and extend

**Exercises**:

## PCA plot

a. Make a plot

| Gander |  |
|:---|:---|
|**Highlight**| dds | 
|**Prompt**| Create a PCA plot using DESeq2 based on treatment variable |

:eyes: **sample gander output**

```
vsd <- DESeq2::vst(dds, blind = FALSE)
DESeq2::plotPCA(vsd, intgroup = "dex")
```

b. Explain code

| Gander |  |
|:---|:---|
|**Highlight**| vsd <- DESeq2::vst(dds, blind = FALSE) | 
|**Prompt**| Explain this line of code |

:eyes: **sample gander output**

```
This line performs a Variance Stabilizing Transformation (VST) on the DESeqDataSet object 'dds' using the DESeq2::vst() function. The 'blind = FALSE' argument means that the experimental design information already present in 'dds' is used to estimate the dispersion trend for the transformation, which is recommended when analyzing differential expression. The transformed data, a DESeqTransform object, is then stored in the variable 'vsd'. This transformation helps normalize variance across different mean expression levels, making the data more suitable for downstream analyses like PCA or clustering.
```

## MA plot

a. Create a plot

| Gander |  |
|:---|:---|
|**Highlight**| res | 
|**Prompt**| Create an MA plot using DESeq2 |

:eyes: **sample gander output**

b. Explain a plot

| Gander |  |
|:---|:---|
|**Highlight**| plotMA(res) | 
|**Prompt**| Explain my MA plot |

:eyes: **sample gander output**

```
An MA plot visualizes the results of a differential expression analysis.
*   **Y-axis (M-value):** Shows the log2 fold change for each gene between the two conditions (e.g., 'dex untrt vs trt').
*   **X-axis (A-value):** Shows the average of normalized counts for each gene across all samples.
*   **Red points:** Represent genes that are significantly differentially expressed (e.g., with an adjusted p-value below a certain threshold, like 0.1, as indicated by `res@metadata$alpha`).
*   **Black points:** Represent genes that are not significantly differentially expressed.
*   The horizontal line at y=0 indicates genes with no change in expression.
*   The plot helps identify genes with significant changes and visualize their expression levels relative to their overall abundance. Genes with low average expression (left side of the plot) tend to show more variability in log2 fold changes.
plotMA(res)
```

## Volcano plot

a. Create a plot

| Gander |  |
|:---|:---|
|**Highlight**| res | 
|**Prompt**| Create a volcano plot |

:eyes: **sample gander output**

```
as.data.frame(res) %>%
  dplyr::mutate(
    direction = dplyr::case_when(
      padj < 0.05 & log2FoldChange > 0 ~ "Up",
      padj < 0.05 & log2FoldChange < 0 ~ "Down",
      TRUE ~ "Not significant"
    )
  ) %>%
  ggplot2::ggplot(ggplot2::aes(x = log2FoldChange, y = -log10(padj))) +
  ggplot2::geom_point(ggplot2::aes(color = direction), alpha = 0.6, size = 1) +
  ggplot2::scale_color_manual(values = c("Up" = "red", "Down" = "blue", "Not significant" = "grey")) +
  ggplot2::geom_hline(yintercept = -log10(0.05), linetype = "dashed", color = "black") +
  ggplot2::geom_vline(xintercept = 0, linetype = "dashed", color = "black") +
  ggplot2::labs(
    title = "Volcano Plot",
    x = "log2(Fold Change)",
    y = "-log10(Adjusted p-value)",
    color = "Regulation"
  ) +
  ggplot2::theme_minimal() +
  ggplot2::theme(plot.title = ggplot2::element_text(hjust = 0.5))
```


---


# 0.5 Annotate with gander


**Exercises**:

## 1. Filter genes based on significance

| Gander |  |
|:---|:---|
|**Highlight**| res | 
|**Prompt**| Filter res based on cutoff padj < 0.01 and save this result into a csv file |

:eyes: **sample gander output**

```
res_filtered <- as.data.frame(res) |>
  dplyr::filter(padj < 0.01)
readr::write_csv(res_filtered, "filtered_results.csv")
```

## 2. Look up gene symbols

Gene symbols are already a part of the airway dataset, but they did not make it to the res or res_filtered file. By providing both, res_filtred AND airwyay as context for gander, gander will know to get gene from the dataset.

| Gander |  |
|:---|:---|
|**Highlight**| res_filtered AND airway| 
|**Prompt**| add another column to res_filtered with gene symbols |

:eyes: **sample gander output**

```
gene_symbol_lookup <- setNames(mcols(rowRanges(airway))$symbol, mcols(rowRanges(airway))$gene_id)
res_filtered$gene_symbol_airway <- gene_symbol_lookup[rownames(res_filtered)]
```

Note, you will need to type `res_filtered` to see the change

## 3. Convert Ensembl IDs to gene symbols and save the output as a csv file

Sometimes, the input files will not contain gene symbols, in which case you might want to convert your e.g. Ensembl IDs to readable gene symbols

| Gander |  |
|:---|:---|
|**Highlight**| res | 
|**Prompt**| Convert Ensembl gene IDs to readable gene symbols |

:eyes: **sample gander output**

```
library(AnnotationDbi)
library(org.Hs.eg.db)
library(dplyr)

res_filtered %>%
  tibble::rownames_to_column("ENSEMBL") %>%
  mutate(SYMBOL = mapIds(org.Hs.eg.db,
                         keys = ENSEMBL,
                         column = "SYMBOL",
                         keytype = "ENSEMBL",
                         multiVals = "first")) %>%
  tibble::column_to_rownames("ENSEMBL")
  ```

> [!WARNING]
> You will get an error \
> The error is due to required packages (e.g. org.Hs.eg.db) not installed \
> However, troubleshooting this error (e.g. by highlighting the entire output code and the entire error) may still not explicitly indicate that the problem is lack of installed packages. And the explanation may look like this:
  - The `mapIds` function, from `AnnotationDbi`, is not found.
  - Ensure `AnnotationDbi` is loaded in your R session.
> In this case, the easiest thing to do may be to manyally install these packages using this code: `BiocManager::install(c("org.Hs.eg.db", "AnnotationDbi"))`  
  
b. **Re-run code after installing the required packages**

c. Save results with gene symbols

## 4. Perform Gene Ontology analysis

a. Perform gene ontology analysis

| Gander |  |
|:---|:---|
|**Highlight**| res_filtered | 
|**Prompt**| Perform gene ontology analysis |

:eyes: **sample gander output**

```
library(clusterProfiler)
library(org.Hs.eg.db)
library(dplyr)

genes <- res_filtered %>%
  filter(padj < 0.05) %>%
  pull(SYMBOL) %>%
  na.omit() %>%
  unique()

ego <- enrichGO(gene = genes,
                OrgDb = org.Hs.eg.db,
                keyType = "SYMBOL",
                ont = "BP",
                pAdjustMethod = "BH",
                pvalueCutoff = 0.05,
                qvalueCutoff = 0.05,
                readable = TRUE)
```

> [!WARNING]
> You will likely get an error \
> You will need to install clusterProfiler before you load it, using 'BiocManager::install("clusterProfiler")'


b. Explore results

| Gander |  |
|:---|:---|
|**Highlight**| ego | 
|**Prompt**| explore gene ontology 'ego' results |

:eyes: **sample gander output**

```
dotplot(ego)
```

c. Summarize results

| Gander |  |
|:---|:---|
|**Highlight**| ego | 
|**Prompt**| Summarize ego results |

:eyes: **sample gander output**

```
summary(ego@result)
```

---

## References

- `gander` documentation: <https://github.com/tidyverse/gander>
- `ellmer` (LLM chat client): <https://ellmer.tidyverse.org/>
- https://www.bioconductor.org/packages/release/data/experiment/html/airway.html
- https://leanpub.com/library/take/fredhutch/ai_for_software/99527/4
- DESeq2 Bioconductor vignette: https://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html



