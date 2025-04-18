The PropertyInfo Component
==========================

    The PropertyInfo component allows you to get information
    about class properties by using different sources of metadata.

While the :doc:`PropertyAccess component </components/property_access>`
allows you to read and write values to/from objects and arrays, the PropertyInfo
component works solely with class definitions to provide information about the
data type and visibility - including via getter or setter methods - of the properties
within that class.

.. _`components-property-information-installation`:

Installation
------------

.. code-block:: terminal

    $ composer require symfony/property-info

.. include:: /components/require_autoload.rst.inc

Additional dependencies may be required for some of the
:ref:`extractors provided with this component <components-property-info-extractors>`.

.. _`components-property-information-usage`:

Usage
-----

To use this component, create a new
:class:`Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor` instance and
provide it with a set of information extractors::

    use Example\Namespace\YourAwesomeCoolClass;
    use Symfony\Component\PropertyInfo\Extractor\PhpDocExtractor;
    use Symfony\Component\PropertyInfo\Extractor\ReflectionExtractor;
    use Symfony\Component\PropertyInfo\PropertyInfoExtractor;

    // a full list of extractors is shown further below
    $phpDocExtractor = new PhpDocExtractor();
    $reflectionExtractor = new ReflectionExtractor();

    // list of PropertyListExtractorInterface (any iterable)
    $listExtractors = [$reflectionExtractor];

    // list of PropertyTypeExtractorInterface (any iterable)
    $typeExtractors = [$phpDocExtractor, $reflectionExtractor];

    // list of PropertyDescriptionExtractorInterface (any iterable)
    $descriptionExtractors = [$phpDocExtractor];

    // list of PropertyAccessExtractorInterface (any iterable)
    $accessExtractors = [$reflectionExtractor];

    // list of PropertyInitializableExtractorInterface (any iterable)
    $propertyInitializableExtractors = [$reflectionExtractor];

    $propertyInfo = new PropertyInfoExtractor(
        $listExtractors,
        $typeExtractors,
        $descriptionExtractors,
        $accessExtractors,
        $propertyInitializableExtractors
    );

    // see below for more examples
    $class = YourAwesomeCoolClass::class;
    $properties = $propertyInfo->getProperties($class);

Extractor Ordering
~~~~~~~~~~~~~~~~~~

The order of extractor instances within an array matters: the first non-null
result will be returned. That is why you must provide each category of extractors
as a separate array, even if an extractor provides information for more than
one category.

For example, while the :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor`
and :class:`Symfony\\Bridge\\Doctrine\\PropertyInfo\\DoctrineExtractor`
both provide list and type information it is probably better that:

* The :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor`
  has priority for list information so that all properties in a class (not
  just mapped properties) are returned.
* The :class:`Symfony\\Bridge\\Doctrine\\PropertyInfo\\DoctrineExtractor`
  has priority for type information so that entity metadata is used instead
  of type-hinting to provide more accurate type information::

    use Symfony\Bridge\Doctrine\PropertyInfo\DoctrineExtractor;
    use Symfony\Component\PropertyInfo\Extractor\ReflectionExtractor;
    use Symfony\Component\PropertyInfo\PropertyInfoExtractor;

    $reflectionExtractor = new ReflectionExtractor();
    $doctrineExtractor = new DoctrineExtractor(/* ... */);

    $propertyInfo = new PropertyInfoExtractor(
        // List extractors
        [
            $reflectionExtractor,
            $doctrineExtractor
        ],
        // Type extractors
        [
            $doctrineExtractor,
            $reflectionExtractor
        ]
    );

.. _`components-property-information-extractable-information`:

Extractable Information
-----------------------

The :class:`Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor`
class exposes public methods to extract several types of information:

* :ref:`List of properties <property-info-list>`: :method:`Symfony\\Component\\PropertyInfo\\PropertyListExtractorInterface::getProperties`
* :ref:`Property type <property-info-type>`: :method:`Symfony\\Component\\PropertyInfo\\PropertyTypeExtractorInterface::getTypes`
  (including typed properties)
