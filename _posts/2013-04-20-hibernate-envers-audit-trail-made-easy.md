---
layout: post
title:  "Hibernate Envers - Audit Logging Made Easy"
date:   2013-04-20 22:26:00 -0700
summary: "Need to keep a detailed audit trail for your database entities? Hibernate Envers is the way to go!"
---
One of the requirements for my most recent project was to provide a detailed audit trail for any changes made to my database entities through the administration panel. I needed to keep track of what was changed (at a field level) and who made the change. There are a ton of ways to do this, including triggers, plain old system logging, and homebrew audit tables. But, I decided to look around for something cooler than that. Enter Hibernate Envers. What's nice about it is configuration is basically a one-liner (for the most part), and it plays nice with my existing all-annotations JPA2 Spring MVC project. All you need to do is mark your entities with the @Audited annotation, and it takes care of the rest for you. Here's an example:

### Maven dependency (pom.xml):

{% highlight xml %}
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-envers</artifactId>
  <version>4.1.9.Final</version>
</dependency>
{% endhighlight %}

### Entity:

{% highlight java %}
@Audited
@Entity
@Table(name = "PERSON")
public class PersonEntity implements Serializable
{

  @Id
  @GeneratedValue( strategy = GenerationType.AUTO )
  @Column( name = "ID" )
  private Integer id;

  @Column( name= "FIRST_NAME")
  private String firstName;

  @Column( name= "LAST_NAME")
  private String lastName;

}
{% endhighlight %}

And, that's it! So what does this do? Let's look at the tables that Hibernate will generate:

{% highlight sql %}
create table PERSON_AUD (
    ID number(10,0) not null,
    REV number(10,0) not null,
    REVTYPE number(3,0),
    FIRST_NAME varchar2(255 char),
    LAST_NAME varchar2(255 char),
    primary key (ID, REV)
)

create table REVINFO (
    REV number(10,0) not null,
    REVTSTMP number(19,0),
    primary key (REV)
)

alter table PERSON_AUD
    add constraint FK_PERSON_AUD -- I renamed the system generated name... ;)
    foreign key (REV)
    references REVINFO
{% endhighlight %}

Basically, with the default configuration, you end up with another table with the same name as your existing table but with "_AUD" appended on the end. You can also change the generated name of the tables if you want. It has the same fields as your entity table but adds a REV number to keep track of the revision, and a REVTYPE to indicate an addition, modification, or delete. Read more about this in the [documentation](http://docs.jboss.org/hibernate/envers/3.6/reference/en-US/html_single/#tables). Each time any field is changed on one of your entities an entry will be written to this table saving the state, which allows you to easily see what changed between 2 revisions. Envers also includes utilities to query over these tables to find all the revisions for a certain entity.

### Generating Schema DDL

By the way, if you are looking for an easy way to generate the DDL for your entities *AND* your envers tables, here's some code. I had trouble finding a Maven plugin that supported BOTH Hibernate 4 and Envers, so this is what I did:

{% highlight java %}
Configuration config = new Configuration();

// don't forget to get the right dialect for Oracle, MySQL, etc
config.setProperty("hibernate.dialect","org.hibernate.dialect.Oracle10gDialect");
config.addAnnotatedClass(PersonEntity.class);

SchemaExport export = new EnversSchemaGenerator(config).export().setOutputFile("person-schema.sql");
export.execute(true, false, false, false);
{% endhighlight %}

### Summary

So yeah, Hibernate Envers is an awesome time-saver if you need an audit trail. Keep in mind that these tables can grow quickly in size if you have a lot of changes to your entities happening regularly. But otherwise, I'm really glad I found this since it was just what I needed. Enjoy!
