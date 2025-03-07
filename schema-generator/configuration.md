# Configuration

The following options can be used in the configuration file.

## Customizing PHP Namespaces

Namespaces of generated PHP classes can be set globally, respectively for entities, enumerations and interfaces (if used
with Doctrine Resolve Target Entity Listener option).

Example:

```yaml
namespaces:
    entity: "Dunglas\EcommerceBundle\Entity"
    enum: "Dunglas\EcommerceBundle\Enum"
    interface: "Dunglas\EcommerceBundle\Model"
```

Namespaces can also be specified for a specific type. It will take precedence over any globally configured namespace.

Example:

```yaml
types:
    Thing:
        namespaces:
            class: "Dunglas\CoreBundle\Entity" # Namespace for the Thing entity (works for enumerations too)
            interface: "Schema\Model" # Namespace of the related interface
```

## Forcing a Field Range

Schema.org allows a property to have several types. However, the generator allows only one type by property. If not configured,
it will use the first defined type.
The `range` option is useful to set the type of a given property. It can also be used to force a type (even if not in the
Schema.org definition).

Example:

```yaml
types:
    Brand:
        properties:
            logo: { range: "ImageObject" } # Force the range of the logo property to ImageObject (can also be URL according to Schema.org)

    PostalAddress:
        properties:
            addressCountry: { range: "Text" } # Force the type to Text instead of Country
```

## Forcing a Field Cardinality

The cardinality of a property is automatically guessed. The `cardinality` option allows to override the guessed value.
Supported cardinalities are:

* `(0..1)`: scalar, not required
* `(0..*)`: array, not required
* `(1..1)`: scalar, required
* `(1..*)`: array, required
* `(*..0)`
* `(*..1)`
* `(*..*)`

Cardinalities are enforced by the class generator, the Doctrine ORM generator and the Symfony validation generator.

Example:

```yaml
types:
    Product:
        properties:
            sku:
                cardinality: "(0..1)"
```

## Forcing a Relation Table Name

The relation table name between two entities is automatically guessed by Doctrine. The `relationTableName` option allows
to override the default value.

This is useful when you need two entities to have more than one relation.

Example:

```yaml
types:
    Organization:
        properties:
            contactPoint: { range: Person, relationTableName: organization_contactPoint }
                member: { range: Person, cardinality: (1..*) } ## Will be default value : organization_person
```

## Forcing (or Disabling) a Class Parent

Override the guessed class hierarchy of a given type with this option.

Example:

```yaml
types:
    ImageObject:
        parent: Thing # Force the parent to be Thing instead of CreativeWork > MediaObject
        properties: ~
    Drug:
        parent: false # No parent
```

## Forcing a Class to be Abstract

Force a class to be (or to not be) `abstract`.

Example:

```yaml
types:
    Person:
        abstract: true
```

## Define Operations

API Platform operations can be added this way:

```yaml
types:
    Person:
        operations:
            item:
                get:
                    method: GET
            collection:
                get:
                    route_name: get_person_collection
```

## Forcing a Nullable Property

Force a property to be (or to not be) `nullable`.

By default this option is `true`.

Example:

```yaml
    Person:
        properties:
            name: { nullable: false }
```

The `@Assert\NotNull` constraint is automatically added.

```php
<?php

/**
 * The name of the item.
 *
 */
  #[ORM\Column]
  #[Assert\NotNull]
  private string $name;
```

## Forcing a Unique Property

Force a property to be (or to not be) `unique`.

By default this option is `false`.

Example:

```yaml
    Person:
        properties:
            email: { unique: true }
```

Output:

```php
<?php

...
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;
use Doctrine\ORM\Mapping as ORM;

/**
 * A person (alive, dead, undead, or fictional).
 *
 * @see http://schema.org/Person Documentation on Schema.org
 *
 * @Iri("http://schema.org/Person")
 */
#[ORM\Entity]
#[UniqueEntity('email')]
class Person
{
    /**
     * Email address.
     *
     */
    #[ORM\Column]
    #[Assert\Email]
    private string $email;
```

## Making a Property Read-Only

A property can be marked read-only with the following configuration:

```yaml
    Person:
        properties:
            email: { writable: false }
```

In such case, no mutator method will be generated.

## Making a Property Write-Only

A property can be marked write-only with the following configuration:

```yaml
    Person:
        properties:
            email: { readable: false }
```

In this case, no getter method will be generated.

## Forcing a Property to be in a Serialization Group

Force a property to be in a `groups`.

Enabling the `SerializerGroupsAnnotationGenerator` generator:

