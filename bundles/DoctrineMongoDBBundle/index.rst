DoctrineMongoDBBundle
=====================

.. toctree::
   :hidden:

   config
   form

L'Object Document Mapper (ODM) `MongoDB`_ assomiglia all'ORM di Doctrine2 per
filosofia e funzionamento, In altre parole, come l':doc:`ORM di Doctrine2</book/doctrine>`,
con l'ODM di Doctrine si ha a che fare solo con oggetti PHP puri, che sono persistiti
in modo trasparente a MongoDB.

.. tip::

    Si può approfondire l'ODM MongoDB di Doctrine nella `documentazione`_ del progetto.

È disponibile un bundle che integra l'ODM MongoDB di Doctrine in Symfony,
facilitandone la configurazione e l'utilizzo.

.. note::

    Questo capitolo assomiglia molto al :doc:`capitolo sull'ORM Doctrine2</book/doctrine>`,
    che parla di come l'ORM di Doctrine2 possa essere usato per persistere dati in una
    base dati relazionale (p.e. MySQL). La somiglianza è intenzionale: che si persista in
    una base dati relazionale con ORM o in MongoDB con ODM, la filosofia è quasi
    la stessa.

Installazione
-------------

Per usare l'ODM MongoDB, occorrono due librerie fornite da Doctrine e un bundle che le
integri in Symfony.

Installare il bundle tramite Composer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Per installare DoctrineMongoDBBundle con Composer, aggiungere le seguenti righe al
file `composer.json`::

    {
        "require": {
            "doctrine/mongodb-odm": "1.0.*@dev",
            "doctrine/mongodb-odm-bundle": "3.0.*@dev"
        },
    }

La definizione ``@dev`` è necessaria, perché al momento non sono disponibili versioni del bundle e di
ODM stabili. A seconda delle necessità di un progeto, si potrebbero usare altre stringhe della versione,
come spiegato nella `documentazione sullo schema`_ di Composer.

Ora si possono installare le nuove dipendenza, eseguendo il comando ``update``
di Composer, dalla cartella in cui si trova il file ``composer.json``:

.. code-block :: bash

    php composer.phar update

Composer scaricherà tutti i file necessari e li installerà.


Registrare le annotazioni e il bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Quindi, registrare la libreria delle annotazioni, aggiungendo il seguente all'autoloader
(sotto la riga, già esistente, ``AnnotationRegistry::registerLoader``)::

    // app/autoload.php
    use Doctrine\ODM\MongoDB\Mapping\Driver\AnnotationDriver;
    AnnotationDriver::registerAnnotationClasses();

Infine, abilitare il bundle nel kernel e registrare il
bundle::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Doctrine\Bundle\MongoDBBundle\DoctrineMongoDBBundle(),
        );

        // ...
    }

Si è ora pronti per lavorare.

Configurazione
--------------

Per iniziare, sono necessarie alcune configurazioni di base, che impostano il document
manager. Il modo più semplice è abilitare ``auto_mapping``, che attiverà
l'ODM MongoDB su tutta l'applicazione:

.. code-block:: yaml

    # app/config/config.yml
    doctrine_mongodb:
        connections:
            default:
                server: mongodb://localhost:27017
                options: {}
        default_database: test_database
        document_managers:
            default:
                auto_mapping: true

.. note::

    Ovviamente, occorre anche assicurarsi che il server MongoDB stia girando.
    Per maggiori dettagli, vedere la guida `Quick Start`_ di MongoDB.

Un semplice esempio: un prodotto
--------------------------------

Il modo migliore per capire l'ODM MongoDB ODM è vederlo in azione. In questa sezione,
affronteremo tutti i passi necessari per iniziare a persistere documenti in
MongoDB.

.. sidebar:: Codice con l'esempio

    Se si vuole seguire l'esempio in questo capitolo, creare
    ``AcmeStoreBundle``, con:

    .. code-block:: bash

        php app/console generate:bundle --namespace=Acme/StoreBundle