* :ref:`Property description <property-info-description>`: :method:`Symfony\\Component\\PropertyInfo\\PropertyDescriptionExtractorInterface::getShortDescription` and :method:`Symfony\\Component\\PropertyInfo\\PropertyDescriptionExtractorInterface::getLongDescription`
* :ref:`Property access details <property-info-access>`: :method:`Symfony\\Component\\PropertyInfo\\PropertyAccessExtractorInterface::isReadable` and  :method:`Symfony\\Component\\PropertyInfo\\PropertyAccessExtractorInterface::isWritable`
* :ref:`Property initializable through the constructor <property-info-initializable>`:  :method:`Symfony\\Component\\PropertyInfo\\PropertyInitializableExtractorInterface::isInitializable`

.. note::

    Be sure to pass a *class* name, not an object to the extractor methods::

        // bad! It may work, but not with all extractors
        $propertyInfo->getProperties($awesomeObject);

        // Good!
        $propertyInfo->getProperties(get_class($awesomeObject));
        $propertyInfo->getProperties('Example\Namespace\YourAwesomeClass');
        $propertyInfo->getProperties(YourAwesomeClass::class);

.. _property-info-list:

List Information
~~~~~~~~~~~~~~~~

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyListExtractorInterface`
provide the list of properties that are available on a class as an array
containing each property name as a string::

    $properties = $propertyInfo->getProperties($class);
    /*
        Example Result
        --------------
        array(3) {
            [0] => string(8) "username"
            [1] => string(8) "password"
            [2] => string(6) "active"
        }
    */

.. _property-info-type:

Type Information
~~~~~~~~~~~~~~~~

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyTypeExtractorInterface`
provide :ref:`extensive data type information <components-property-info-type>`
for a property::

    $types = $propertyInfo->getTypes($class, $property);
    /*
        Example Result
        --------------
        array(1) {
            [0] =>
                class Symfony\Component\PropertyInfo\Type (6) {
                private $builtinType          => string(6) "string"
                private $nullable             => bool(false)
                private $class                => NULL
                private $collection           => bool(false)
                private $collectionKeyType    => NULL
                private $collectionValueType  => NULL
            }
        }
    */

See :ref:`components-property-info-type` for info about the ``Type`` class.

Documentation Block
~~~~~~~~~~~~~~~~~~~

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyDocBlockExtractorInterface`
can provide the full documentation block for a property as a string::

    $docBlock = $propertyInfo->getDocBlock($class, $property);
    /*
        Example Result
        --------------
        string(79):
            This is the subsequent paragraph in the DocComment.
            It can span multiple lines.
    */

.. versionadded:: 7.1

    The :class:`Symfony\\Component\\PropertyInfo\\PropertyDocBlockExtractorInterface`
    interface was introduced in Symfony 7.1.

.. _property-info-description:

Description Information
~~~~~~~~~~~~~~~~~~~~~~~

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyDescriptionExtractorInterface`
provide long and short descriptions from a properties annotations as
strings::

    $title = $propertyInfo->getShortDescription($class, $property);
    /*
        Example Result
        --------------
        string(41) "This is the first line of the DocComment."
    */

    $paragraph = $propertyInfo->getLongDescription($class, $property);
    /*
        Example Result
        --------------
        string(79):
            This is the subsequent paragraph in the DocComment.
            It can span multiple lines.
    */

.. _property-info-access:

Access Information
~~~~~~~~~~~~~~~~~~

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyAccessExtractorInterface`
provide whether properties are readable or writable as booleans::

    $propertyInfo->isReadable($class, $property);
    // Example Result: bool(true)

    $propertyInfo->isWritable($class, $property);
    // Example Result: bool(false)

The :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor` looks
for getter/isser/setter/hasser method in addition to whether or not a property is public
to determine if it's accessible. This based on how the :doc:`PropertyAccess </components/property_access>`
works. It assumes camel case style method names following `PSR-1`_. For example,
both ``myProperty`` and ``my_property`` properties are readable if there's a
``getMyProperty()`` method and writable if there's a ``setMyProperty()`` method.

.. _property-info-initializable:

Property Initializable Information
----------------------------------

Extractors that implement :class:`Symfony\\Component\\PropertyInfo\\PropertyInitializableExtractorInterface`
provide whether properties are initializable through the class's constructor as booleans::

    $propertyInfo->isInitializable($class, $property);
    // Example Result: bool(true)

