# ⚖️ Pravni asistent za iznajmljivače: RAG nad hrvatskim zakonima o turizmu

RAG (Retrieval-Augmented Generation) sustav koji odgovara na pitanja o hrvatskim zakonima iz turizma (iznajmljivanje u domaćinstvu, ugostiteljska djelatnost, usluge u turizmu, turističke zajednice) i uz svaki odgovor navodi izvor: zakon i članak.

Sve radi **lokalno**, na laptopu s RTX 5070 Laptop GPU (8 GB VRAM), bez API ključeva i bez slanja podataka vani.



## Kako radi

1. **Učitavanje i čišćenje** PDF-ova zakona
2. **Chunking po člancima**: svaki članak je zaseban dio s jasnim izvorom, a dugi članci se dijele uz preklapanje
3. **Embeddinzi** s modelom `BAAI/bge-m3` (višejezični, podržava hrvatski) i spremanje u **ChromaDB**
4. **Pretraga**: usporedba vektorske (dense), leksičke (BM25) i hibridne (Reciprocal Rank Fusion)
5. **Generiranje odgovora** lokalnim LLM-om `gemma3:4b` preko **Ollame**, uz pravilo da odgovara samo iz priloženih članaka i citira izvor
6. **Evaluacija** pretrage i odgovora
7. **Chat demo** u Gradiju

## Rezultati

|Metoda pretrage|Sintetička pitanja Hit@5|Sintetička MRR|Ručna pitanja Hit@5|Ručna MRR|
|-|-|-|-|-|
|Dense|0.75|0.56|**0.68**|**0.50**|
|BM25|0.57|0.44|0.55|0.32|
|Hibridna|**0.80**|**0.61**|0.59|0.47|

* Na sintetičkim pitanjima (koje generira LLM iz članaka) najbolja je hibridna pretraga, a na ručno napisanim pitanjima svakodnevnim jezikom najbolja je vektorska. **Sintetička evaluacija precjenjuje kvalitetu pretrage.**
* BM25 je najslabiji zbog bogate morfologije hrvatskog jezika.
* LLM-sudac je ocijenio **75 %** odgovora kao točne.
* Sustav je odbio odgovoriti na **5/5** pitanja izvan domene umjesto da izmišlja.
* Pretraga traje 20–70 ms, a cijeli odgovor 1–7 s.

Detaljna analiza i ograničenja su na kraju notebooka.

## Stack

`Python` · `sentence-transformers` · `ChromaDB` · `rank-bm25` · `Ollama` · `PyTorch (CUDA)` · `Gradio` · `pandas` · `matplotlib`

## Pokretanje

Testirano na Windowsu 11 + WSL2 (Ubuntu) s NVIDIA grafičkom karticom.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install torch --index-url https://download.pytorch.org/whl/cu128
pip install -r requirements.txt

# Ollama: https://ollama.com
ollama pull gemma3:4b
```

PDF-ove zakona stavio sam u `data/zakoni/` (neslužbeni pročišćeni tekstovi s [Ministarstva turizma i sporta](https://mint.gov.hr)). Zatim se pokreće `rag\_zakoni\_turizam.ipynb`.

## 

