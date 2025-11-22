
<!-- README.md is generated from README.Rmd. Please edit that file -->

## TCC - Maria Eduarda - Bibliometria

``` r
library(rvest)
library(tidyverse)

# Caminho dos arquivos HTML
html_files <- list.files("data_raw/wos/",
                         full.names = TRUE,
                         pattern = "\\.htm|\\.html")
html_files <- html_files[-c(95,228)]
pg <- read_html(html_files[1])


pg |> 
    html_element("title") |> 
    html_text2() |> 
    str_replace(" Web of Science", "")
#> [1] "“O Direito à Smart City”: Desafios da Integração das \r Tecnologias Digitais e da Coesão Socio-Espacial-ProQuest ™ Dissertations\r & Theses Citation Index"

pg |> html_elements("h1") |> html_text2()
#> [1] "WOS Top Header"
```

``` r
library(rvest)
library(stringr)
library(purrr)
library(dplyr)
library(tibble)

extrair_wos <- function(caminho_html) {
  pg <- read_html(caminho_html)
  # ---- TÍTULO ----
  titulo <- pg |> 
    html_element("title") |> 
    html_text2() |> 
    str_replace(" Web of Science", "")
  # ---- ABSTRACT ----
  h2_abs <- pg |>
    html_elements("h2") |>
    (\(x) x[html_text2(x) == "Abstract"])()
  abstract <- h2_abs |>
    html_elements(xpath = "following-sibling::*[position() < 5]") |>
    html_text2()
  # ---- H3 BÁSICOS ----
  pegar_bloco <- function(label) {
    node <- pg |>
      html_elements("h3") |>
      (\(x) x[html_text2(x) == label])()
    if (length(node) == 0) return(NA_character_)
    node |>
      html_elements(xpath = "following-sibling::*[position() < 5]") |>
      html_text2() |>
      paste(collapse = "\n")
  }
  by <- pegar_bloco("By")
  year <- pegar_bloco("Published")
  source <- pegar_bloco("Source")
  address <- pegar_bloco("Addresses")
  indexed <- pegar_bloco("Indexed")
  doc_type <- pegar_bloco("Document Type")
  research_areas <- pegar_bloco("Research Areas")
  language <- pegar_bloco("Language")
  acc_number <- pegar_bloco("Accession Number")
  isbn <- pegar_bloco("ISBN")
  advisor <- pegar_bloco("Advisor")
  committee_member <- pegar_bloco("Committee member")
  
  # ---- Data frame ----
  tibble(
    Autor       = by,
    Ano         = year,
    Instituicao = source,
    Endereco    = address,
    Pais        = case_when(
      str_detect(source, "\\(") ~ str_extract(source, "(?<=\\().+?(?=\\))"),
      TRUE ~ "USA"
    ),
    Titulo      = titulo,
    abstract_1  = abstract[1],
    abstract_2  = abstract[2],
    Indexed          = indexed,
    Document_Type    = doc_type,
    Research_Areas   = research_areas,
    Language         = language,
    Accession_Number = acc_number,
    ISBN             = isbn,
    Advisor          = advisor,
    Committee        = committee_member
  )
}
df <- map_df(html_files[-c(53,109, 241,282)],extrair_wos)
df
#> # A tibble: 348 × 16
#>    Autor   Ano   Instituicao Endereco Pais  Titulo abstract_1 abstract_2 Indexed
#>    <chr>   <chr> <chr>       <chr>    <chr> <chr>  <chr>      <chr>      <chr>  
#>  1 Rodrig… 2021  Universida… Univers… Port… "“O D… "Today,\r… "Com\r a … 2023-0…
#>  2 Menend… 2022  ISCTE - In… ISCTE -… Port… "“We … "The\r co… "A\r empr… 2024-1…
#>  3 Santos… 2021  Universida… Univers… Port… "Abor… "With\r t… "Com\r a … 2023-0…
#>  4 Narang… 2025  Washington… Washing… USA   "Adva… "The\r ca…  <NA>      2025-1…
#>  5 Saini,… 2022  The Univer… The Uni… USA   "Asse… "The\r la…  <NA>      2023-0…
#>  6 Luís, … 2020  Universida… Univers… Port… "Cons… "Loyalty\…  <NA>      2024-0…
#>  7 Pires,… 2020  Universida… Univers… Port… "Cons… "A\r new …  <NA>      2024-0…
#>  8 Sanche… 2024  Universida… Univers… Colo… "Deve… "The\r ma… "La\r pre… 2025-1…
#>  9 Morris… 2022  San Diego … San Die… USA   "Fast… "With\r t…  <NA>      2023-0…
#> 10 Hoque,… 2025  Morgan Sta… Morgan … USA   "Gene… "Defect\r…  <NA>      2025-0…
#> # ℹ 338 more rows
#> # ℹ 7 more variables: Document_Type <chr>, Research_Areas <chr>,
#> #   Language <chr>, Accession_Number <chr>, ISBN <chr>, Advisor <chr>,
#> #   Committee <chr>
writexl::write_xlsx(df,"data_raw/wob_extraidos.xlsx")
```
