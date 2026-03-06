# 📡 News Sentiment Terminal v2.0
### Mesa de Dinero · Quant Desk · Real-Time NLP

> Terminal de análisis de sentimiento de noticias financieras y geopolíticas en tiempo real.  
> Diseñado para operadores de mesa, traders algorítmicos y analistas cuantitativos.

---

## ¿Qué hace?

Consume feeds RSS de 7 fuentes simultáneas, aplica NLP bilingüe (EN/ES) sin API externa, y produce un dashboard de sentimiento en consola con estética Bloomberg — incluyendo heat map regional, monitor geopolítico y alertas de cambio brusco.

```
[Bloomberg] [WSJ] [Yahoo Finance] [AP News] [Al Jazeera] [Ambito] [iProfesional]
         ↓ fetch paralelo (ThreadPoolExecutor)
    NLP bilingüe → score por fuente / tema / región
         ↓
    COCKPIT · HEAT MAP · GEOPOLITICA · TITULARES EXTENDIDOS · ALERTAS
```

---

## Instalación

```bash
# 1. Clonar
git clone https://github.com/tu-usuario/news-sentiment-terminal.git
cd news-sentiment-terminal

# 2. Instalar dependencias
pip install requests feedparser colorama tabulate

# 3. Ejecutar
python news_sentiment_terminal_v2.py
```

> **Python 3.8+** requerido. Sin API keys. Sin suscripciones.

---

## Uso

```bash
# Correr una vez y salir
python news_sentiment_terminal_v2.py --una-vez

# Loop automático cada 2 minutos
python news_sentiment_terminal_v2.py --intervalo 2

# Debug (muestra errores de feeds)
python news_sentiment_terminal_v2.py --debug

# Personalizar titulares y workers
python news_sentiment_terminal_v2.py --top 10 --bottom 6 --workers 12
```

| Argumento | Default | Descripción |
|---|---|---|
| `--intervalo N` | `2` | Minutos entre refreshes |
| `--una-vez` | `False` | Correr ciclo único y salir |
| `--top N` | `8` | Titulares alcistas a mostrar |
| `--bottom N` | `5` | Titulares bajistas a mostrar |
| `--workers N` | `8` | Threads paralelos de fetch |
| `--debug` | `False` | Verbose de errores RSS |

---

## Paneles del dashboard

### 🎛 COCKPIT — Risk Overview
Score global de sentimiento, flag **RISK-ON / RISK-OFF**, barra de momentum (velocidad del cambio), distribución bull/bear/neutral y monitor geopolítico.

### 🗺 HEAT MAP REGIONAL
Score de sentimiento por región con delta vs ciclo anterior:

```
USA · Europa · China/Asia · LatAm/EM · Medio Oriente
```

### 📰 Fuentes (7 activas)

| Fuente | Idioma | Cobertura |
|---|---|---|
| Bloomberg | EN | Mercados, economía, política |
| WSJ | EN | Mercados, negocios, mundo |
| Yahoo Finance | EN | Equities, ETFs, commodities |
| AP News | EN | Breaking news, geopolítica |
| Al Jazeera | EN | Conflictos, Medio Oriente, EM |
| Ámbito Financiero | ES | Argentina macro, FX, mercados |
| iProfesional | ES | Argentina, COMEX, finanzas |

### 📊 Temas monitoreados

`Geopolítica` · `Fed/Tasas` · `Inflación` · `Macro Global` · `Argentina` · `Commodities` · `Equities` · `Crédito/FX` · `Comercio Exterior`

### ⚔ Sección Geopolítica
Feed dedicado exclusivamente a conflictos, sanciones, escaladas militares y riesgo soberano — ordenado por intensidad de señal.

### ⚡ Alertas de cambio brusco
Se disparan cuando el delta entre ciclos supera umbrales configurables:
- General: `|Δ| ≥ 18`
- Por fuente: `|Δ| ≥ 22`
- Por tema/región: `|Δ| ≥ 22`

---

## Arquitectura técnica

### Fix de freeze (problema v1 → v2)

**v1 — secuencial:**
```
Yahoo Finance: 6 feeds × (10s timeout + 0.8s delay) = 64.8s
5 fuentes secuenciales → hasta 5+ minutos de freeze
```

**v2 — paralelo:**
```python
with ThreadPoolExecutor(max_workers=8) as ex:
    futures = {ex.submit(fetch_single_feed, task) for task in all_tasks}
    # Tiempo total ≈ feed más lento (~6s), no la suma
```

### Loop blindado
```python
while True:
    try:
        prev_scores = run_once(...)
    except KeyboardInterrupt:
        break
    except Exception as e:
        # Nunca muere silenciosamente
        print(f"[ERROR ciclo #{n}]: {e} — reintentando en 30s")
        time.sleep(30)
        continue
```

### NLP sin API
- Lexicón bilingüe EN/ES: ~200 términos financieros y geopolíticos
- Negadores (`not`, `no`, `sin`, `nunca`) invierten el score localmente
- Amplificadores (`record`, `unprecedented`, `masivamente`) escalan ×1.6
- Promedio ponderado por `|score|` — los neutros no arrastran la media
- Score normalizado: `max(-100, min(100, (suma / √palabras) × 22))`

---

## Limitaciones (conocidas por diseño)

- **RSS con delay**: los feeds RSS tienen latencia de 1–15 min respecto al evento real
- **NLP heurístico**: sin modelo de lenguaje — no detecta sarcasmo ni contexto complejo
- **Feeds bloqueados**: Reuters y MSN Money no exponen RSS público consumible
- **Sin datos de precio**: no integra tick data, orderflow ni volatilidad implícita
- **Sin backtesting**: el score de sentimiento no está validado estadísticamente contra retornos

> Para uso en producción con capital real, este sistema es un **input complementario**, no una señal de trading aislada.

---

## Roadmap sugerido

- [ ] Integrar `yfinance` para mostrar precio/variación del activo mencionado inline
- [ ] Correlación histórica sentimiento → retorno (validación estadística)
- [ ] Export a CSV/Parquet para backtesting
- [ ] Webhook a Slack / Telegram para alertas
- [ ] Modelo NLP fine-tuned (FinBERT / RoBERTa-financial)
- [ ] Dashboard web con Streamlit o Dash

---

## Disclaimer

Este terminal es una herramienta de **análisis informativo**. Los scores de sentimiento son heurísticos y no constituyen asesoramiento financiero ni señales de trading. El uso de esta herramienta es responsabilidad exclusiva del operador.

---

## Licencia

MIT — libre para uso personal y comercial con atribución.
