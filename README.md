
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
df <- map_df(html_files,extrair_wos)
df
#> # A tibble: 336 × 16
#>    Autor   Ano   Instituicao Endereco Pais  Titulo abstract_1 abstract_2 Indexed
#>    <chr>   <chr> <chr>       <chr>    <chr> <chr>  <chr>      <chr>      <chr>  
#>  1 Rodrig… 2021  "Universid… Univers… Port… "“O D… "Today,\r… "Com\r a … 2023-0…
#>  2 Menend… 2022  "ISCTE - I… ISCTE -… Port… "“We … "The\r co… "A\r empr… 2024-1…
#>  3 Dobrov… 2024  "Universit… Univers… Germ… "3D I… "The\r ri…  <NA>      2025-0…
#>  4 Keskin… 2021  "Syracuse … Syracus… USA   "A Bu… "An\r inc…  <NA>      2023-0…
#>  5 Ni, Zh… 2023  "Linkoping… Linkopi… Swed… "A Di… "Smart\r … "Smart\r … 2024-0…
#>  6 Vilaça… 2024  "Instituto… Institu… Port… "A Fr… "Lean\r a… "As\r met… 2025-0…
#>  7 Lei, N… 2022  "Northwest… Northwe… USA   "A Hy… "With\r t…  <NA>      2023-0…
#>  8 Hambli… 2025  "Arizona S… Arizona… USA   "A Me… "Implemen…  <NA>      2025-0…
#>  9 Uslu, … 2024  "Marmara U… Marmara… Turk… "A Mu… "Cloud\r … "Bilgi\r … 2024-1…
#> 10 Pereir… 2022  "Instituto… Institu… Port… "A No… "Electric… "Os\r últ… 2025-0…
#> # ℹ 326 more rows
#> # ℹ 7 more variables: Document_Type <chr>, Research_Areas <chr>,
#> #   Language <chr>, Accession_Number <chr>, ISBN <chr>, Advisor <chr>,
#> #   Committee <chr>
writexl::write_xlsx(df,"data_raw/wob_extraidos.xlsx")
```
