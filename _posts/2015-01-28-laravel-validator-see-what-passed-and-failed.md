---
title: 'Laravel Validator &#8211; See What Passed and Failed'
author: Tyler
layout: post
permalink: /laravel-validator-see-what-passed-and-failed/
categories:
  - Posts
---
The Laravel [Validator][1] as of 1/28/15 doesn&#8217;t tell you what passed and failed on a field-by-field basis. Here&#8217;s a quick hack to find out.

**Scenario**

I have a form with the fields: &#8216;name&#8217; and &#8216;domain&#8217;. The name is required but the domain is not. If something is entered for the domain, it must be validated according to some rule, e.g. WHOIS query. We don&#8217;t want this form to be an all-or-nothing deal. If a single field passes validation, our table should be updated. If a field does not pass validation, do nothing and spit out some [error messages][2].

{% highlight php startinline=true %}
// Some rules for the form
$rules = array(
    'name' =&gt; 'required|min:1|max:50', 
    'domain' =&gt; 'available'
);

// Add custom rule
Validator::extend('available', function($attribute, $value, $parameters)
{
    return true; // Could be any function, really
});

$validator = Validator::make($_POST, $rules);
{% endhighlight %}

After the validator does its thing, we pass the whole object to passedOrFailed which does what its name suggests.

{% highlight php startinline=true %}
public function passedOrFailed($validator)
{
	$results = [
		"passed" =&gt; [],
		"failed" =&gt; []
	];

	$data = $validator-&gt;getData();
	$fields = array_keys($data);

	foreach ($fields as $field) {
		if ($validator-&gt;messages()-&gt;get($field)) {
			$results["failed"][$field] = $data[$field];
		} else {
			$results["passed"][$field] = $data[$field];
		}
	}

	return $results;
}
{% endhighlight %}

You&#8217;ll get out a result like this:

{% highlight php startinline=true %}
array (size=2)
  'passed' =&gt; 
    array (size=2)
      'name' =&gt; string 'Bob' (length=3)
      'domain' =&gt; string 'bob.com' (length=7)
  'failed' =&gt; 
    array (size=0)
      empty
{% endhighlight %}

From there you can update your table or do whatever you want.

 [1]: http://laravel.com/docs/4.2/validation
 [2]: http://laravel.com/docs/4.2/validation#working-with-error-messages