Creare una classe documento
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si supponga di dover costruire un'applicazione in cui devono essere mostrati dei prodotti.
Senza nemmeno pensare a Doctrine o a MongoDB, già si sa di aver bisogno di un oggetto
``Product``, che rappresenti tali prodotti. Creare questa classe nella cartella
``Document`` di ``AcmeStoreBundle``::

    // src/Acme/StoreBundle/Document/Product.php
    namespace Acme\StoreBundle\Document;

    class Product
    {
        protected $name;

        protected $price;
    }

La classe, chiamata spesso "documento", che significa *una classe di base che contiene
dati*, è semplice e aiuta a soddisfare i requisiti per i prodotti
dell'applicazione. Questa classe non può ancora essere persistita in MongoDB, è solo una
semplice classe PHP.

Aggiungere informazioni di mappatura
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine consente di lavorare con MongoDB in un modo molto più interessante rispetto
al semplice recupero di dati in un array. Invece, Doctrine
consente di persistere interi *oggetti* in MongoDB e di recuperare interi oggetti
da MongoDB. Funziona mappando una classe PHP le sue proprietà su elementi di
una collezione di MongoDB.

Per fare in modo che Doctrine possa fare ciò, occorre solo creare dei "metadati", ovvero
la configurazione che dice esattamente a Doctrine come la classe ``Product`` e le sue
proprietà debbano essere *mappate* su MongoDB. Questi metadati possono essere specificati
in diversi formati, inclusi YAML, XML o direttamente dentro la classe ``Product``,
tramite annotazioni:

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Document/Product.php
        namespace Acme\StoreBundle\Document;

        use Doctrine\ODM\MongoDB\Mapping\Annotations as MongoDB;

        /**
         * @MongoDB\Document
         */
        class Product
        {
            /**
             * @MongoDB\Id
             */
            protected $id;

            /**
             * @MongoDB\String
             */
            protected $name;

            /**
             * @MongoDB\Float
             */
            protected $price;
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.mongodb.yml
        Acme\StoreBundle\Document\Product:
            fields:
                id:
                    id:  true
                name:
                    type: string
                price:
                    type: float

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.mongodb.xml -->
        <doctrine-mongo-mapping xmlns="http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping
                            http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping.xsd">

            <document name="Acme\StoreBundle\Document\Product">
                <field fieldName="id" id="true" />
                <field fieldName="name" type="string" />
                <field fieldName="price" type="float" />
            </document>
        </doctrine-mongo-mapping>

Doctrine consente di scegliere tra una grande varietà di tipi di campo, ognuno
con le sue opzioni Per informazioni sui tipi disponibili, vedere la sezione
:ref:`cookbook-mongodb-field-types`.

.. seealso::

    Si può anche consultare la `Documentazione di base sulla mappatura`_ di Doctrine
    per tutti i dettagli sulla mappatura. Se si usano le annotazioni, occorrerà
    aggiungere a ogni annotazione il prefisso ``MongoDB\`` (p.e. ``MongoDB\String``),
    che non è mostrato nella documentazione di Doctrine. Occorrerà anche includere
    l'istruzione ``use Doctrine\ODM\MongoDB\Mapping\Annotations as MongoDB;``, che
    *importa* il prefisso ``MongoDB`` delle annotazioni.

Generare getter e setter
~~~~~~~~~~~~~~~~~~~~~~~~

Sebbene ora Doctrine sappia come persistere un oggetto ``Product`` in MongoDB,
la classe stessa non è molto utile. Poiché ``Product`` è solo una normale classe
PHP, occorre creare dei metodi getter e setter (p.e. ``getName()``, ``setName()``)
per poter accedere alle sue proprietà (essendo le proprietà protette).
Fortunatamente, Doctrine può farlo al posto nostro, basta eseguire:

.. code-block:: bash

    php app/console doctrine:mongodb:generate:documents AcmeStoreBundle

Il comando si assicura che i getter e i setter siano generati per la classe
``Product``. È un comando sicuro, lo si può eseguire diverse volte: genererà i
getter e i setter solamente se non esistono (ovvero non sostituirà eventuali
metodi già presenti).

.. note::

    Doctrine non si cura se le proprietà sono ``protected`` o ``private``,
    o se siano o meno presenti getter o setter per una proprietà.
    I getter e i setter sono generati qui solo perché necessari per
    interagire con l'oggetto PHP.

Persistere gli oggetti in MongoDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ora che il documento ``Product`` è stato mappato ed è completo di getter e setter,
si è pronti per persistere i dati in MongoDB. Da dentro un controllore, è
molto facile. Aggiungere il seguente metodo a ``DefaultController``
del bundle:

.. code-block:: php
    :linenos:

    // src/Acme/StoreBundle/Controller/DefaultController.php
    use Acme\StoreBundle\Document\Product;
    use Symfony\Component\HttpFoundation\Response;
    // ...

    public function createAction()
    {
        $product = new Product();
        $product->setName('A Foo Bar');
        $product->setPrice('19.99');

        $dm = $this->get('doctrine_mongodb')->getManager();
        $dm->persist($product);
        $dm->flush();

        return new Response('Created product id '.$product->getId());
    }

.. note::

    Se si sta seguendo questo esempio, occorrerà creare una
    rotta che punti a questa azione, per poterla vedere in azione.

Analizziamo questo esempio:

* **righe 8-10** In questa sezione, si istanzia e si lavora con l'oggetto
  ``$product``, come qualsiasi altro normale oggetto PHP;

* **riga 12** Questa riga recupera l'oggetto *gestore di documenti* di Doctrine,
  responsabile della gestione del processo di persistenza e del recupero di
  oggetti in MongoDB;

* **riga 13** Il metodo ``persist()`` dice a Doctrine di "gestire" l'oggetto
  ``$product``. Questo non fa (ancora) eseguire una query su MongoDB.

* **riga 14** Quando il metodo ``flush()`` è richiamato, Doctrine cerca tutti gli
  oggetti che sta gestendo, per vedere se hanno bisogno di essere persistiti su MongoDB
  In questo esempio, l'oggetto ``$product`` non è stato ancora persistito, quindi il
  gestore di documenti esegue una query su MongoDB e crea una nuova voce.

.. note::

    Di fatto, essendo Doctrine consapevole di tutte le entità gestite,
    quando si chiama il metodo ``flush()``, esso calcola un insieme globale di
    modifiche ed esegue l'operazione più efficiente possibile.

Quando si creano o aggiornano oggetti, il flusso è sempre lo stesso. Nella prossima
sezione, si vedrà come Doctrine sia abbastanza intelligente da aggiornare le voci
che già esistono in MongoDB.

.. tip::

    Doctrine fornisce una libreria che consente di caricare dati di test
    in un progetto (le cosiddette "fixture"). Per informazioni, vedere
    :doc:`/bundles/DoctrineFixturesBundle/index`.

Recuperare oggetti da MongoDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuperare un oggetto da MongoDB è ancora più facile. Per esempio,
supponiamo di aver configurato una rotta per mostrare uno specifico ``Product``,
in base al valore del suo ``id``::

    public function showAction($id)
    {
        $product = $this->get('doctrine_mongodb')
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        // fare qualcosa, come passare l'oggetto $product a un template
    }

Quando si cerca un particolare tipo di oggetto, si usa sempre quello che è noto
come il suo "repository". Si può pensare a un repository come a una classe PHP il cui
unico compito è quello di aiutare nel recuperare entità di una certa classe. Si può
accedere all'oggetto repository per una classe documento tramite::

    $repository = $this->get('doctrine_mongodb')
        ->getManager()
        ->getRepository('AcmeStoreBundle:Product');

.. note::

    La stringa ``AcmeStoreBundle:Product`` è una scorciatoia utilizzabile ovunque in
    Doctrine al posto del nome intero della classe dell'entità (cioè ``Acme\StoreBundle\Entity\Product``).
    Questo funzionerà finché i documenti rimarranno sotto lo spazio dei nomi ``Document``
    del bundle.

Una volta ottenuto il repository, si avrà accesso a tanti metodi utili::

    // cerca per chiave primaria (di solito "id")
    $product = $repository->find($id);

    // nomi di metodi dinamici per cercare in base al valore di una colonna
    $product = $repository->findOneById($id);
    $product = $repository->findOneByName('foo');

    // trova *tutti* i prodotti
    $products = $repository->findAll();

    // trova un gruppo di prodotti in base a un valore arbitrario di una colonna
    $products = $repository->findByPrice(19.99);

.. note::

    Si possono ovviamente fare anche query complesse, su cui si può avere maggiori
    informazioni nella sezione :ref:`book-doctrine-queries`.

Si possono anche usare gli utili metodi ``findBy`` e ``findOneBy`` per
recuperare facilmente oggetti in base a condizioni multiple::

    // cerca un prodotto in base a nome e prezzo
    $product = $repository->findOneBy(array('name' => 'foo', 'price' => 19.99));

    // cerca tutti i prodotti in base al nome, ordinati per prezzo
    $product = $repository->findBy(
        array('name' => 'foo'),
        array('price', 'ASC')
    );

Aggiornare un oggetto
~~~~~~~~~~~~~~~~~~~~~

Una volta che Doctrine ha recuperato un oggetto, il suo aggiornamento è facile. Supponiamo
di avere una rotta che mappi un id di prodotto a un'azione di aggiornamento in un controllore::

    public function updateAction($id)
    {
        $dm = $this->get('doctrine_mongodb')->getManager();
        $product = $dm->getRepository('AcmeStoreBundle:Product')->find($id);

        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        $product->setName('New product name!');
        $dm->flush();

        return $this->redirect($this->generateUrl('homepage'));
    }

L'aggiornamento di un oggetto si svolge in tre passi:

1. recuperare l'oggetto da Doctrine;
2. modificare l'oggetto;
3. richiamare ``flush()`` sul gestore di documenti

Si noti che non è necessario richiamare ``$dm->persist($product)``. Ricordiamo che
questo metodo dice semplicemente a Doctrine di gestire o "osservare" l'oggetto ``$product``.
In questo caso, poiché l'oggetto ``$product`` è stato recuperato da Doctrine, è
già gestito.

Cancellare un oggetto
~~~~~~~~~~~~~~~~~~~~~

La cancellazione di un oggetto è molto simile, ma richiede una chiamata al metodo
``remove()`` del gestore dei documenti::

    $dm->remove($product);
    $dm->flush();

Come ci si potrebbe aspettare, il metodo ``remove()`` rende noto a Doctrine che si
vorrebbe rimuovere la data entità dalla base dati. Tuttavia, l'operazione di cancellazione non viene
realmente eseguita finché non si richiama il metodo ``flush()``.

Cercare gli oggetti
-------------------

Come già visto, la classe repository consente di cercare uno o più oggetti, in base
a diversi parametri. Quando ciò è sufficiente, è il modo più facile per recuperare
documenti. Ovviamente, si possono anche creare query più
complesse.

Usare query builder
~~~~~~~~~~~~~~~~~~~

l'ODM di Doctrine ha un oggetto "Builder", che consente di costruire una query
che restituisce esattamente i documenti desiderati. Se si usa un IDE, si può anche
trarre vantaggio dall'auto-completamento durante la scrittura dei nomi dei metodi.
Da dentro un controllore::

    $products = $this->get('doctrine_mongodb')
        ->getManager()
        ->createQueryBuilder('AcmeStoreBundle:Product')
        ->field('name')->equals('foo')
        ->limit(10)
        ->sort('price', 'ASC')
        ->getQuery()
        ->execute()

In questo caso, saranno restituiti 10 prodotti con nome "foo", ordinati da quello
con il prezzo più basso a quello con il prezzo più alto.

L'oggetto ``QueryBuilder`` contiene tutti i metodi necessari per costruire
una query. Per maggiori informazioni su query builder, consultare la
documentazione di Doctrine `Query Builder`_. Per la lista delle condizioni disponibili
per una query, vedere la documentazione `Operatori condizionali`_.

Classi repository personalizzate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nelle sezioni precedenti, si è iniziato costruendo e usando query più complesse da
dentro un controllore. Per isolare, testare e riusare queste query, è una buona idea
creare una classe repository personalizzata per l'entità e aggiungere
metodi, con la logica delle query, al suo interno.

Per farlo, aggiungere il nome della classe del repository alla definizione della mappatura.

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Document/Product.php
        namespace Acme\StoreBundle\Document;

        use Doctrine\ODM\MongoDB\Mapping\Annotations as MongoDB;

        /**
         * @MongoDB\Document(repositoryClass="Acme\StoreBundle\Repository\ProductRepository")
         */
        class Product
        {
            //...
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.mongodb.yml
        Acme\StoreBundle\Document\Product:
            repositoryClass: Acme\StoreBundle\Repository\ProductRepository
            # ...

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.mongodb.xml -->
        <!-- ... -->
        <doctrine-mongo-mapping xmlns="http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping
                            http://doctrine-project.org/schemas/odm/doctrine-mongo-mapping.xsd">

            <document name="Acme\StoreBundle\Document\Product"
                    repository-class="Acme\StoreBundle\Repository\ProductRepository">
                <!-- ... -->
            </document>

        </doctrine-mong-mapping>

Doctrine può generare la classe repository, eseguendo:

.. code-block:: bash

    php app/console doctrine:mongodb:generate:repositories AcmeStoreBundle

Quindi, aggiungere un nuovo metodo, chiamato ``findAllOrderedByName()``, alla classe
repository appena generata. Questo metodo cercherà tutti i documenti ``Product``,
ordinati alfabeticamente.

.. code-block:: php

    // src/Acme/StoreBundle/Repository/ProductRepository.php
    namespace Acme\StoreBundle\Repository;

    use Doctrine\ODM\MongoDB\DocumentRepository;

    class ProductRepository extends DocumentRepository
    {
        public function findAllOrderedByName()
        {
            return $this->createQueryBuilder()
                ->sort('name', 'ASC')
                ->getQuery()
                ->execute();
        }
    }

Si può usare il metodo appena creato, proprio come i metodi predefiniti del repository::

    $products = $this->get('doctrine_mongodb')
        ->getManager()
        ->getRepository('AcmeStoreBundle:Product')
        ->findAllOrderedByName();


.. note::

    Quando si usa una classe repository personalizzata, si ha ancora accesso ai metodi
    predefiniti di ricerca, come ``find()`` e ``findAll()``.

Estensioni Doctrine: Timestampable, Sluggable, ecc.
---------------------------------------------------

Doctrine è alquanto flessibile e diverse estensioni di terze parti sono disponibili,
consentendo di eseguire facilmente compiti comuni e ripetitivi sui documenti.
Sono inclusi *Sluggable*, *Timestampable*, *Loggable*, *Translatable* e
*Tree*.

Per maggiori informazioni su come trovare e usare tali estensioni, vedere la ricetta
:doc:`usare le estensioni comuni di Doctrine</cookbook/doctrine/common_extensions>`.

.. _cookbook-mongodb-field-types:

Riferimento sui tipi di campo di Doctrine
-----------------------------------------

Doctrine ha un gran numero di tipi di campo a disposizione. Ognuno di questi mappa
un tipo di dato PHP su uno specifico `tipo MongoDB`_. I seguenti sono solo *alucni* dei
tipi supportati in Doctrine:

* ``string``
* ``int``
* ``float``
* ``date``
* ``timestamp``
* ``boolean``
* ``file``

Per maggiori informazioni, vedere `Documentazione sulla mappatura dei tipi`_.

.. index::
   single: Doctrine; Comandi console ODM
   single: CLI; ODM Doctrine

Comandi da console
------------------

L'integrazione con l'ODM Doctrine2 offre diversi comandi da console, sotto lo spazio
dei nomi ``doctrine:mongodb``. Per vedere la lista dei comandi, si può eseguire la
console senza parametri:

.. code-block:: bash

    php app/console

Verrà mostrata una lista dei comandi disponibili, molti dei quali iniziano
col prefisso ``doctrine:mongodb``. Si possono trovare maggiori informazioni eseguendo il
comando ``help``. Per esempio, per ottenere dettagli sul task
``doctrine:mongodb:query``, eseguire:

.. code-block:: bash

    php app/console help doctrine:mongodb:query

.. note::

   Per poter caricare fixture in MongoDB, occorre avere il bundle
   ``DoctrineFixturesBundle`` installato. Per sapere come fare, leggere la voce
   ":doc:`/bundles/DoctrineFixturesBundle/index`" della
   documentazione.

.. index::
   single: Configurazione; ODM MongoDB Doctrine
   single: Doctrine; Configurazione ODM MongoDB

Configurazione
--------------

Per informazioni dettagliate sulle opzioni di configurazione disaponibili per l'uso
dell'ODM di Doctrine, vedere la sezione :doc:`Riferimenti MongoDB</bundles/DoctrineMongoDBBundle/config>`.

Registrare ascoltatori e sottoscrittori di eventi
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine consente di registrare ascoltatori e sottoscrittori di eventi, che sono
notificati quando i vari eventi accadono all'interno dell'ODM di Doctrine. Per ulteriori
informazioni, vedere la `Documentazione sugli eventi`_ di Doctrine.

.. tip::

    Oltre agli eventi dell'ODM, si possono ascoltare anche eventi a basso livello di MongoDBIn addition to ODM events, you may also listen on lower-level MongoDB,
    che si possono trovare definiti nella classe ``Doctrine\MongoDB\Events``.

.. note::

    Ciascuna connessione in Doctrine ha il suo gestore di eventi, condiviso con
    i gestori di documenti legati a tale connessione. Gli ascoltatori e i sottoscrittori possono
    essere registrati con tutti i gestori di eventi o solo con uno (usando il nome della connessione).

In Symfony, si possono registrare ascoltatori o sottoscrittori, creando un :term:`servizio`
e :ref:`taggandolo<book-service-container-tags>` uno specifico tag.

*   **ascoltatore di eventi**: Usare il tag ``doctrine_mongodb.odm.event_listener`` pe
    registrare un ascoltatore. L'attributo ``event`` è obbligatorio e denota
    l'evento da ascoltare. Per impostazione predefinita, tutti gli ascoltatori saranno registrati con
    i gestori di eventi per tutte le connessioni. Per limitare un ascoltatore a una singola
    connessione, specificare il suo nome nell'attributo ``connection`` del tag.

    L'attributo ``priority``, con valore ``0`` se omesso, può essere usato
    per controllare l'ordine in cui registrare gli ascoltatori. Come il
    :doc:`distributore di eventi</components/event_dispatcher/introduction>` di Symfony2,
    numeri maggiori daranno precedenza all'esecuzione dell'ascoltatore, mentre gli ascoltatori con
    uguale priorità saranno eseguiti nell'ordine in cui sono stati registrati con
    il gestore di eventi.

    Infine, l'attributo ``lazy``, con valore predefinito ``false``, può essere
    usato per richiedere che l'ascoltatore sia caricato pigramente dal gestore
    di eventi, quando distribuito.

    .. configuration-block::

        .. code-block:: yaml

            services:
                my_doctrine_listener:
                    class:   Acme\HelloBundle\Listener\MyDoctrineListener
                    # ...
                    tags:
                        -  { name: doctrine_mongodb.odm.event_listener, event: postPersist }

        .. code-block:: xml

            <service id="my_doctrine_listener" class="Acme\HelloBundle\Listener\MyDoctrineListener">
                <!-- ... -->
                <tag name="doctrine_mongodb.odm.event_listener" event="postPersist" />
            </service>.

        .. code-block:: php

            $definition = new Definition('Acme\HelloBundle\Listener\MyDoctrineListener');
            // ...
            $definition->addTag('doctrine_mongodb.odm.event_listener', array(
                'event' => 'postPersist',
            ));
            $container->setDefinition('my_doctrine_listener', $definition);

*   **sottoscrittore di eventi**: Usare il tag ``doctrine_mongodb.odm.event_subscriber``
    per registrare un sottoscrittore. I sottoscrittorii devono implementare
    ``Doctrine\Common\EventSubscriber`` e un metodo per restituire gli eventi
    che osservano. Per tale ragione, questo tag non ha un attributo ``event``.
    Sono invece disponibili gli attributi ``connection``, ``priority`` e 
    ``lazy``.

.. note::

    Diversamente dagli ascoltatori di eventi di Symfony2, il gestore di eventi di Doctrine si aspetta che ciascun
    ascoltatore e sottoscrittore abbia un nome di metodo corrispondente all'evento o agli eventi
    osservati. Per tale ragione, i tag menzionati in precedenza non hanno un attributo
    ``method``.

Integrazione con SecurityBundle
-------------------------------

Si può usare un fornitore di utente in un progetto MongoDB, che funziona esattamente come
il fornitore di entità descritto :doc:`nel ricettario</cookbook/security/entity_provider>`

.. configuration-block::

    .. code-block:: yaml

        security:
            providers:
                my_mongo_provider:
                    mongodb: {class: Acme\DemoBundle\Document\User, property: username}

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="my_mongo_provider">
                <mongodb class="Acme\DemoBundle\Document\User" property="username" />
            </provider>
        </config>

Riepilogo
---------

Con Doctrine, ci si può concentrare suglio oggetti e su come siano utili
nell'applicazione e preoccuparsi della persistenza su MongoDB in un secondo momento. Questo perché 
Doctrine consente di usare qualsiasi oggetto PHP per tenere dei dati e si appoggia
su metadati di mappatura per mappare i dati di un oggetto su una collezione di MongoDB.

Sebbene Doctrine giri intorno a un semplice concetto, è incredibilmente potente,
consentendo di creare query complesse e sottoscrivere eventi che consentono
di intraprendere diverse azioni, mentre gli oggetti viaggiano lungo il loro ciclo
di vita della persistenza.

Imparare di più dal ricettario
------------------------------

* :doc:`/bundles/DoctrineMongoDBBundle/form`

.. _`MongoDB`:          http://www.mongodb.org/
.. _`documentazione`:   http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/
.. _`documentazione sullo schema`: http://getcomposer.org/doc/04-schema.md#minimum-stability
.. _`Quick Start`:      http://www.mongodb.org/display/DOCS/Quickstart
.. _`Documentazione di base sulla mappatura`: http://www.doctrine-project.org/docs/mongodb_odm/1.0/en/reference/basic-mapping.html
.. _`tipo MongoDB`: http://php.net/manual/it/mongo.types.php
.. _`Documentazione sulla mappatura dei tipi`: http://docs.doctrine-project.org/projects/doctrine-mongodb-odm/en/latest/reference/basic-mapping.html#doctrine-mapping-types
.. _`Query Builder`: http://www.doctrine-project.org/docs/mongodb_odm/1.0/en/reference/query-builder-api.html
.. _`Operatori condizionali`: http://www.doctrine-project.org/docs/mongodb_odm/1.0/en/reference/query-builder-api.html#conditional-operators
.. _`Documentazione sugli eventi`: http://www.doctrine-project.org/docs/mongodb_odm/1.0/en/reference/events.html
