# 09. La solution `PKCE`

Proof Key for Code Exchange

`PKCE` est basé sur une solution `cryptographique` utilisant le `hashage`.

> Le hashage, en revanche, est une technique spécifique utilisée en cryptographie pour produire une valeur de hachage (ou "hash") à partir de données brutes de taille arbitraire. Cette valeur de hachage est généralement de taille fixe et est calculée de manière déterministe à partir des données d'entrée. Les fonctions de hachage sont conçues pour être à sens unique, ce qui signifie qu'il est difficile (théoriquement impossible pour une bonne fonction de hachage) de retrouver les données d'origine à partir de la valeur de hachage. Les fonctions de hachage sont largement utilisées pour l'intégrité des données, la vérification de l'authenticité des données et d'autres applications cryptographiques.
>
> **ChatGPT**

## Les paramètres de `PKCE`

- `code_verifier` un `string` cryptographiquement aléatoire
- `code_challenge`
- `code_challenge_method`