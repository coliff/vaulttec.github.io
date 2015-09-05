---
layout: post
title:  "Isis Script - Updated (alpha) version available"
date:   2015-09-05 20:15:00
author: Torsten Juergeleit
tags: isis dsl eclipse xtext
comments: true
---
As mentioned in my [previous post]({% post_url 2015-09-01-isis-script-dsl-description %}) a formal [description of the Isis Script DSL](https://github.com/vaulttec/isis-script/blob/develop/dsl.md) can be found on GitHub. An updated implementation of the [Isis Script Eclipse editor](https://github.com/vaulttec/isis-script#the-eclipse-dsl-editor) is available as well.

![Isis Script DSL Editor](/images/2015-09-05-isis-script-updated-alpha-version/simpleobject-dsl-editor.png)

Now the following features are support:

 * [Entities](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#entities) with injections, properties, collections, action, events and UI hints
 * [Services](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#services) with injections, actions and events

This feature set allows us to implement the entity and the repository service of the domain object `SimpleObject` from the [Apache Isis Maven archetype SimpleApp](http://isis.apache.org/guides/ug.html#_ug_getting-started_simpleapp-archetype). The corresponding Isis Scripts (based on the new version of the DSL) are as follows:

{% highlight java %}
package domainapp.dom.modules.simple

import javax.jdo.annotations.Column
import javax.jdo.annotations.DatastoreIdentity
import javax.jdo.annotations.IdGeneratorStrategy
import javax.jdo.annotations.IdentityType
import javax.jdo.annotations.PersistenceCapable
import javax.jdo.annotations.Queries
import javax.jdo.annotations.Query
import javax.jdo.annotations.Unique
import javax.jdo.annotations.Version
import javax.jdo.annotations.VersionStrategy
import org.apache.isis.applib.annotation.Action
import org.apache.isis.applib.annotation.BookmarkPolicy
import org.apache.isis.applib.annotation.DomainObject
import org.apache.isis.applib.annotation.DomainObjectLayout
import org.apache.isis.applib.annotation.Editing
import org.apache.isis.applib.annotation.Parameter
import org.apache.isis.applib.annotation.ParameterLayout
import org.apache.isis.applib.annotation.Property
import org.apache.isis.applib.annotation.Title
import org.apache.isis.applib.services.i18n.TranslatableString

@PersistenceCapable(identityType=IdentityType.DATASTORE)
@DatastoreIdentity(strategy=IdGeneratorStrategy.IDENTITY, column="id")
@Version(strategy=VersionStrategy.VERSION_NUMBER, column="version")
@Queries(#[
	@Query(name = "find", language = "JDOQL",
		value = "SELECT FROM domainapp.dom.modules.simple.SimpleObject"),
	@Query(name = "findByName", language = "JDOQL",
		value = "SELECT FROM domainapp.dom.modules.simple.SimpleObject WHERE name.indexOf(:name) >= 0")
])
@Unique(name="SimpleObject_name_UNQ", members = #["name"])
@DomainObject(objectType = "SIMPLE")
@DomainObjectLayout(bookmarking = BookmarkPolicy.AS_ROOT)
entity SimpleObject {

	@Column(allowsNull="false", length = 40)
	@Title(sequence="1")
	@Property(editing = Editing.DISABLED)
	property String name

	@Action(domainEvent = UpdateNameDomainEvent)
	action SimpleObject updateName {
		@Parameter(maxLength = 40)
		@ParameterLayout(named = "New name")
		parameter String newName {
			default {
				getName
			}
		}
		body {
			setName(newName)
			this
		}
		validate {
			if (newName.contains("!"))
				TranslatableString.tr("Exclamation mark is not allowed")
			else null
		}
		event UpdateNameDomainEvent
	}

	title {
		TranslatableString.tr("Object: {name}", "name", name)
	}
}
{% endhighlight %}

{% highlight java %}
package domainapp.dom.modules.simple

import org.apache.isis.applib.annotation.DomainServiceLayout
import org.apache.isis.applib.annotation.DomainService
import org.apache.isis.applib.annotation.Action
import org.apache.isis.applib.annotation.SemanticsOf
import org.apache.isis.applib.annotation.BookmarkPolicy
import org.apache.isis.applib.annotation.ActionLayout
import org.apache.isis.applib.annotation.MemberOrder
import org.apache.isis.applib.annotation.ParameterLayout
import org.apache.isis.applib.query.QueryDefault
import java.util.List

@DomainService(repositoryFor = SimpleObject)
@DomainServiceLayout(menuOrder = "10")
service SimpleObjects {

	@Action(semantics = SemanticsOf.SAFE)
	@ActionLayout(bookmarking = BookmarkPolicy.AS_ROOT)
	@MemberOrder(sequence = "1")
	action List<SimpleObject> listAll {
		body {
			container.allInstances(SimpleObject)
		}
	}

	@Action(semantics = SemanticsOf.SAFE)
	@ActionLayout(bookmarking = BookmarkPolicy.AS_ROOT)
	@MemberOrder(sequence = "2")
	action List<SimpleObject> findByName {
		@ParameterLayout(named="Name") 
		parameter String name
		body {
			container.allMatches(new QueryDefault(SimpleObject, "findByName", "name", name))
		}
	}

	@MemberOrder(sequence = "3")
	action SimpleObject create {
		@ParameterLayout(named="Name")
		parameter String name
		body {
			val obj = container.newTransientInstance(SimpleObject)
			obj.name = name
			container.persistIfNotAlready(obj)
			obj
		}
	}
}
{% endhighlight %}

If you're interested in giving the updated version of the [Eclipse editor for Isis Script](https://github.com/vaulttec/isis-script#the-eclipse-dsl-editor) a try then you can find the corresponding information [here](https://github.com/vaulttec/isis-script#installation).

How to use Isis Script in a Maven build is described in [this post]({% post_url 2015-07-22-isis-script-and-maven %}).

