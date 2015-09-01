---
layout: post
title:  "Isis Script DSL Description"
date:   2015-09-01 18:15:00
author: Torsten Juergeleit
tags: isis dsl xtext
comments: true
---
A formal [description of the Isis Script DSL](https://github.com/vaulttec/isis-script/blob/develop/dsl.md) is available on GitHub now. It contains all the details found in [ISIS-369](https://issues.apache.org/jira/browse/ISIS-369) and the [Apache Isis mailing list](http://mail-archives.apache.org/mod_mbox/incubator-isis-dev/201210.mbox/%3cCALJOYLGGptSXtW2PgZxgiK09_ZfedXmpZEk+Bv7=_AhHRG81Mw@mail.gmail.com%3e).

In comparison to the formerly published [alpha version of the DSL]({% post_url 2015-07-06-isis-script-first-alpha-version %}) this version provides the complete feature set of Apache Isis [Entities](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#entities) and [Services](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#services):

 * The simplistic actions are replaced by [full-blown actions](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#actions) with parameters, rules and events
 * [Collections](https://github.com/vaulttec/isis-script/blob/develop/dsl.md#collections) are defined now
 * Separate events for properties, collections and actions

The concept of a dedicated service for entity repositories was dropped due to the increased complexity of the resulting DSL grammar while providing only small benefits (e.g. service name derived from entity).

So any further implementation of [Isis Script](https://github.com/vaulttec/isis-script) will follow [this DSL description](https://github.com/vaulttec/isis-script/blob/develop/dsl.md).
