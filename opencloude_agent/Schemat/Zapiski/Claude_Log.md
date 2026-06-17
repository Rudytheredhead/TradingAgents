---
type: log
tags:
  - opencloude-agent
  - claude-log
---

# Claude Log

## 2026-06-16

### Fixed yfinance MultiIndex market parsing

- Root cause: `yf.download(..., group_by="ticker")` returns MultiIndex columns like `(Ticker, Price)`, so the old `MarketWatcher` looked for flat `"Close"` and `"Volume"` columns and produced `null` values in `results/continuous/watch_log.jsonl`.
- Fix: added `MarketWatcher.snapshot_from_dataframe(...)` with MultiIndex handling, corrected `_latest_close(...)` to operate on an already-selected ticker frame, and changed `benchmark_return_20d` to mean the benchmark ticker's 20d return.
- Added `relative_strength_20d = return_20d - benchmark_return_20d` to the market snapshot.
- Updated `OpportunityScanner._score(...)` to use `relative_strength_20d` directly.
- Added regression test: `test_market_watcher_parser_handles_yfinance_multiindex_columns`.
- Verification: `python -m unittest opencloude_agent.tests.test_opencloude_agent` passes.
- Runtime smoke test: one cycle now produces non-null market data and opportunities, though current market scores are below buy threshold, so decisions are still `hold`.

- Zmapowałem projekt `opencloude_agent` do pełnego szkieletu notatek w sejfie Obsidian `Schemat`.
- Dodałem notatki dla projektu, kodu, architektury, runtime, testów, wyników i indeksu.
- Zachowano konwencję wikilinków i frontmatter YAML zgodnie z [[../CLAUDE.md]].

## 2026-06-17

### Integracja AI i Modułu Pamięci (Portfolio Memory)

- Zastąpiłem logikę statyczną (score >= 60) wewnątrz `run.py` o wbudowany w projekt model `TradingAgentsGraph`, wykorzystując natywną funkcję `propagate(ticker, date)`.
- System od teraz skutecznie korzysta z prawdziwych zapytań LLM do generowania logiki transakcyjnej dla wyszukanych przez scannera "szans".
- Dodano try-except catch ze zwracanym statusem error dla `TradingAgentsGraph` aby uchronić ciągłą pętlę paper-trading przed awarią w przypadku przerw sieciowych na API zewnętrznym.
- Wprowadzono w pełni poprawną architekturę **Portfolio Memory** polegającą na natywnym `TradingMemoryLog` zapisaną w `config` agenta. Odrzucono próbę autorskiej klasy w pliku `run.py`. System bezpiecznie zachowuje w pliku `memory.log` dane w formacie tekstowym / markdown z separatorem `<!-- ENTRY_END -->` oraz odczytuje kontekst historyczny i dostarcza wiedzę z poprzednich cykli do podejmowania nowych decyzji.
- Zmodyfikowałem klasę `PaperPortfolio` do mniejszego budżetu ($5,000) i wyposażyłem z góry w portfele (AAPL, TSLA) do testowania skuteczności integracji na rzeczywistych zleceniach "Hold".
