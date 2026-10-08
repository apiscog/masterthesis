# PAC2 — Estat de l'art

Aquest directori contindrà les fonts de treball i el document LaTeX de la PAC2.
La PAC té com a objectiu elaborar la base de l'estat de l'art del TFM sobre una
plataforma web per simular i comparar polítiques de planificació de tasques
asíncrones en sistemes backend.

## Estat actual

La fase actual és de delimitació i cerca. Encara no s'ha iniciat la redacció del
document final ni s'han assignat números definitius a les referències.

## Documents de treball

- `research/index-provisional.md`: estructura argumental i distribució orientativa.
- `research/protocol-cerca.md`: preguntes, fonts, cadenes de cerca i criteris.
- `research/matriu-literatura.md`: plantilla per registrar i comparar les fonts.
- `research/fonts-candidates.md`: inventari bibliogràfic provisional verificat.
- `research/lectura-analitica-ronda-1.md`: evidència, ubicació i limitacions de
  les fonts prioritàries.
- `research/taules-comparatives-provisionals.md`: síntesi inicial de polítiques
  i solucions existents.
- `research/tancament-buits-ronda-1.md`: treballs similars, fonts de mètriques i
  resultat de la cerca específica sobre cues backend.

## Compilació

Des del directori `PAC_2`:

```powershell
latexmk -xelatex -interaction=nonstopmode -halt-on-error -outdir=build main.tex
```

El document requereix XeLaTeX perquè utilitza la font Arial instal·lada al
sistema.

Els números de citació es fixaran al final, quan la bibliografia seleccionada
estigui completa i ordenada alfabèticament.
