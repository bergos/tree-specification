# Traversing Node elements

A `tree:Node` element __may__ have one or more `tree:relation` properties. A relation is an entity of the type `tree:Relation`, and __may__ have a more specific types. A `tree:Relation` __must__ have one `tree:node` object of the type `tree:Node`. By default, all nodes need to be followed, unless the client flags this relation for pruning.

The `tree:Relation`’s `tree:value` __must__ be set. The object of `tree:value` __should__ be accompanied by a data type when it is a literal value.

Every `tree:Relation` __should__ have a `tree:path`, indicating on what exact triple(s) the `tree:value` applies. For the different ways to express or handle a `tree:path`, we refer to [2.3.1 in the shacl specification](https://www.w3.org/TR/shacl/#x2.3.1-shacl-property-paths). Mind that all possible combinations of e.g., `shacl:alternativePath` or `shacl:inversePath` may be used. When `shacl:alternativePath` is used, the order in the list will define the importance of the order when evaluating the `tree:Relation`. If no `tree:path` is provided in this document, the `tree:value` must be compared to all objects that _can be compared to_ the `tree:value` as defined by the type of the relation.

If a `tree:path` is defined, and
 1. If no `hydra:member` relations are defined, the `tree:path` needs to be evaluated on all triples in the page
 2. If a `hydra:member` relationship is defined, the `tree:path` starts from the member URI. The target object __should__ be materialized in the current Node document, but when it is not, the object __may__ be considered implicit.
<!-- MAYBE 3. For quad representations, you can find the triple on which the `tree:path` should be evaluated by adding the graph name as an object of `tree:memberGraph` to the `hydra:Collection`. #PC: I’m unsure why to add this. I think it only adds complexity without adding real benefits to the data model, serialization, bandwidth, query performance, etc.-->

The result of the evaluation of the `tree:path`, is the value on which the `tree:value` is based.

When no `tree:path` is defined, the `tree:value` applies to the `hydra:member` elements, or all of the triples their objects given in the page when no `hydra:member` is available. If due to `rdfs:range` incompatibility, the object cannot be compared, the triple automatically becomes not part of the comparison.

Every node __may__ provide a `tree:remainingItems`. A client __may__ use `tree:remainingItems` to estimate the completeness of the downloaded elements to the end-user.

__Informative note 1__: Not having a `hydra:member` or `tree:path` may be useful for triple-based indexes such as [Triple Pattern Fragments](https://www.hydra-cg.com/spec/latest/triple-pattern-fragments/). In order to support metadata about the triples itself, something like [RDF*](http://blog.liu.se/olafhartig/tag/rdf-star/) would be needed.

__Informative note 2__: A client needs to keep a list of already visited pages, as despite this being the Tree Ontology, circular references and back-links are allowed.

__Informative note 3__: In contrast to `shacl:path`, a `tree:path` __may__ refer to an implicit property and may not materialized in the current response. The property can be inferred, fetched through another `hydra:Collection`, or simply retrieved by URI dereferencing of the subject. The latter is only recommended for a low amount of resources.

## Relation

When the _only_ type given for a certain Relation is `tree:Relation`, then the client must dereference all of the nodes. While this may seem useless, it can be used for the same use case as a `hydra:PartialCollectionView`.

For other types: see [vocabulary](../vocabulary.md) for now.

## Comparing strings

When using the `tree:PrefixRelation` or the `tree:SubstringRelation`, the strings __must__ be compared according to _case insensitive unicode ordering_.
Some flags __may__ however indicate a small derivation from this approach:

A comparison based on locale and other options can be done by using these predicates:
 1. `tree:stringComparisonLocale`: a BCP 47 language as defined in JavaScript: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl#Locale_identification_and_negotiation
 2. `tree:stringComparisonLocaleMatcher`
 3. `tree:stringComparisonUsage`
 4. `tree:stringComparisonSensitivity`
 5. `tree:stringComparisonIgnorePunctuation`
 6. `tree:stringComparisonNumeric`
 7. `tree:stringComparisonCaseFirst`

When a `tree:path` is defined, mind that you also may have to check the language of the element using the property `tree:stringComparisonLanguage`.
One or more languages __may__ be set.
When no language is set, all strings are compared.
When empty language strings only need to be compared, you have to explicitly set `tree:stringComparisonLanguage` as `""`.

__Informative note 1__: By default this thus means, when typing `à`, the links to `a` can be pruned.

__Informative note 2__: The settings used for autocompletion on the client may be different on the client than on the server. The only thing the string comparison settings are used for, is for the client to understand whether it can safely prune its search tree or not.

## Comparing geospatial features

The `tree:GeospatiallyContainsRelation` is the relation than can be used to express all further members will be contained within a geospatial region defined by the WKT String in the `tree:value`.

The `tree:path` __must__ refer to a literal containing a WKT string, such as `geosparql:asWKT`.

## Comparing time literals

When using relations such as `tree:LesserThanRelation` or `tree:GreaterThanRelation`, the time literals need to be compared according to these 3 possible data types: `xsd:date`, `xsd:dateTime` or `xsd:dateTimeStamp`.

# Compliance testing

You can test compliance if the following query executed on your page gives a valid response. You need the current page URL and bind this to `<page_url>`.

## Finding relations

```sparql
SELECT ?relationType ?value ?node
WHERE {
  ?collection void:subset|hydra:view|^dcterms:isPartOf <page_url> ;
              tree:relation ?relation .
  ?relation a ?relationType ;
            tree:value ?value ;
            tree:node ?node .
}
```

## Finding all elements in the current page

```sparql
CONSTRUCT {
  ?s ?p ?o .
} WHERE {
  ?collection void:subset|hydra:view|^dcterms:isPartOf <page_url> ;
              hydra:member ?s .
  ?s ?p ?o .
}
```