:method:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor::isInitializable`
returns ``true`` if a constructor's parameter of the given class matches the
given property name.

.. tip::

    The main :class:`Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor`
    class implements all interfaces, delegating the extraction of property
    information to the extractors that have been registered with it.

    This means that any method available on each of the extractors is also
    available on the main :class:`Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor`
    class.

.. _`components-property-info-type`:

Type Objects
------------

Compared to the other extractors, type information extractors provide much
more information than can be represented as simple scalar values. Because
of this, type extractors return an array of :class:`Symfony\\Component\\PropertyInfo\\Type`
objects for each type that the property supports.

For example, if a property supports both ``integer`` and ``string`` (via
the ``@return int|string`` annotation),
:method:`PropertyInfoExtractor::getTypes() <Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor::getTypes>`
will return an array containing **two** instances of the :class:`Symfony\\Component\\PropertyInfo\\Type`
class.

.. note::

    Most extractors will return only one :class:`Symfony\\Component\\PropertyInfo\\Type`
    instance. The :class:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpDocExtractor`
    is currently the only extractor that returns multiple instances in the array.

Each object will provide 6 attributes, available in the 6 methods:

.. _`components-property-info-type-builtin`:

``Type::getBuiltInType()``
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :method:`Type::getBuiltinType() <Symfony\\Component\\PropertyInfo\\Type::getBuiltinType>`
method returns the built-in PHP data type, which can be one of these
string values: ``array``, ``bool``, ``callable``, ``float``, ``int``,
``iterable``, ``null``, ``object``, ``resource`` or ``string``.

Constants inside the :class:`Symfony\\Component\\PropertyInfo\\Type`
class, in the form ``Type::BUILTIN_TYPE_*``, are provided for convenience.

``Type::isNullable()``
~~~~~~~~~~~~~~~~~~~~~~

The :method:`Type::isNullable() <Symfony\\Component\\PropertyInfo\\Type::isNullable>`
method will return a boolean value indicating whether the property parameter
can be set to ``null``.

``Type::getClassName()``
~~~~~~~~~~~~~~~~~~~~~~~~

If the :ref:`built-in PHP data type <components-property-info-type-builtin>`
is ``object``, the :method:`Type::getClassName() <Symfony\\Component\\PropertyInfo\\Type::getClassName>`
method will return the fully-qualified class or interface name accepted.

``Type::isCollection()``
~~~~~~~~~~~~~~~~~~~~~~~~

The :method:`Type::isCollection() <Symfony\\Component\\PropertyInfo\\Type::isCollection>`
method will return a boolean value indicating if the property parameter is
a collection - a non-scalar value capable of containing other values. Currently
this returns ``true`` if:

* The :ref:`built-in PHP data type <components-property-info-type-builtin>`
  is ``array``;
* The mutator method the property is derived from has a prefix of ``add``
  or ``remove`` (which are defined as the list of array mutator prefixes);
* The `phpDocumentor`_ annotation is of type "collection" (e.g.
  ``@var SomeClass<DateTime>``, ``@var SomeClass<integer,string>``,
  ``@var Doctrine\Common\Collections\Collection<App\Entity\SomeEntity>``, etc.)

``Type::getCollectionKeyTypes()`` & ``Type::getCollectionValueTypes()``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the property is a collection, additional type objects may be returned
for both the key and value types of the collection (if the information is
available), via the :method:`Type::getCollectionKeyTypes() <Symfony\\Component\\PropertyInfo\\Type::getCollectionKeyTypes>`
and :method:`Type::getCollectionValueTypes() <Symfony\\Component\\PropertyInfo\\Type::getCollectionValueTypes>`
methods.

.. note::

    The ``list`` pseudo type is returned by the PropertyInfo component as an
    array with integer as the key type.

.. _`components-property-info-extractors`:

Extractors
----------

The extraction of property information is performed by *extractor classes*.
An extraction class can provide one or more types of property information
by implementing the correct interface(s).

The :class:`Symfony\\Component\\PropertyInfo\\PropertyInfoExtractor` will
iterate over the relevant extractor classes in the order they were set, call
the appropriate method and return the first result that is not ``null``.

.. _`components-property-information-extractors-available`:

While you can create your own extractors, the following are already available
to cover most use-cases:

ReflectionExtractor
~~~~~~~~~~~~~~~~~~~

