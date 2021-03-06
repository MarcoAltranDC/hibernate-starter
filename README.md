[![Build Status](https://travis-ci.org/jooby-project/hibernate-starter.svg?branch=master)](https://travis-ci.org/jooby-project/hibernate-starter)
# hibernate

Starter project for [hibernate ORM](http://hibernate-orm.github.io/).

## quick preview

This project contains a simple application that:

* Uses a memory database
* Insert some database records on application startup time
* Expose data via `JSON` routes.
* [Query DSL](http://www.querydsl.com)

[App.java](https://github.com/jooby-project/hibernate-starter/blob/master/src/main/java/starter/hbm/App.java):

```java
/**
 * Hibernate ORM Starter project.
 */
public class App extends Jooby {

  {
    /** JSON: */
    use(new Jackson());

    /** Jdbc: */
    use(new Jdbc());

    /**
     * Hibernate:
     */
    use(new Hbm()
        .classes(Pet.class)
    );

    /**
     * Insert some data on startup:
     */
    onStart(() -> {
      EntityManagerFactory factory = require(EntityManagerFactory.class);

      EntityManager em = factory.createEntityManager();

      EntityTransaction trx = em.getTransaction();
      trx.begin();
      em.persist(new Pet("Lala"));
      em.persist(new Pet("Mandy"));
      em.persist(new Pet("Fufy"));
      em.persist(new Pet("Dina"));
      trx.commit();

      em.close();
    });

    /** Open session in view filter: */
    use("*", Hbm.openSessionInView());

    /**
     * Find all via query-dsl:
     */
    get("/pets", req -> {
      EntityManager em = require(EntityManager.class);
      List<Pet> pets = new JPAQuery<>(em)
          .select(pet)
          .from(pet)
          .fetch();
      return pets;
    });

    /**
     * Find by id via entity manager:
     */
    get("/pets/{id:\\d+}", req -> {
      int id = req.param("id").intValue();
      EntityManager em = require(EntityManager.class);
      return em.find(Pet.class, id);
    });

    /**
     * Find by name via query-dsl:
     */
    get("/pets/:name", req -> {
      EntityManager em = require(EntityManager.class);
      String name = req.param("name").value();
      List<Pet> pets = new JPAQuery<>(em)
          .select(pet)
          .from(pet)
          .where(pet.name.likeIgnoreCase(name))
          .fetch();
      return pets;
    });
  }

  public static void main(final String[] args) {
    run(App::new, args);
  }

}
```

## run

    mvn jooby:run

## help

* Read the [module documentation](http://jooby.org/doc/hbm)
* Join the [channel](https://gitter.im/jooby-project/jooby)
* Join the [group](https://groups.google.com/forum/#!forum/jooby-project)
