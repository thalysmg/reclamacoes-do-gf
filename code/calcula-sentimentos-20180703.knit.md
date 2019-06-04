
<!-- rnb-text-begin -->

---
title: "Analisa sentimentos das reclamacoes"
output: html_notebook
---


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeSh0aWR5dmVyc2UpXG5saWJyYXJ5KHRpZHl0ZXh0KVxubGlicmFyeShoZXJlKVxubGlicmFyeShsZXhpY29uUFQpXG50aGVtZV9zZXQodGhlbWVfYncoKSlcbmBgYCJ9 -->

```r
library(tidyverse)
library(tidytext)
library(here)
library(lexiconPT)
theme_set(theme_bw())
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuXG4jIyAxIFBBU1NPOiBNdWRhbW9zIG8gYXJxdWl2byBhIHNlciBsaWRvLi4uXG5yZWNsYW1hY29lcyA9IHJlYWRfY3N2KGhlcmUoXCJkYXRhLzMtYXZhbGlhY2FvLWh1bWFuYS9yZWNsYW1hY29lcy1hdmFsaWFkYXMtMjAxOTA1MTUuY3N2XCIpKVxuXG4jcmVjbGFtYWNvZXMgPSByZWNsYW1hY29lc19yYXcgJT4lIFxuIyAgICBtdXRhdGUoXG4jICAgICAgICBub21lX29yZ2FvX3NpdGUgPSBvcmdhbyxcbiMgICAgICAgIG9yZ2FvID0gc3RyX3NwbGl0KGxpbmssIFwiL1wiKSAlPiUgbWFwX2Nocih+IC5bWzVdXSlcbiMgICAgKSAlPiUgXG4jICAgIGZpbHRlcihvcmdhbyAlaW4lIGMoXCJpbnNzLW1pbmlzdGVyaW8tZGEtcHJldmlkZW5jaWEtc29jaWFsXCIsICNcImFuYWMtYWdlbmNpYS1uYWNpb25hbC1kZS1hdmlhY2FvLWNpdmlsXCIpKSAlPiUgXG4jICAgIG11dGF0ZShpZCA9IDE6bigpLCBcbiMgICAgICAgICAgIGdydXBvX2F2YWxpYW5kbyA9IGlkICUlIDYgKyAxKSBcbmBgYCJ9 -->

```r

## 1 PASSO: Mudamos o arquivo a ser lido...
reclamacoes = read_csv(here("data/3-avaliacao-humana/reclamacoes-avaliadas-20190515.csv"))

#reclamacoes = reclamacoes_raw %>% 
#    mutate(
#        nome_orgao_site = orgao,
#        orgao = str_split(link, "/") %>% map_chr(~ .[[5]])
#    ) %>% 
#    filter(orgao %in% c("inss-ministerio-da-previdencia-social", #"anac-agencia-nacional-de-aviacao-civil")) %>% 
#    mutate(id = 1:n(), 
#           grupo_avaliando = id %% 6 + 1) 
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


O processo de estimativa sera muito baseado em https://sillasgonzaga.github.io/2017-09-23-sensacionalista-pt01/ . 


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGF0YShcIm9wbGV4aWNvbl92My4wXCIpXG5kYXRhKFwic2VudGlMZXhfbGVtX1BUMDJcIilcblxub3AzMCA8LSBvcGxleGljb25fdjMuMFxuc2VudCA8LSBzZW50aUxleF9sZW1fUFQwMlxuXG5nbGltcHNlKG9wMzApXG5gYGAifQ== -->

```r
data("oplexicon_v3.0")
data("sentiLex_lem_PT02")

op30 <- oplexicon_v3.0
sent <- sentiLex_lem_PT02

glimpse(op30)
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Precisamos de um dataframe onde cada observacao eh uma palavra. 


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuIyMgMiBQQVNTTzogTXVkYW1vcyBhIGNvbHVuYSBhIHNlciBsaWRhLCBkZSBcInJlY2xhbWFjb2VzXCIgcGFyYSBcInRleHRvXCIuLi5cbnBhbGF2cmFfYV9wYWxhdnJhID0gcmVjbGFtYWNvZXMgJT4lIFxuICAgIHNlbGVjdChpZCwgdGV4dG8pICU+JSBcbiAgICB1bm5lc3RfdG9rZW5zKHRlcm1vLCB0ZXh0bylcblxucGFsYXZyYV9hX3BhbGF2cmEgJT4lXG4gIHNlbGVjdChpZCwgdGVybW8pICU+JVxuICBoZWFkKDIwKVxuXG5wYWxhdnJhc19jb21fc2VudGltZW50byA9IHBhbGF2cmFfYV9wYWxhdnJhICU+JSBcbiAgbGVmdF9qb2luKG9wMzAgJT4lIHNlbGVjdCh0ZXJtLCBvcDMwID0gcG9sYXJpdHkpLCBieSA9IGMoXCJ0ZXJtb1wiID0gXCJ0ZXJtXCIpKSAlPiUgXG4gIGxlZnRfam9pbihzZW50ICU+JSBzZWxlY3QodGVybSwgc2VudCA9IHBvbGFyaXR5KSwgYnkgPSBjKFwidGVybW9cIiA9IFwidGVybVwiKSkgXG5gYGAifQ== -->

```r
## 2 PASSO: Mudamos a coluna a ser lida, de "reclamacoes" para "texto"...
palavra_a_palavra = reclamacoes %>% 
    select(id, texto) %>% 
    unnest_tokens(termo, texto)

palavra_a_palavra %>%
  select(id, termo) %>%
  head(20)

palavras_com_sentimento = palavra_a_palavra %>% 
  left_join(op30 %>% select(term, op30 = polarity), by = c("termo" = "term")) %>% 
  left_join(sent %>% select(term, sent = polarity), by = c("termo" = "term")) 
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Agora, de fato, calculamos qual a polaridade acumulada (via somatorio) de cada reclamacao e salvamos em um csv.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc2VudGltZW50b3MgPSBwYWxhdnJhc19jb21fc2VudGltZW50byAlPiUgXG4gICAgZ3JvdXBfYnkoaWQpICU+JVxuICAgIHN1bW1hcmlzZShzZW50aW1lbnRvX29wMzAgPSBzdW0ob3AzMCwgbmEucm0gPSBUUlVFKSxcbiAgICAgICAgICAgICAgcGFsYXZyYXNfb3AzMCA9IHN1bSghaXMubmEob3AzMCkpLFxuICAgICAgICAgICAgICBzZW50aW1lbnRvX3NlbnQgPSBzdW0oc2VudCwgbmEucm0gPSBUUlVFKSwgXG4gICAgICAgICAgICAgIHBhbGF2cmFzX3NlbnQgPSBzdW0oIWlzLm5hKHNlbnQpKSwgXG4gICAgICAgICAgICAgIHBhbGF2cmFzID0gbigpKVxuc2VudGltZW50b3MgJT4lIFxuICAgIHdyaXRlX2NzdihoZXJlKFwiZGF0YS81LXNlbnRpbWVudG9zL3NlbnRpbWVudG8uY3N2XCIpKVxuYGBgIn0= -->

```r
sentimentos = palavras_com_sentimento %>% 
    group_by(id) %>%
    summarise(sentimento_op30 = sum(op30, na.rm = TRUE),
              palavras_op30 = sum(!is.na(op30)),
              sentimento_sent = sum(sent, na.rm = TRUE), 
              palavras_sent = sum(!is.na(sent)), 
              palavras = n())
sentimentos %>% 
    write_csv(here("data/5-sentimentos/sentimento.csv"))
```

<!-- rnb-source-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->