Using PHP reflection, the :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor`
provides list, type and access information from setter and accessor methods.
It can also give the type of a property (even extracting it from the constructor
arguments), and if it is initializable through the constructor. It supports
return and scalar types::

    use Symfony\Component\PropertyInfo\Extractor\ReflectionExtractor;

    $reflectionExtractor = new ReflectionExtractor();

    // List information.
    $reflectionExtractor->getProperties($class);

    // Type information.
    $reflectionExtractor->getTypes($class, $property);

    // Access information.
    $reflectionExtractor->isReadable($class, $property);
    $reflectionExtractor->isWritable($class, $property);

    // Initializable information
    $reflectionExtractor->isInitializable($class, $property);

.. note::

    When using the Symfony framework, this service is automatically registered
    when the ``property_info`` feature is enabled:

    .. code-block:: yaml

        # config/packages/framework.yaml
        framework:
            property_info:
                enabled: true

PhpDocExtractor
~~~~~~~~~~~~~~~

.. note::

    This extractor depends on the `phpdocumentor/reflection-docblock`_ library.

Using `phpDocumentor Reflection`_ to parse property and method annotations,
the :class:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpDocExtractor`
provides type and description information. This extractor is automatically
registered with the ``property_info`` in the Symfony Framework *if* the dependent
library is present::

    use Symfony\Component\PropertyInfo\Extractor\PhpDocExtractor;

    $phpDocExtractor = new PhpDocExtractor();

    // Type information.
    $phpDocExtractor->getTypes($class, $property);
    // Description information.
    $phpDocExtractor->getShortDescription($class, $property);
    $phpDocExtractor->getLongDescription($class, $property);
    $phpDocExtractor->getDocBlock($class, $property);

.. versionadded:: 7.1

    The :method:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpDocExtractor::getDocBlock`
    method was introduced in Symfony 7.1.

PhpStanExtractor
~~~~~~~~~~~~~~~~

.. note::

    This extractor depends on the `phpstan/phpdoc-parser`_ and
    `phpdocumentor/reflection-docblock`_ libraries.

This extractor fetches information thanks to the PHPStan parser. It gathers
information from annotations of properties and methods, such as ``@var``,
``@param`` or ``@return``::

    // src/Domain/Foo.php
    class Foo
    {
        /**
         * @param string $bar
         */
        public function __construct(
            private string $bar,
        ) {
        }
    }

    // Extraction.php
    use Symfony\Component\PropertyInfo\Extractor\PhpStanExtractor;
    use App\Domain\Foo;

    $phpStanExtractor = new PhpStanExtractor();

    // Type information.
    $phpStanExtractor->getTypesFromConstructor(Foo::class, 'bar');
    // Description information.
    $phpStanExtractor->getShortDescription($class, 'bar');
    $phpStanExtractor->getLongDescription($class, 'bar');

.. versionadded:: 7.3

    The :method:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpStanExtractor::getShortDescription`
    and :method:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpStanExtractor::getLongDescription`
    methods were introduced in Symfony 7.3.

SerializerExtractor
~~~~~~~~~~~~~~~~~~~

.. note::

    This extractor depends on the `symfony/serializer`_ library.

Using :ref:`groups metadata <serializer-groups-attribute>` from the
:doc:`Serializer component </serializer>`, the
:class:`Symfony\\Component\\PropertyInfo\\Extractor\\SerializerExtractor`
provides list information. This extractor is *not* registered automatically
with the ``property_info`` service in the Symfony Framework::

    use Symfony\Component\PropertyInfo\Extractor\SerializerExtractor;
    use Symfony\Component\Serializer\Mapping\Factory\ClassMetadataFactory;
    use Symfony\Component\Serializer\Mapping\Loader\AttributeLoader;

    $serializerClassMetadataFactory = new ClassMetadataFactory(new AttributeLoader());
    $serializerExtractor = new SerializerExtractor($serializerClassMetadataFactory);

    // the `serializer_groups` option must be configured (may be set to null)
    $serializerExtractor->getProperties($class, ['serializer_groups' => ['mygroup']]);

If ``serializer_groups`` is set to ``null``, serializer groups metadata won't be
checked but you will get only the properties considered by the Serializer
Component (notably the ``#[Ignore]`` attribute is taken into account).

DoctrineExtractor
~~~~~~~~~~~~~~~~~

.. note::

    This extractor depends on the `symfony/doctrine-bridge`_ and `doctrine/orm`_
    libraries.

Using entity mapping data from `Doctrine ORM`_, the
:class:`Symfony\\Bridge\\Doctrine\\PropertyInfo\\DoctrineExtractor`
provides list and type information. This extractor is not registered automatically
with the ``property_info`` service in the Symfony Framework::

    use Doctrine\ORM\EntityManager;
    use Doctrine\ORM\Tools\Setup;
    use Symfony\Bridge\Doctrine\PropertyInfo\DoctrineExtractor;

    $config = Setup::createAnnotationMetadataConfiguration([__DIR__], true);
    $entityManager = EntityManager::create([
        'driver' => 'pdo_sqlite',
        // ...
    ], $config);
    $doctrineExtractor = new DoctrineExtractor($entityManager);

    // List information.
    $doctrineExtractor->getProperties($class);
    // Type information.
    $doctrineExtractor->getTypes($class, $property);

.. _components-property-information-constructor-extractor:

ConstructorExtractor
~~~~~~~~~~~~~~~~~~~~

The :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ConstructorExtractor`
tries to extract properties information by using either the
:class:`Symfony\\Component\\PropertyInfo\\Extractor\\PhpStanExtractor` or
the :class:`Symfony\\Component\\PropertyInfo\\Extractor\\ReflectionExtractor`
on the constructor arguments::

    // src/Domain/Foo.php
    class Foo
    {
        public function __construct(
            private string $bar,
        ) {
        }
    }

    // Extraction.php
    use App\Domain\Foo;
    use Symfony\Component\PropertyInfo\Extractor\ConstructorExtractor;

    $constructorExtractor = new ConstructorExtractor([new ReflectionExtractor()]);
    $constructorExtractor->getTypes(Foo::class, 'bar')[0]->getBuiltinType(); // returns 'string'

