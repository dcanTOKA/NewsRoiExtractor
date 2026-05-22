# News RoI Extraction – Normal vs LLM (2×3 Ablation)
 
Bu repo, haber detay sayfalarından **News RoI** (haber başlığı + ana metin akışı) çıkarımı için iki ana yaklaşımı içerir:
 
1) **Normal (rule-based) extractor**: DOM üzerinde heuristics ile içerik çıkarır.  
2) **LLM-based extractor**: LLM’e farklı DOM kapsamı ve temsil türleriyle input vererek RoI üretir (**2×3 ablation**).
 
Ayrıca `eval.py` ile YAML dataset üzerinde **token-level metrics** ve **ROUGE-L** ölçümü yapılır.
 
---
 
## İçerik
 
- [1. News RoI tanımı](#1-news-roi-tanımı)
- [2. Yaklaşımlar](#2-yaklaşımlar)
  - [2.1 Normal extractor](#21-normal-extractor)
  - [2.2 LLM extractor (2×3 ablation)](#22-llm-extractor-23-ablation)
- [3. Promptlar](#3-promptlar)
- [4. Evaluation](#4-evaluation)
  - [4.1 Dataset formatı (YAML)](#41-dataset-formatı-yaml)
  - [4.2 Metrikler](#42-metrikler)
  - [4.3 Çalıştırma](#43-çalıştırma)
  - [4.4 Çıktılar](#44-çıktılar)
- [5. Konfigürasyon / Environment](#5-konfigürasyon--environment)
- [6. Pratik notlar ve troubleshooting](#6-pratik-notlar-ve-troubleshooting)
 
---
 
## 1) News RoI tanımı
 
Bu projede **News RoI**, haber sayfasındaki:
 
- **Başlık (H1)**
- **Haberin ana metni** (paragraflar, alt başlıklar)
 
olarak kabul edilir.
 
Amaç, sayfadaki menü / footer / sidebar / “ilgili haberler” / reklam / paylaş çağrıları gibi **boilerplate** öğeleri ayıklayıp **haber metnini mümkün olduğunca sırasını bozmadan** üretmektir.
 
---
 
## 2) Yaklaşımlar
 
### 2.1 Normal extractor
 
**Normal extractor**, BeautifulSoup + lxml ile parse edilmiş DOM üzerinde rule-based heuristics kullanır:
 
- Aday “article container” bulma (`find_article_block`)
  - ör: `section.post-detail`, `articleBody`, belirli class pattern’leri, `<article>`, fallback `<body>`
- İçerik akışı çıkarımı (`extract_text_flow`)
  - allowed tags: `p`, `h2`, `strong`, `div`
  - junk class/id filtreleri (container içinde “related/similar/author” vb.)
  - junk phrase filtreleri (örn. “paylaş”, “yorum”, “anasayfa”…)
  - min-length ve “real text” kontrolleri
- Daha geniş tag setiyle akış çıkarımı (`extract_text_flow_all`)
  - `FLOW_ALLTEXT` modları için kullanılır
  - daha fazla text-bearing element toplar
  - filtered flow’a göre daha yüksek recall, daha fazla gürültü riski taşır
- Çıktıyı Markdown benzeri bir düzende üretir (başlık + alt başlıklar + paragraflar)
 
**Normal extractor**, LLM kullanmaz; hızlıdır ve maliyeti düşüktür, ancak DOM çeşitliliği arttıkça recall kaybı yaşayabilir.
 
---
 
### 2.2 LLM extractor (2×3 ablation)
 
LLM extractor, **Scope** ve **Representation** boyutlarında **2×3 kombinasyonla** test edilir.
 
#### Factor A — Scope (kapsam)
 
- **Container scope:** heuristically seçilen article container (örn. `<article>` veya benzeri subtree)
- **Body scope:** sayfanın tüm `<body>` subtree’si
 
#### Factor B — Representation (temsil)
 
- **HTML:** subtree HTML’i temizlenir + minify edilir ve LLM’e HTML olarak verilir
- **FLOW_FILTERED:** subtree’den daha dar/filtreli bir tag-marked linear text flow çıkarılır
- **FLOW_ALLTEXT:** subtree’den daha geniş bir tag seti üzerinden tag-marked linear text flow çıkarılır
 
#### Modlar (stable IDs)
 
| Mode ID | Scope | Representation | Açıklama |
|---|---|---|---|
| `C_HTML` | Container | HTML | Seçilmiş container HTML → LLM |
| `C_FLOW_FILTERED` | Container | FLOW_FILTERED | Container’dan filtreli text-flow → LLM |
| `C_FLOW_ALLTEXT` | Container | FLOW_ALLTEXT | Container’dan daha geniş text-flow → LLM |
| `B_HTML` | Body | HTML | `<body>` HTML → LLM |
| `B_FLOW_FILTERED` | Body | FLOW_FILTERED | `<body>`’den filtreli text-flow → LLM |
| `B_FLOW_ALLTEXT` | Body | FLOW_ALLTEXT | `<body>`’den daha geniş text-flow → LLM |
 
#### Neden 2×3 ablation?
 
Bu düzenek şu soruyu incelemek için kullanılır:
 
- **Tam HTML en yüksek kaliteyi verebilir; peki daha düşük token maliyetli temsiller bu performansın ne kadarını koruyabiliyor?**
 
Bu yüzden deney yalnızca extraction quality’yi değil, aynı zamanda:
 
- input token maliyetini
- temsil türünün performansa etkisini
- scope daraltmanın faydasını
- daha filtreli vs daha geniş flow temsillerinin trade-off’unu
 
gözlemlemeyi amaçlar.
 
#### LLM çıktı kontratı
 
- LLM **yalnızca JSON** üretmelidir:
  ```json
  { "markdown": "..." }
  ```
- JSON parse edilemezse: sistem mümkünse heuristic fallback çıktısı döndürür.
 
---
 
## 3) Promptlar
 
Her mod kendi system prompt’una sahiptir (per-mode prompts).
 
`prompts/` dizini altında şu dosyalar bulunmalıdır:
 
- `news_container_html_extractor_system.jinja2` → `C_HTML`
- `news_container_flow_filtered_extractor_system.jinja2` → `C_FLOW_FILTERED`
- `news_container_flow_alltext_extractor_system.jinja2` → `C_FLOW_ALLTEXT`
- `news_body_html_extractor_system.jinja2` → `B_HTML`
- `news_body_flow_filtered_extractor_system.jinja2` → `B_FLOW_FILTERED`
- `news_body_flow_alltext_extractor_system.jinja2` → `B_FLOW_ALLTEXT`
 
Promptların ortak hedefi:
 
- Başlığı (H1) ve ana metni üretmek
- Boilerplate / noise içerikleri ayıklayıp temizlemek
- Girdide olmayan bilgi uydurmamak
- Sadece JSON döndürmek
 
`FLOW_FILTERED` ve `FLOW_ALLTEXT` promptları sistematik olarak birbirine paralel hazırlanmalıdır.  
İki mod arasındaki temel fark, metin akışının ne kadar filtreli olduğudur; prompt yapısının mümkün olduğunca benzer kalması tercih edilir.
 
---
 
## 4) Evaluation
 
`eval.py`, YAML dataset üzerinde:
 
- Normal extractor
- LLM extractor (tek mod veya tüm modlar)
 
çıktılarını çalıştırır ve metrikleri hesaplar.
 
### 4.1 Dataset formatı (YAML)
 
YAML dosyası her örnek için:
 
- `url`: haber detay sayfası
- `gt`: gold truth (başlık + metin, tek string)
 
Örnek:
 
```yaml
- url: "https://example.com/news/123"
  gt: |
    Haber Başlığı
    Birinci paragraf...
    İkinci paragraf...
```
 
Notlar:
 
- `gt` içinde başlıkların bulunduğu varsayılır.
- Eval sırasında hem `gt` hem prediction hafif normalize edilir (Markdown heading marker’ları silinir, whitespace normalize edilir).
 
### 4.2 Metrikler
 
Bu projede iki ana metrik hesaplanır:
 
1) **token_multiset PRF** (bag-of-tokens / multiset)
   - Token sırasını dikkate almaz; “içerik kapsayıcılığı” için iyi bir sinyaldir.
   - TP / FP / FN multiset kesişimi üzerinden hesaplanır.
   - **support = gold token length** (weighted average standart anlamıyla kullanılabilsin diye)
 
2) **ROUGE-L (order-aware)**
   - Token sequence üzerinde LCS (Longest Common Subsequence) ile hesaplanır.
   - Sıralama bozulmalarına duyarlıdır.
   - **support = gold token length**
 
Aggregate:
 
- Weighted average, **gold length (support)** ile ağırlıklandırılır.
 
### 4.3 Çalıştırma
 
#### Tek mod LLM
 
```bash
python eval.py --yaml dataset.yaml --out eval_out --llm-mode C_HTML
```
 
Örnekler:
 
```bash
python eval.py --yaml dataset.yaml --out eval_out --llm-mode C_FLOW_FILTERED
python eval.py --yaml dataset.yaml --out eval_out --llm-mode B_FLOW_ALLTEXT
```
 
#### Tüm modlar (6’sı birden)
 
```bash
python eval.py --yaml dataset.yaml --out eval_out --llm-run-all
```
 
#### Hızlı test (1 örnek)
 
```bash
python eval.py --yaml dataset.yaml --out eval_out_test --llm-run-all --limit 1
```
 
İsteğe bağlı:
 
- `--sleep 0.5` gibi aralıklı istek atmak için.
 
### 4.4 Çıktılar
 
Eval çıktıları `--out` dizinine yazılır:
 
- `results_normal.json`
  - Normal extractor sonuçları + aggregate metrikler
- `results_llm.json` (tek mod koşulduysa)
- `results_llm_all.json` (run-all koşulduysa)
  - Her mode için ayrı aggregate + örnek bazında sonuçlar
 
Her record genelde:
 
- `url`, `gt`, `pred_markdown`, `pred_text`
- `metrics.token_multiset` ve `metrics.rouge_l`
- LLM için ayrıca:
  - `token_stats_markdown` (input/output/total token bilgisi)
  - `stats_markdown` (LLM input stats)
  - `scope`, `family`, `representation` gibi mode metadata alanları
 
---
 
## 5) Konfigürasyon / Environment
 
### MAX_HTML_CHARS
 
LLM’e gönderilen HTML çok büyüyorsa kırpmak için:
 
```bash
export MAX_BODY_CHARS=200000
```
 
Kodda bu değer:
 
- HTML representation’da (body veya container) uygulanır
- `0` ise limitsizdir
 
### NLTK punkt
 
Tokenization için NLTK `punkt` gerekir. `eval.py` otomatik indirmeye çalışır.
 
---