```yaml
annotationGenerators:
    - ApiPlatform\SchemaGenerator\AnnotationGenerator\SerializerGroupsAnnotationGenerator
  ...
```

This option expects an array of scalar value `{ groups: [ groups1, group2, ... ] }`

Example:

```yaml
    Person:
        properties:
            name: { groups: [ public ] }
```

Output:

```php
<?php

...
use Symfony\Component\Serializer\Annotation\Groups;
use Doctrine\ORM\Mapping as ORM;
use ApiPlatform\Core\Annotation\ApiProperty;
use ApiPlatform\Core\Annotation\ApiResource;

/**
 * A person (alive, dead, undead, or fictional).
 *
 * @see https://schema.org/Person Documentation on Schema.org
 *
 */
#[ORM\Entity]
#[ApiResource(iri: "https://schema.org/Person")]
class Person
{
    /**
     * The name of the item.
     *
     * @see https://schema.org/name
     *
     */
    #[ORM\Column(nullable: true)
    #[Assert\Type(type: 'string')]
    #[Groups(['public'])]
    #[ApiProperty(iri: 'https://schema.org/name')]
    private string $name;

```

## Forcing an Embeddable Class to be Embedded

Force an `embeddable` class to be `embedded`.

Example:

```yaml
    GeoCoordinates:
        embeddable: true
    Place:
        coordinates: { range: "GeoCoordinates", embedded: true, columnPrefix: false }
```

or

```yaml
    QuantitativeValue:
        embeddable: true
    Product:
        weight: { range: "QuantitativeValue", embedded: true, columnPrefix: "weight_" }
```

Output:

```php
<?php

...
use Doctrine\ORM\Mapping as ORM;
use ApiPlatform\Core\Annotation\ApiProperty;
use ApiPlatform\Core\Annotation\ApiResource;

/**
 * Any offered product or service.
 *
 * @see https://schema.org/Product Documentation on Schema.org
 *
 */
#[ORM\Entity]
#[ApiResource(iri: 'https://schema.org/Product')]
#[UniqueEntity('gtin13s')]
class Product
{
    /**
     * the weight of the product or person
     *
     * @see http://schema.org/weight
     *
     */
    #[ORM\Embedded(class: QuantitativeValue::class, columnPrefix: 'weight_')]
    #[ApiProperty(iri: 'https://schema.org/weight')]
    private ?QuantitativeValue $weight = null;

```

## Author PHPDoc

Add a `@author` PHPDoc annotation to class' DocBlock.

Example:

```yaml
author: "Kévin Dunglas <kevin@les-tilleuls.coop>"
```

## Disabling Generators and Creating Custom Ones

By default, all generators except `DunglasJsonLdApi` (API Platform v1) and `SerializerGroups` are enabled.
You can specify the list of generators to use with the `annotationGenerators` option.

Example (enabling only the PHPDoc generator):

```yaml
annotationGenerators:
    - ApiPlatform\SchemaGenerator\AnnotationGenerator\PhpDocAnnotationGenerator
```

You can write your generators by implementing the `AnnotationGeneratorInterface`.
The `AbstractAnnotationGenerator` provides helper methods
useful when creating your own generators.

Enabling a custom generator and the PHPDoc generator:

```yaml
annotationGenerators:
    - ApiPlatform\SchemaGenerator\AnnotationGenerator\PhpDocAnnotationGenerator
    - Acme\Generators\MyGenerator
```

## Skipping Accessor Method Generation

It's possible to skip the generation of accessor methods. This is particularly useful combined with the `visibility: public`
option.

To skip the generation of accessor methods, use the following config:

```yaml
accessorMethods: false
```

## Disabling the `id` Generator

By default, the generator adds a property called `id` not provided by Schema.org.
This is useful when generating an entity for use with an ORM or an ODM but not when generating DTOs.
This behavior can be disabled with the following setting:

```yaml
id:
  generate: false
```

## Generating UUIDs

It's also possible to let the DBMS generate [UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier) instead of autoincremented integers:

```yaml
id:
  generationStrategy: uuid
```

## User submitted UUIDs

To manually set a UUID instead of letting the DBMS generate it, use the following config:

```yaml
id:
  generationStrategy: uuid
  writable: true
```

## Generating Custom IDs

With this configuration option, an `$id` property of type `string` and the corresponding getters and setters will be
generated, but the DBMS will not generate anything. The ID must be set manually.

```yaml
id:
  generationStrategy: none
```

## Disabling Usage of Doctrine Collections

