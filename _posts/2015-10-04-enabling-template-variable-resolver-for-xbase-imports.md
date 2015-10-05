---
layout: post
title:  "Enabling TemplateVariableResolver for Xbase imports in Xtext editor"
date:   2015-10-04 18:23:00
author: Torsten Juergeleit
tags: eclipse xtext xbase
comments: true
---
While implementing [code templates](https://eclipse.org/Xtext/documentation/304_ide_concepts.html#templates) for the [Isis Script DSL editor](https://github.com/vaulttec/isis-script#the-eclipse-dsl-editor) I struggled with the template variable resolver for Xbase Imports.

![Isis Script Templates](/images/2015-10-04-enabling-template-variable-resolver-for-xbase-imports/isis-script-templates.png)

Despite the corresponding `TemplateVariableResolver` implementation [`ImportsVariableResolver`](https://github.com/eclipse/xtext/blob/e9f0e284c42cda97ea57086c31ba2f5160049b26/plugins/org.eclipse.xtext.xbase.ui/src/org/eclipse/xtext/xbase/ui/templates/ImportsVariableResolver.java) is active by default it's not working as expected - no imports are added.

Debugging the implementation of the `resolveVariables()` method reveals that it doesn't get the expected `TemplateContext` implementation [`XbaseTemplateContext`](https://github.com/eclipse/xtext/blob/e9f0e284c42cda97ea57086c31ba2f5160049b26/plugins/org.eclipse.xtext.xbase.ui/src/org/eclipse/xtext/xbase/ui/templates/XbaseTemplateContext.java) which provides access to the DSLs import section:

{% highlight java %}
public List<String> resolveValues(TemplateVariable variable, XtextTemplateContext xtextTemplateContext) {
	variable.setUnambiguous(true);
	variable.setValue(""); //$NON-NLS-1$
	if (xtextTemplateContext instanceof XbaseTemplateContext) {
		XbaseTemplateContext xbaseCtx = (XbaseTemplateContext) xtextTemplateContext;
		List<?> params = variable.getVariableType().getParams();
		if (params.size() > 0) {
			for (Iterator<?> iterator = params.iterator(); iterator.hasNext();) {
				String typeName = (String) iterator.next();
				xbaseCtx.addImport(typeName);
			}
		}
	}
	return new ArrayList<String>();
}
{% endhighlight %}

Xtexts content assist retrives this `TemplateContext` implementation from the  [`XbaseTemplateProposalProvider`](https://github.com/eclipse/xtext/blob/e9f0e284c42cda97ea57086c31ba2f5160049b26/plugins/org.eclipse.xtext.xbase.ui/src/org/eclipse/xtext/xbase/ui/templates/XbaseTemplateProposalProvider.java) - an extension of Xtexts `DefaultTemplateProposalProvider`.

To use `XbaseTemplateProposalProvider` instead of `DefaultTemplateProposalProvider` we have to override the corresponding Guice binding for `ITemplateProposalProvider`: 

{% highlight java %}
public Class<? extends ITemplateProposalProvider> bindITemplateProposalProvider() {
	return DefaultTemplateProposalProvider.class;
}
{% endhighlight %}

Unluckily `XbaseTemplateProposalProvider` can't be used here due to the lack of an `@Inject` annotation for its constructor (Xtext version 2.8.4). So we have to create [our own subclass](https://github.com/vaulttec/isis-script/blob/develop/isis-script-eclipse/org.vaulttec.isis.script.ui/src/org/vaulttec/isis/script/ui/contentassist/IsisTemplateProposalProvider.java) providing an annotated constructor

{% highlight java %}
@Singleton
public class IsisTemplateProposalProvider extends XbaseTemplateProposalProvider {

	@Inject
	public IsisTemplateProposalProvider(TemplateStore templateStore, ContextTypeRegistry registry,
			ContextTypeIdHelper helper) {
		super(templateStore, registry, helper);
	}

}
{% endhighlight %}

and use this in our Xtext editors [UI Guice module](https://github.com/vaulttec/isis-script/blob/develop/isis-script-eclipse/org.vaulttec.isis.script.ui/src/org/vaulttec/isis/script/ui/IsisUiModule.java)

{% highlight java %}
public class IsisUiModule extends AbstractIsisUiModule {

	public IsisUiModule(AbstractUIPlugin plugin) {
		super(plugin);
	}

	@Override
	public Class<? extends ITemplateProposalProvider> bindITemplateProposalProvider() {
		return IsisTemplateProposalProvider.class;
	}
}
{% endhighlight %}

Providing our own subclass of `XbaseTemplateProposalProvider` allows us to customize the template proposals of our Xtext editor. Here we can override `getImage()` to show a different image for the proposals or override `getRelevance()` for changing the order of the proposals.