.. _`components-property-information-extractors-creation`:

Creating Your Own Extractors
----------------------------

You can create your own property information extractors by creating a
class that implements one or more of the following interfaces:
:class:`Symfony\\Component\\PropertyInfo\\Extractor\\ConstructorArgumentTypeExtractorInterface`,
:class:`Symfony\\Component\\PropertyInfo\\PropertyAccessExtractorInterface`,
:class:`Symfony\\Component\\PropertyInfo\\PropertyDescriptionExtractorInterface`,
:class:`Symfony\\Component\\PropertyInfo\\PropertyListExtractorInterface`,
:class:`Symfony\\Component\\PropertyInfo\\PropertyTypeExtractorInterface` and
:class:`Symfony\\Component\\PropertyInfo\\PropertyInitializableExtractorInterface`.

If you have enabled the PropertyInfo component with the FrameworkBundle,
you can automatically register your extractor class with the ``property_info``
service by defining it as a service with one or more of the following
:doc:`tags </service_container/tags>`:

* ``property_info.list_extractor`` if it provides list information.
* ``property_info.type_extractor`` if it provides type information.
* ``property_info.description_extractor`` if it provides description information.
* ``property_info.access_extractor`` if it provides access information.
* ``property_info.initializable_extractor`` if it provides initializable information
  (it checks if a property can be initialized through the constructor).
* ``property_info.constructor_extractor`` if it provides type information from the constructor argument.

  .. versionadded:: 7.3

      The ``property_info.constructor_extractor`` tag was introduced in Symfony 7.3.

.. _`PSR-1`: https://www.php-fig.org/psr/psr-1/
.. _`phpDocumentor Reflection`: https://github.com/phpDocumentor/ReflectionDocBlock
.. _`phpdocumentor/reflection-docblock`: https://packagist.org/packages/phpdocumentor/reflection-docblock
.. _`phpstan/phpdoc-parser`: https://packagist.org/packages/phpstan/phpdoc-parser
.. _`Doctrine ORM`: https://www.doctrine-project.org/projects/orm.html
.. _`symfony/serializer`: https://packagist.org/packages/symfony/serializer
.. _`symfony/doctrine-bridge`: https://packagist.org/packages/symfony/doctrine-bridge
.. _`doctrine/orm`: https://packagist.org/packages/doctrine/orm
.. _`phpDocumentor`: https://www.phpdoc.org/
