# Le bon format n'existe pas (ça dépend qui lit)

> Markdown, JSON, XML, YAML, lequel est le meilleur pour parler à un LLM ? La question revient sans cesse, et elle est mal posée de la même façon que celle de la langue. Il n'y a pas un format optimal, il y a une relation : qui lit (un humain, le modèle, du code), pour quelle tâche (raisonner, extraire), avec quel modèle. Déplacer le danger, c'est arrêter de chercher un gagnant universel et regarder les deux moments séparément, ce qu'on donne au modèle et ce qu'il rend.

Compagnon de l'essai [La langue n'existe pas (pour un LLM)](https://github.com/12georgiadis/la-langue-n-existe-pas). Même logique, autre échelle.

## En bref

- Pas de format universel. Sur la même tâche de raisonnement, GPT-4 fait mieux en Markdown qu'en JSON, et GPT-3.5 fait l'inverse. Jusqu'à 40 % d'écart selon le seul format.
- On juge sur quatre axes qui ne pointent jamais vers le même format : coût en tokens, interprétation par le modèle, lisibilité humaine, parsing machine fiable.
- La vraie ligne de partage, c'est entrée contre sortie. À l'entrée tu optimises pour l'interprétation du modèle et le contexte. À la sortie tu optimises pour qui consomme : un humain ou du code.
- Pour Claude, en une phrase : Markdown plus balises XML en entrée, Markdown en sortie humaine, JSON en sortie machine.

---

## 1. La mauvaise question

L'intuition cherche un classement absolu : « le JSON est structuré donc fiable », « le Markdown est léger donc efficient ». Les deux sont vraies et fausses selon le contexte, exactement comme pour la langue.

L'étude la plus nette sur le sujet (Microsoft, *Does Prompt Formatting Have Any Impact on LLM Performance ?*, [arXiv 2411.10541](https://arxiv.org/html/2411.10541v1)) mesure le même prompt de raisonnement sous plusieurs formats. GPT-4 obtient 81,2 % en Markdown contre 73,9 % en JSON. Sur la même tâche, GPT-3.5 inverse le classement : JSON à 59,7 %, Markdown à 50,0 %. L'écart entre formats peut atteindre 40 % sur les modèles plus petits, et les gros modèles sont plus robustes mais jamais insensibles. Conclusion de l'étude : aucun format universellement supérieur, l'effet dépend du modèle et de la tâche.

La bonne question n'est donc pas « quel format », c'est « quel format, pour qui, et à quel moment ».

## 2. Le coût en tokens

Premier axe, le plus mesurable. Pour un même contenu, le classement du plus léger au plus lourd est robuste : **Markdown, puis YAML, puis JSON, puis XML**. Le JSON paie chaque `{`, `}`, `"` et `:` ; le XML paie chaque balise ouvrante et fermante. Les bancs d'essai indépendants donnent des ordres de grandeur cohérents : le XML coûte environ 80 % de tokens de plus que le Markdown pour la même donnée, et un objet rendu en JSON formaté pèse nettement plus que le même en YAML ([improvingagents](https://www.improvingagents.com/blog/best-nested-data-format), [ShShell](https://shshell.com/blog/token-efficiency-module-4-lesson-4-efficient-formatting)). On retient le classement, pas la décimale.

Pour des données tabulaires ou uniformes, le CSV est imbattable en taille, et un format émergent, TOON (Token-Oriented Object Notation), sérialise les jeux uniformes avec 30 à 60 % de tokens en moins que le JSON tout en gardant une structure lisible par le modèle ([spec et benchmarks TOON](https://github.com/toon-format/toon)).

## 3. Entrée : Markdown plus balises XML

À l'entrée, deux objectifs : que le modèle interprète bien, et que ça ne dévore pas le contexte. Pour Claude, la combinaison gagnante n'est pas un format unique, c'est **du Markdown pour le contenu et des balises XML pour la structure**.

Anthropic recommande explicitement les balises XML comme délimiteurs ([documentation prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). La raison est mécanique : envelopper un document de 10 000 tokens dans `<document>...</document>` ne coûte que quelques tokens et isole parfaitement ce bloc du reste pour le mécanisme d'attention. Tu écris donc tes instructions et ta prose en Markdown, léger et massivement vu à l'entraînement, et tu bornes tes sections et tes documents avec des balises XML (`<contexte>`, `<exemples>`, `<input>`). C'est l'un des rares endroits où ajouter quelques tokens (les balises) en fait gagner en fiabilité.

Pour des données tabulaires injectées en entrée, passer en CSV ou TOON économise directement du contexte sans perte de compréhension.

## 4. Sortie : qui consomme décide

À la sortie, le critère bascule sur le destinataire.

Pour un humain, Markdown sans hésiter : léger, rendu partout, naturel.

Pour du code en aval ou du tool use, JSON, malgré son coût. Mais avec un piège mesuré. L'étude *Let Me Speak Freely ?* ([arXiv 2408.02442](https://arxiv.org/abs/2408.02442), EMNLP 2024) montre que forcer un format structuré strict (JSON-mode, decodage contraint) **dégrade le raisonnement**, tout en améliorant les tâches de classification. Le mécanisme typique : le modèle met la réponse avant le raisonnement parce que le schéma l'y oblige. La parade : laisser le modèle raisonner en prose d'abord, puis émettre le JSON à la fin. Et un résultat contre-intuitif mais sourcé : demander un JSON en clair donne souvent une meilleure exactitude de génération que d'imposer une grammaire stricte ([benchmark TOON vs JSON, arXiv 2603.03306](https://arxiv.org/abs/2603.03306)).

Le YAML en sortie générée est à éviter : compact mais son indentation est fragile, une seule espace de travers casse le parsing. Le XML est robuste et bon quand la sortie mêle prose et structure imbriquée, au prix du nombre de tokens.

Le HTML, lui, n'est jamais un format d'échange avec le modèle. C'est un format de rendu final pour l'œil humain, lourd en balises, sans gain de raisonnement. On génère du Markdown, on le convertit en HTML à la toute fin. Ne demande pas à un modèle de penser en HTML.

## 5. La matrice

| Format | Coût tokens | Interprétation LLM | Édition humaine | Parsing machine | Usage idéal |
|---|---|---|---|---|---|
| Markdown | le plus léger | bon sur modèles récents | excellent | faible (ambigu) | instructions en entrée, sortie humaine |
| XML (balises) | lourd | bon (délimitation Claude) | moyen | robuste | structurer et borner sections et documents en entrée |
| JSON | verbeux | chute si forcé strict | moyen | excellent | sortie machine, tool use, API |
| YAML | compact | bon | bon | fragile (indentation) | config écrite à la main, pas de sortie générée |
| CSV / TOON | très léger (données uniformes) | bon en lecture tabulaire | bon | correct | tableaux et données uniformes en entrée |
| HTML | très lourd | faible | mauvais en brut, parfait en rendu | moyen | rendu final humain seulement |

## 6. Conclusion

Le bon format n'est pas une propriété du format. C'est une fonction de trois variables : qui lit (humain, modèle, parser), pour quelle tâche (raisonner, extraire), avec quel modèle. C'est la même leçon que pour la langue, transposée d'un cran. On croyait comparer des formats, on comparait en réalité des situations de lecture.

Pour Claude, la règle tient en une ligne : Markdown plus balises XML en entrée, Markdown en sortie pour l'humain, JSON en sortie pour la machine, et le tabulaire en CSV ou TOON quand le volume compte. Le reste est du folklore de classement absolu, et le classement absolu n'existe pas.

---

## Sources

- Does Prompt Formatting Have Any Impact on LLM Performance ? (Microsoft) : https://arxiv.org/html/2411.10541v1
- Let Me Speak Freely ? A Study on the Impact of Format Restrictions (EMNLP 2024) : https://arxiv.org/abs/2408.02442
- Token-Oriented Object Notation vs JSON, benchmark : https://arxiv.org/abs/2603.03306
- TOON, spec et benchmarks : https://github.com/toon-format/toon
- TOON, présentation InfoQ : https://www.infoq.com/news/2025/11/toon-reduce-llm-cost-tokens/
- Anthropic, prompting best practices : https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Which nested data format do LLMs understand best : https://www.improvingagents.com/blog/best-nested-data-format
- Token-efficient formatting, Markdown vs XML vs JSON : https://shshell.com/blog/token-efficiency-module-4-lesson-4-efficient-formatting

Par [Ismaël Joffroy Chandoutis](https://ismaeljoffroychandoutis.com).
