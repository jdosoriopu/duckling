# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Duckling is a Haskell library that parses natural language text into structured data. It supports 52 languages and 13 dimensions (AmountOfMoney, CreditCardNumber, Distance, Duration, Email, Numeral, Ordinal, PhoneNumber, Quantity, Temperature, Time, Url, Volume).

## Prerequisites

- [Stack](https://docs.haskellstack.org/en/stable/) (Haskell build tool)
- PCRE development headers (`brew install pcre` on macOS)

## Build & Test Commands

```bash
# Build the library
stack build

# Regenerate classifiers, then run tests (standard development cycle)
stack build :duckling-regen-exe && stack exec duckling-regen-exe && stack test

# Run tests only (classifiers must already be up to date)
stack test

# Run the example HTTP server (port 8000)
stack exec duckling-example-exe

# Debug in REPL
stack repl --no-load
# then: :l Duckling.Debug
# then: debug (makeLocale EN $ Just US) "in two minutes" [Seal Time]
```

**Important:** Always regenerate classifiers after changing rules or corpus files and before running tests. The classifiers are ML-based and trained from corpus data.

## Architecture

### Parsing Pipeline

1. `Duckling.Core` / `Duckling.Api` — public API entry points (`parse`, `analyze`)
2. `Duckling.Engine` — pattern matching engine applies rules to input text using regex and predicate matching
3. `Duckling.Resolve` — resolves matched tokens into typed dimension values
4. `Duckling.Ranking` — ML classifiers disambiguate and rank candidate parses

### Key Directory Layout

- `Duckling/<Dimension>/<Lang>/Rules.hs` — parsing rules (patterns + production functions)
- `Duckling/<Dimension>/<Lang>/Corpus.hs` — test examples (also used for classifier training)
- `Duckling/<Dimension>/Types.hs` — data types for each dimension
- `Duckling/Dimensions/<Lang>.hs` — which dimensions are enabled per language
- `Duckling/Rules/<Lang>.hs` — which rules are loaded per language
- `Duckling/Ranking/Classifiers/<Locale>.hs` — generated classifier files (do not edit manually)
- `tests/Duckling/<Dimension>/<Lang>/Tests.hs` — test modules
- `exe/` — executables (server example, classifier regen, custom dimension example)

### Rule System

Rules have three parts: a name, a pattern, and a production function. Patterns combine character-level matching (regexes) with concept-level matching (predicates on tokens). Productions take matched tokens and return a new token.

### Corpus & Testing

Corpus files define positive examples (should parse) and negative examples (should not parse). The reference time for all corpus tests is **Tuesday Feb 12, 2013 at 4:30am UTC-2**. Test predicates have the signature `Context -> ResolvedToken -> Bool`.

## Extending Duckling

### Adding a new dimension to an existing language

Update these 4 files:
1. `Duckling/<Dimension>/<Lang>/Rules.hs` — add parsing rules
2. `Duckling/<Dimension>/<Lang>/Corpus.hs` — add test examples
3. `Duckling/Dimensions/<Lang>.hs` — register the dimension (if not in `Common.hs`)
4. `Duckling/Rules/<Lang>.hs` — register the rules

Then regenerate classifiers and run tests.

### Adding a new language

- Use [ISO-639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) language codes
- Start by implementing the `Numeral` dimension first

## Coding Style

- 80-character line limit
- Imports: one block for standard/external packages, one block for project modules. Qualified imports at the end of each block, alphabetical ordering.
- Match the style of surrounding code
- HLint is enabled (`.hlint.yaml` ignores "Do not use -1")