By default, the generator uses classes provided by the [Doctrine Collections](https://github.com/doctrine/collections) library
to store collections of entities. This is useful (and required) when using Doctrine ORM or Doctrine ODM.
This behavior can be disabled (to fallback to standard arrays) with the following setting:

```yaml
doctrine:
    useCollection: false
```

## Changing the Field Visibility

Generated fields have a `private` visibility and are exposed through getters and setters.
The default visibility can be changed with the `fieldVisibility` option.

Example:

```yaml
fieldVisibility: "protected"
```

## Generating `@Assert\Type` Annotations

It's possible to automatically generate Symfony validator's `@Assert\Type` annotations using the following config:

```yaml
validator:
  assertType: true
```

## Forcing Doctrine Inheritance Mapping Annotation

The standard behavior of the generator is to use the `@MappedSuperclass` Doctrine annotation for classes with children and
`@Entity` for classes with no child.

The inheritance annotation can be forced for a given type in the following way:

```yaml
types:
    Product:
        doctrine:
            inheritanceMapping: "@MappedSuperclass"
```

*This setting is only relevant when using the Doctrine ORM generator.*

## Interfaces and Doctrine Resolve Target Entity Listener

[`ResolveTargetEntityListener`](https://www.doctrine-project.org/projects/doctrine-orm/en/current/cookbook/resolve-target-entity-listener.html)
is a feature of Doctrine to keep modules independent. It allows to specify interfaces and `abstract` classes in relation
mappings.

If you set the option `useInterface` to true, the generator will generate an interface corresponding to each generated
entity and will use them in relation mappings.

To let PHP Schema generate the XML mapping file usable with Symfony, add the following to your config file:

```yaml
doctrine:
    resolveTargetEntityConfigPath: path/to/doctrine.xml
```

## Custom Schemas

The generator can use your own schema definitions. They must be written in RDFa and follow the format of the [Schema.org's
definition](https://schema.org/docs/schema_org_rdfa.html). This is useful to document your [Schema.org extensions](https://schema.org/docs/extension.html) and use them
to generate the PHP data model of your application.

Example:

```yaml
vocabularies:
    - https://raw.githubusercontent.com/schemaorg/schemaorg/master/data/schema.rdfa # Experimental version of Schema.org
    - http://example.com/data/myschema.rfa # Additional types
```

You can also use any other vocabulary. Check the [Linked Open Vocabularies](https://lov.okfn.org/dataset/lov/) to find one fitting your needs.

For instance, to generate a data model from the [Video Game Ontology](http://purl.org/net/VideoGameOntology), use the following config file:

```yaml
vocabularies:
  - http://vocab.linkeddata.es/vgo/GameOntologyv3.owl # The URL of the vocabulary definition

types:
  Session:
    vocabularyNamespace: http://purl.org/net/VideoGameOntology#

  # ...
```

## Checking GoodRelation Compatibility

If the `checkIsGoodRelations` option is set to `true`, the generator will emit a warning if an encountered property is not
par of the [GoodRelations](http://www.heppnetz.de/projects/goodrelations/) schema.

This is useful when generating e-commerce data models.

## PHP File Header

Prepend all generated PHP files with a custom comment.

Example:

```yaml
header: |
    /*
     * This file is part of the Ecommerce package.
     *
     * (c) Kévin Dunglas <kevin@dunglas.fr>
     *
     * For the full copyright and license information, please view the LICENSE
     * file that was distributed with this source code.
     */
```

## Full Configuration Reference

```yaml
config:

    # RDF vocabularies
    vocabularies:

        # Prototype
        -

            # RDF vocabulary to use
            uri:                  'https://schema.org/version/latest/schemaorg-current-http.rdf' # Example: 'https://schema.org/version/latest/schemaorg-current-http.rdf'

            # RDF vocabulary format
            format:               null # Example: rdfxml

    # Namespace of the vocabulary to import
    vocabularyNamespace:  'http://schema.org/' # Example: 'http://www.w3.org/ns/activitystreams#'

    # OWL relation files containing cardinality information in the GoodRelations format
    relations:            # Example: 'https://purl.org/goodrelations/v1.owl'

        # Default:
        - https://purl.org/goodrelations/v1.owl

    # Debug mode
    debug:                false

    # IDs configuration
    id:

        # Automatically add an id field to entities
        generate:             true

        # The ID generation strategy to use ("none" to not let the database generate IDs).
        generationStrategy:   auto # One of "auto"; "none"; "uuid"; "mongoid"

        # Is the ID writable? Only applicable if "generationStrategy" is "uuid".
        writable:             false

        # Set to "child" to generate the id on the child class, and "parent" to use the parent class instead.
        onClass:              child # One of "child"; "parent"

    # Generate interfaces and use Doctrine's Resolve Target Entity feature
    useInterface:         false

    # Emit a warning if a property is not derived from GoodRelations
    checkIsGoodRelations: false

    # A license or any text to use as header of generated files
    header:               false # Example: '// (c) Kévin Dunglas <dunglas@gmail.com>'

    # PHP namespaces
    namespaces:

        # The global namespace's prefix
        prefix:               null # Example: App\

        # The namespace of the generated entities
        entity:               App\Entity # Example: App\Entity

        # The namespace of the generated enumerations
        enum:                 App\Enum # Example: App\Enum

        # The namespace of the generated interfaces
        interface:            App\Model # Example: App\Model

    # Doctrine
    doctrine:

        # Use Doctrine's ArrayCollection instead of standard arrays
        useCollection:        true

        # The Resolve Target Entity Listener config file pass
        resolveTargetEntityConfigPath: null

        # Doctrine inheritance annotations (if set, no other annotations are generated)
        inheritanceAnnotations: []

    # Symfony Validator Component
    validator:

        # Generate @Assert\Type annotation
        assertType:           false

    # The value of the phpDoc's @author annotation
    author:               false # Example: 'Kévin Dunglas <dunglas@gmail.com>'

    # Visibility of entities fields
    fieldVisibility:      private # One of "private"; "protected"; "public"

    # Set this flag to false to not generate getter, setter, adder and remover methods
    accessorMethods:      true

    # Set this flag to true to generate fluent setter, adder and remover methods
    fluentMutatorMethods: false
    rangeMapping:

        # Prototype
        name:                 ~

    # Generate all types, even if an explicit configuration exists
    allTypes:             false

    # Types to import from the vocabulary
    types:

        # Prototype
        id:

            # Exclude this type, even if "allTypes" is set to true"
            exclude:              false

            # Namespace of the vocabulary of this type (defaults to the global "vocabularyNamespace" entry)
            vocabularyNamespace:  null # Example: 'http://www.w3.org/ns/activitystreams#'

            # Is the class abstract? (null to guess)
            abstract:             null

            # Is the class embeddable?
            embeddable:           false

            # Type namespaces
            namespaces:

                # The namespace for the generated class (override any other defined namespace)
                class:                null

                # The namespace for the generated interface (override any other defined namespace)
                interface:            null
            doctrine:

                # Doctrine annotations (if set, no other annotations are generated)
                annotations:          []

            # The parent class, set to false for a top level class
            parent:               false

            # If declaring a custom class, this will be the class from which properties type will be guessed
            guessFrom:            Thing

            # Operations for the class
            operations:           []

            # Import all existing properties
            allProperties:        false

            # Properties of this type to use
            properties:

                # Prototype
                id:

                    # Exclude this property, even if "allProperties" is set to true"
                    exclude:              false

                    # The property range
                    range:                null # Example: Offer

                    # The relation table name
                    relationTableName:    null # Example: organization_member
                    cardinality:          unknown # One of "(0..1)"; "(0..*)"; "(1..1)"; "(1..*)"; "(*..0)"; "(*..1)"; "(*..*)"; "unknown"

                    # The doctrine column annotation content
                    ormColumn:            null # Example: 'type="decimal", precision=5, scale=1, options={"comment" = "my comment"}'

                    # Symfony Serialization Groups
                    groups:               []

                    # The doctrine mapped by attribute
                    mappedBy:             null # Example: partOfSeason

                    # The doctrine inversed by attribute
                    inversedBy:           null # Example: episodes

                    # Is the property readable?
                    readable:             true

                    # Is the property writable?
                    writable:             true

                    # Is the property nullable?
                    nullable:             true

                    # The property unique
                    unique:               false

                    # Is the property embedded?
                    embedded:             false

                    # The property columnPrefix
                    columnPrefix:         false

    # Annotation generators to use
    annotationGenerators:

        # Defaults:
        - ApiPlatform\SchemaGenerator\AnnotationGenerator\PhpDocAnnotationGenerator
        - ApiPlatform\SchemaGenerator\AnnotationGenerator\DoctrineOrmAnnotationGenerator
        - ApiPlatform\SchemaGenerator\AnnotationGenerator\ApiPlatformCoreAnnotationGenerator
        - ApiPlatform\SchemaGenerator\AnnotationGenerator\ConstraintAnnotationGenerator
        - ApiPlatform\SchemaGenerator\AnnotationGenerator\SerializerGroupsAnnotationGenerator

    # Directories for custom generator twig templates
    generatorTemplates:   []
```
