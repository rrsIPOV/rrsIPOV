---
layout: post
title: Scrolling with Grails and HQL
---

## Or: Using leaky abstractions.

I have a need to export CSV and other data formats from a 
small Grails app. For several of the services, I use the HibernateCriteriaBuilder and it's .scroll() method 
which works well. However on one of the other services I was dealing with 
some denormalized data where I needed a complex join on, I had implemented the query in 
HQL and it worked ok, but then after some changes I noticed an integration test 
failing. The issue seems to be with the Hiberate to MS SQL adapter when paging.

In the integration test I call a service method that runs a 
for-each loop over a data set, simulating export. I was reading 100 records at a 
time looping using `DomainClass.executeQuery(hql, paramsMap, [offset: off, max:100])`. It 
seems that because my select had a nested select, the alias on the nested 
select was working its way up to the constructed query instead of the alias 
for the correct column - since MS SQL (older version) doesn't support 
the LIMIT statement, the SQL that was run looked like:

```sql
WITH query AS 
	(
	SELECT inner_query.*, ROW_NUMBER() OVER (ORDER BY CURRENT_TIMESTAMP) as __hibernate_row_nr__  FROM 
	   (
		select 
		TOP(?) ... as col_0_0_,
		-- ... additional query stuff...
		  (select 
			 max(X.x) as page0_ 
			 from X 
			 where A AND B
		  ) as col_N_0_
		from 
		--- ...more stuff ...
		) inner_query 
	) SELECT col_0_0, ..., page0_ from query WHERE ...
```

So where the constructed outer query needed to 
use col_N_0_ it was instead referencing page0_. I suppose I could have 
poked around to see if there was some way to control this, 
but after a couple of minutes scanning through documents it seemed a bit 
like a dead end. If I'd had time I would have setup some tests and 
seen about submitting a but, I still might if I have a chance. However, 
I needed something that would work now.

One of the nice things about Grails (and GORM) is that it's built on top of some fairly powerful 
tools. This can also be a bit of a problem sometimes as the 
documentation occasionally just trails off. I'm sure it's not that the doc writers don't care, 
it's just that the solution is actually inside one of the other tools and 
they tend to assume in some cases that one has worked directly with 
those tools before coming to Grails, rather than having started with Grails. In this 
case I simply couldn't seem to find a good way to go from an HQL 
statement to a Scrollable result using Grails / GORM API... possibly because a lot of the Grails 
documents still link to Hibernate 3 api, even though the default is now Hibernate 5.

So, after working my way though some documentation, 
and neverously waiting while my test ran here's the result:

```groovy
/**
 * @param hql HQL statement to execute, including any named parameters.
 * @param params Map of key->value pairs to replace as named params within the HQL.
 * @param callback  A closure to receive each HQL result object.
 */
void forEach(String hql, Map params, Closure callback) 
{
	DomainClass.withSession { session ->
		
		Query q = session.createQuery( hql )
					.setReadOnly(true)
					.setCacheable(false);  
		
		params.each { k,v >
			if (v instanceof List) {    
				q.setParameterList(k,v)   
			} else {    
				q.setParameter(k,v);   
			}  
		}

		ScrollableResults scrollable = q.scroll(ScrollMode.FORWARD_ONLY);

		try { 
			while (scrollable.next()) { 
				c.call(scrollable.get(0)); 
			} 
		} finally { 
			scrollable.close(); 
		}
	}
}
```

Note: There has apparently been some work on annotating the Closure to assist w/ expected values, but 
it's rather ugly to implement, especially for something that is only an internal API.

