
+++
date = "2012-06-17 14:55:00"
draft = false
title = "Building an API Part #1"
slug = "building-an-api-part-1"
tags = []
banner = ""
aliases = ['/building-an-api-part-1/']
+++

<p><em>This blog post is a part of &ldquo;the things I learned while working on popcorn&rdquo;, Popcorn tracks usage of linux packages. <a href="http://mapleoin.eu/" target="_blank">Ionuț Arțăriși</a></em><em> is my mentor for this <a href="http://www.google-melange.com/gsoc/proposal/review/google/gsoc2012/axitkhurana/3002" title="GSoC Proposal" target="_blank">GSoC project</a> for openSUSE.<br/></em></p>
<p>PART #1: Views return JSON in addition to HTML</p>
<p><strong><a href="http://www.json.org/" title="JSON" target="_blank">JSON</a>?</strong></p>
<p>JSON = Javascript Object Notation, it&rsquo;s a data interchange format. People used to use XML for data interchange, but JSON is even simpler than XML and it maps more directly onto the data structures used in modern programming languages</p>
<p>It is built on two structures:</p>
<ul><li>Dictionary (Name/Value pairs)</li>
<li>List (Ordered list of values)</li>
</ul><p><strong>Returning JSON and HTML:</strong></p>
<p>1. The render decorator:</p>
<p>Instead of cluttering up the views, Ionuț advised me to write a decorator that renders HTML or JSON depending on the request&rsquo;s header accept mimetypes. To do that, views needed to return a dictionary.</p>
<p>The <a href="https://github.com/axitkhurana/popcorn/commit/2c5cb34a83205233ec65a193f650db7ff512323b#L0R13" title="Github" target="_blank">decorator</a> would act on the dictionary and depending on the mimetype, render the template, (say a browser sent a request) or jsonify the dictionary and returns it (someone is trying to use the API).</p>
<p>2. SQLalchemy objects are not JSON serializable.</p>
<p>You can&rsquo;t just do flask.jsonify(system=System.query.all()) (System is a model). Searching a little, I realized I had to write a serialize function for the model. But there was a problem.</p>
<p>The system model looks like this:</p>
<pre><span class="lnr">34 </span>    sys_hwuuid = Column(String(<span class="Constant">32</span>), primary_key=<span class="Identifier">True</span>)
<span class="lnr">35 </span>    distro_name = Column(String(<span class="Constant">20</span>), nullable=<span class="Identifier">False</span>)
<span class="lnr">36 </span>    distro_version = Column(String(<span class="Constant">10</span>), nullable=<span class="Identifier">False</span>)
<span class="lnr">37 </span>    arch = Column(String(<span class="Constant">10</span>), ForeignKey(<span class="Constant">'arches.arch'</span>), nullable=<span class="Identifier">False</span>)
<span class="lnr">38 </span>
<span class="lnr">39 </span>    submissions = relationship(<span class="Constant">'Submission'</span>, backref=<span class="Constant">'system'</span>,
<span class="lnr">40 </span>                               order_by=Submission.sub_date.desc())
</pre>
<p>System model has a relationship with Submission, which in turn has a relationship with Submission_Packages and so on. So, if we are serializing System, to what depth do we go?</p>
<p>There are a few suggestions Ionuț gave me:</p>
<blockquote>
<p>One answer is to provide links to external resources. Then the user would get the static data of the System resource and a list of links for each of the Submissions. The user can then choose if he wants to know more about those Submissions by issuing more requests.</p>
<p>Also check out <a href="http://stateless.co/hal_specification.html" target="_blank">http://stateless.co/hal_specification.html</a> for a format standard built on top of json, which adds hypermedia. There are other proposed standards, too.</p>
</blockquote>
<p>I&rsquo;ll include the discussion on Hypermedia, HAL specification and more in part #2. For now, we decided to make two methods, one that returns only the object&rsquo;s attributes, accessible without relationships and the other, which returns the attributes + the first layer of relationship&rsquo;s attributes. So, the serialize function would go one level.</p>
<p>3. Python datetime objects are not handled by flask.jsonify:</p>
<p>Initially I wrote a function to dump the datetime and take care if its none, Ionuț pointed out, we don&rsquo;t need time and the date can&rsquo;t be None as it is the primary key.</p>
<p>So, a simple value.strftime(&ldquo;%Y-%m-%d&rdquo;) worked.</p>
<p>4. Modifying views to apply decorators:</p>
<p>Removed the render_template function by dicts of parameters, and handling them by a render decorator written in 1.</p>
<p>5. Something that is untested is broken.</p>
<p>Pull request for the above changes: <a href="https://github.com/mapleoin/popcorn/pull/10" target="_blank">https://github.com/mapleoin/popcorn/pull/10</a></p>

