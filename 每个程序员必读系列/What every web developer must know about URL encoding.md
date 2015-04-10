###What every web developer must know about URL encoding
This article describes common misconceptions about Uniform Resource Locator (URL) encoding, then attempts to clarify URL encoding for HTTP, before presenting frequent problems and their solutions. While this article is not specific to any programming language, we illustrate the problems in Java) and finish by explaining how to fix URL encoding problems in Java, and in a web application at several levels.

####Introduction

There are a number of technologies we use everyday when we browse the web. There is the data itself (the web pages) obviously, the formatting of this data, the transport mechanism which allows us to retrieve this data, and there is the foundation, the root, the thing that makes the web a web: links from one page to the other. These links are URLs.

#####General URL syntax

Everyone by now has seen a URL at least once in his life. Take "http://www.google.com" for instance. This is a URL. A URL is a Uniform Resource Locator and is really a pointer to a web page (in most cases). URLs actually have a very well-defined structure since the first specification in 1994.

We can extract detailed information about the "http://www.google.com" URL:

<pre>
Part	Data
Scheme	http
Host address	www.google.com
</pre>

If we look at a more complex URL such as "https://bob:bobby@www.lunatech.com:8080/file;p=1?q=2#third" we can extract the following information:

<pre>
Part	Data
Scheme	https
User	bob
Password	bobby
Host address	www.lunatech.com
Port	8080
Path	/file
Path parameters	p=1
Query parameters	q=2
Fragment	third
</pre>
The Scheme (here http and https (secure HTTP)) define the structure of the rest of the URL. Most internet URL schemes have a common first part which indicates the user, password, host name and port, followed by a scheme-specific part. This common part deals with authentication and being able to know where to connect in order to request data.

#####HTTP URL syntax
For HTTP URLs (with the http or https schemes), the scheme-specific part of the URL defines the path to the data, followed by an optional query and fragment.

The path part consists in a hierarchical view similar to a file system hierarchy with folders and files. The path starts with a "/" character, then each folder is separated from one another by a "/" again until we reach the file. For example "/photos/egypt/cairo/first.jpg" has four path segments: "photos", "egypt", "cairo" and "first.jpg", which can be extrapolated as: the "first.jpg" file in the "cairo" folder, which is in the "egypt" folder located in the "photos" folder at the root of the web site.

Each path segment can have optional path parameters (also called matrix parameters) which are located at the end of the path segment after a ";", and separated by ";" characters. Each parameter name is separated from its value by the "=" character like this: "/file;p=1" which defines that the path segment "file" has a path parameter "p" with the value "1". These parameters are not often used — let's face it — but they exist nonetheless, and we've even found a very good justification for their use on a Yahoo RESTful API documentation:

Matrix parameters enable the application to retrieve part of a collection when calling an HTTP GET operation. See Paging a Collection for an example. Because matrix parameters can follow any collection path segment in a URI, they can be specified on an inner path segment.

After the path segments we can find the query which is separated from the path with a "?" character, and contains a list (separated by "&") of parameter names and values separated by "=". For example "/file?q=2" defines a query parameter "q" with the value "2". This is used a lot when submitting HTML forms, or when calling applications such as Google search.

Last in an HTTP URL is the fragment which is used to refer not to the whole HTML page but to a specific part within that file. When you click on a link and the browser automatically scrolls down to display a part which was not visible from the top of the page, you have clicked a URL with a fragment part.

#####URL grammar

The http URL scheme was first defined in RFC 1738 (actually even before in RFC 1630) and while the http URL scheme has not been redefined later the whole URL syntax has been generalised into Uniform Resource Identifiers (URIs) from a specification which has been extended a few times to accommodate for evolutions.

There is a grammar which defines how URLs are assembled, and how parts are separated. For instance, the "://" part separates the scheme from the host part. The host and path fragments parts are separated by "/", while the query part follows a "?". This means that certain characters are reserved for the syntax. Some are reserved for all URIs, while some are only reserved for specific schemes. All reserved characters that are used in a part where they are not allowed (for instance a path segment — a file name for example — which would contain a "?" character) must be URL-encoded.

URL-encoding is the transformation of a character ("?") into a harmless representation of this character which has no syntactic meaning in the URL. This is done by converting the character into a sequence of bytes in a specific character encoding, then writing these bytes in hexadecimal preceded by "%". A question mark in URL-encoding is therefore "%3F".

We can write a URL pointing to the "to_be_or_not_to_be?.jpg" image as such: "http://example.com/to_be_or_not_to_be%3F.jpg" which makes sure that nobody would think there might be a query part in there.

Most browsers nowadays display the URLs by decoding (converting percent-encoded bytes back to their original characters) them first, while keeping them encoded when fetching them for the network. This means users are almost never aware of such encoding.

Developers, or web page authors on the other hand have to be aware of it, because there are many pitfalls.

####Common pitfalls of URLs

If you are working with URLs, it pays to know some of the most common traps you should avoid. Here we give a non-exhaustive list of some of those traps.

#####Which character encoding?

URL-encoding does not define any particular character encoding for percent-encoded bytes. Generally ASCII alphanumeric characters are allowed unescaped, but for reserved characters and those that do not exist in ASCII (the French "œ" from the word "nœud" — "knot" — for instance), we have to wonder which encoding to use when converting them to percent-encoded bytes.

Of course the world would be easier if there were just Unicode, because every character exists in this set, but this is a set — a list if you will — and not an encoding per se. Unicode can be encoded using several encodings such as UTF-8 or UTF-16 (there are several others), but then the problem is still there: which encoding should URLs (generally URIs) use?

The standards do not define any way by which a URI might specify the encoding it uses, so it has to be deduced from the surrounding information. For HTTP URLs it can be the HTML page encoding, or HTTP headers. This is often confusing and a source of many errors. In fact, the latest version of the URI standard defines that new URI schemes use UTF-8, and that host names (even on existing schemes) also use this encoding, which really rouses my suspicion: can the host name and the path parts really use different encodings?

#####The reserved characters are different for each part

Yes they are. Yes they are. Yes they are. (Yes they are).

For HTTP URLs, a space in a path fragment part has to be encoded to "%20" (not, absolutely not "+"), while the "+" character in the path fragment part can be left unencoded.

Now in the query part, spaces may be encoded to either "+" (for backwards compatibility: do not try to search for it in the URI standard) or "%20" while the "+" character (as a result of this ambiguity) has to be escaped to "%2B".

This means that the "blue+light blue" string has to be encoded differently in the path and query parts: "http://example.com/blue+light%20blue?blue%2Blight+blue". From there you can deduce that encoding a fully constructed URL is impossible without a syntactical awareness of the URL structure.

Suppose the following Java code to construct a URL:

    String str = "blue+light blue";
	String url = "http://example.com/" + str + "?" + str;
Encoding the URL is not a simple iteration of characters in order to escape those that fall outside of the reserved set: we have to know which reserved set is active for each part we want to encode.

This means that most URL-rewriting filters would be wrong if they decide to take a URL substring from one part into another without proper encoding care. It is impossible to encode a URL without knowing about its specific parts.

#####The reserved characters are not what you think they are

Most people ignore that "+" is allowed in a path part and that it designated the plus character and not a space. There are other surprises:

- "?" is allowed unescaped anywhere within a query part,
- "/" is allowed unescaped anywhere within a query part,
- "=" is allowed unescaped anywhere within a path parameter or query parameter value, and within a path segment,
- ":@-._~!$&'()*+,;=" are allowed unescaped anywhere within a path segment part,
- "/?:@-._~!$&'()*+,;=" are allowed unescaped anywhere within a fragment part.

While this is slightly nuts and 

    "http://example.com/:@-._~!$&'()*+,=;:@-._~!$&'()*+,=:@-._~!$&'()*+,==?/?:@-._~!$'()*+,;=/?:@-._~!$'()*+,;==#/?:@-._~!$&'()*

is a valid HTTP URL, this is the standard.

For the curious, the previous URL expands to:

<pre>
Part					Value
Scheme					http
Host					example.com
Path					/:@-._~!$&'()*+,=
Path parameter name		:@-._~!$&'()*+,
Path parameter value	:@-._~!$&'()*+,==
Query parameter name	/?:@-._~!$'()* ,;
Query parameter value	/?:@-._~!$'()* ,;==
Fragment					/?:@-._~!$&'()*+,;=
Nuts.
</pre>

#####A URL cannot be analysed after decoding

The syntax of the URL is only meaningful before it is URL-decoded: after URL-decoding, reserved characters may appear.

For example "http://example.com/blue%2Fred%3Fand+green" has the following parts before decoding:

<pre>
Part					Value
Scheme					http
Host					example.com
Path segment			blue%2Fred%3Fand+green
Decoded Path segment	blue/red?and+green
</pre>

Thus, we are looking for a file called "blue/red?and+green", not for the "red?and+green" file of the "blue" folder.

If we decode it to "http://example.com/blue/red?and+green" before analysis the parts would give:

<pre>

Part				Value
Scheme				http
Host				example.com
Path segment		blue
Path segment		red
Query parameter 	name and green
</pre>

This is clearly wrong: analysis of reserved characters and URL parts has to be done before URL-decoding. The implication is that URL-rewriting filters should never decode a URL before attempting to match it iff reserved characters are allowed to be URL-encoded (which may or may not be the case depending on you application).

#####Decoded URLs cannot be reencoded to the same form

If you decode "http://example.com/blue%2Fred%3Fand+green" to "http://example.com/blue/red?and+green" and proceed to encode it (even with an encoder which knows about each syntactical URL part) you will get "http://example.com/blue/red?and+green" because that is a valid URL. It just happens to be very different from the original URL we decoded.

####Handling URLs correctly in Java

When you have mastered your black belt in URL-fu you will notice that there are still quite a few Java-specific pitfalls when it comes to URLs. The road to URL handling correctness is not for the faint of heart.

Do not use java.net.URLEncoder or java.net.URLDecoder for whole URLs
We are not kidding. These classes are not made to encode or decode URLs, as their API documentation clearly says:

> Utility class for HTML form encoding. This class contains static methods for converting a String to the application/x-www-form-urlencoded MIME format. For more information about HTML form encoding, consult the HTML specification.

This is not about URLs. At best it resembles the query part encoding. It is wrong to use it to encode or decode entire URLs. You would think the standard JDK had a standard class to deal with URL encoding properly (part by part, that is) but either it is not there, or we have not found it, which lures a lot of people into using URLEncoder for the wrong purpose.

#####Do not construct URLs without encoding each part

As we have already stated: fully constructed URLs cannot be URL-encoded.

Take the following code for instance:

    String pathSegment = "a/b?c";
	String url = "http://example.com/" + pathSegment;

It is impossible to convert "http://example.com/a/b?c" back to what it should have been if "a/b?c" was meant to be a path segment, because it happens to be a valid URL. We have already explained this earlier.

Here is the proper code:

    String pathSegment = "a/b?c";
	String url = "http://example.com/"
            + URLUtils.encodePathSegment(pathSegment);

We are now using a utility class URLUtils which we had to make ourselves for lack of finding an exhaustive one available online fast enough. The previous code will give you the properly encoded URL "http://example.com/a%2Fb%3Fc".

Note that the same applies to the query string:

    String value = "a&b==c";
	String url = "http://example.com/?query=" + value;


This will give you "http://example.com/?query=a&b==c" which is a valid URL, but not the "http://example.com/?query=a%26b==c" we wanted.

#####Do not expect URI.getPath() to give you structured data

Since once a URL has been decoded, syntactical information is lost, the following code is wrong:

    URI uri = new URI("http://example.com/a%2Fb%3Fc");
	for(String pathSegment : uri.getPath().split("/"))
	  System.err.println(pathSegment);
It would first decode the path "a%2Fb%3Fc" into "a/b?c", then split it where it should not have been split into path segment parts.

The correct code of course uses the undecoded path:

    URI uri = new URI("http://example.com/a%2Fb%3Fc");

	for(String pathSegment : uri.getRawPath().split("/"))
	  System.err.println(URLUtils.decodePathSegment(pathSegment));

Do note that path parameters will still be present: deal with them if required.

#####Do not expect Apache Commons HTTPClient's URI class to get this right

The Apache Commons HTTPClient 3's URI class uses Apache Commons Codec's URLCodec for URL-encoding, which is wrong as their API documentation mentions since it is just as wrong as using java.net.URLEncoder. Not only does it use the wrong encoder, but it also decodes each part as if they all had the same reserved set.

####Fixing URL encoding at every level in a web application

We have had to fix quite a few URL-encoding issues in our application lately. From support in Java, down to the lower level of URL rewriting. We will here list a few of changes which were required.

#####Always encode URLs as you build them

In our HTML files, we replaced all occurrences of this:

    var url = "#{vl:encodeURL(contextPath + '/view/' + resource.name)}";

with:

    var url = "#{contextPath}/view/#{vl:encodeURLPathSegment(resource.name)}";

And similarly for query parameters.

#####Make sure your URL-rewrite filters deal with URLs correctly

Url Rewrite Filter is a URL rewriting filter we use in Seam to transform pretty URLs into application-dependent URLs.

For example, we use it to rewrite http://beta.visiblelogistics.com/view/resource/FOO/bar into http://beta.visiblelogistics.com/resources/details.seam?owner=FOO&name=bar. Obviously this involves taking some strings from one URL part to another, which means we have to decode from the path segment part and reencode as a query value part.

Our initial rule looked as follows:

    <urlrewrite decode-using="utf-8">
		<rule>
		  <from>^/view/resource/(.*)/(.*)$</from>
		  <to encode="false">/resources/details.seam?owner=$1&name=$2</to>
		 </rule>
	</urlrewrite>

It turns out that there are only two ways to deal with URL-decoding in Url Rewrite Filter: either every URL is decoded prior to doing the rule matching (the <to> patterns), or it is disabled and each rule has to deal with decoding. In our opinion the latter is the sanest option, especially if you move URL parts around, and/or want to match path segments which may contain URL-encoded path separators.

Within the replacement pattern (the <to> patterns) you can then deal with URL encoding/decoding using the inline functions escape(String) and unescape(String).

As of this writing, Url Rewrite Filter Beta 3.2 contains several bugs and limitations which blocked our progress towards URL-correctness:

- URL decoding was done using java.net.URLDecoder (which is wrong),
- the escape(String) and unescape(String) inline functions used java.net.URLDecoder and java.net.URLEncoder (which is not specific enough and will only work for entire query strings, beading in mind any "&" or "=" will not be encoded).

We therefore made a big patch fixing a few issues like URL decoding, and adding the inline functions escapePathSegment(String) and unescapePathSegment(String).

We can now write the almost correct:

    <urlrewrite decode-using="null">
		<rule>
		  <from>^/view/resource/(.*)/(.*)$</from>
		  <-- Line breaks inserted for readability -->
		  <to encode="false">/resources/details.seam
		                     ?owner=${escape:${unescapePath:$1}}
		                     &name=${escape:${unescapePath:$2}}</to>
		 </rule>
	</urlrewrite>

It is only almost correct because our patch still lacks a few things:

- the inline escaping/unescaping functions should be able to specify the encoding as either fixed (this is already done) or by determining it from the HTTP call (not supported yet),
- the old escape(String) and unescape(String) inline functions were left intact and still call java.net.URLDecoder which is wrong as it will not escape "&" or "=",
- we need to add more part-specific encoding/decoding functions,
- we need to add a way to specify the decoding behaviour per-rule as opposed to globally in <urlrewrite>.

As soon as we get the time, we will send a second patch.

#####Using Apache mod-rewrite correctly

Apache mod-rewrite is an Apache web server module for URL-rewriting which we use to proxy all our http://beta.visiblelogistics.com/foo traffic to http://our-internal-server:8080/vl/foo for instance.

This is the last thing to fix, and just like Url Rewrite Filter, it defaults to decoding the URL for us, and reencoding the rewritten URL for us, which is wrong, as decoded URLs cannot be reencoded.

There is one way to get around this however, at least in our case since we are not switching one URL part for another, we do not need to decode a path part and reencode it into a query part for example: do not decode and do not reencode.

We accomplished it by using THE_REQUEST for URL-matching which is the full HTTP request (including the HTTP method and version) undecoded. We just take the URL part after the host, change the host and prepend the /vl prefix, and tada:

    ...

	# This is required if we want to allow URL-encoded slashes a path segment
	AllowEncodedSlashes On
	
	# Enable mod-rewrite
	RewriteEngine on
	
	# Use THE_REQUEST to not decode the URL, since we are not moving
	# any URI part to another part so we do not need to decode/reencode
	
	RewriteCond %{THE_REQUEST} "^[a-zA-Z]+ /(.*) HTTP/\d\.\d$"
	RewriteRule ^(.*)$ http://our-internal-server:8080/vl/%1 [P,L,NE]

####Conclusion

We hope to have clarified some URL myths and common mistakes. Short of clarifying them, making it obvious that it is not as simple as some people think will suffice. We have illustrated common mistakes in Java, and in the whole chain of a web application deployment. Now everyone is a URL expert and we hope never to see related bugs again. Please SUN, please: add standard support for URL encoding/decoding part by part.

####Link

[URL](http://en.wikipedia.org/wiki/Uniform_resource_locator)

[URL encoding](http://en.wikipedia.org/wiki/Percent-encoding)

[HTTP](http://en.wikipedia.org/wiki/HTTP)

[Java](http://en.wikipedia.org/wiki/Java_%28programming_language)

[URL First Sepecification](http://tools.ietf.org/html/rfc1738)

[URL schemes](http://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml)

[Matrix parameters](http://www.w3.org/DesignIssues/MatrixURIs.html)

[RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer)

[Yahoo RESTful API documention](https://developer.yahoo.com/social/rest_api_guide/uri-general.html)

[HTML forms](http://www.w3.org/TR/html401/interact/forms.html)

[RFC 1738](http://tools.ietf.org/html/rfc1738)

[RFC 1630](http://tools.ietf.org/html/rfc1630)

[RFC 1808](http://tools.ietf.org/html/rfc1808)

[RFC 2396](http://tools.ietf.org/html/rfc2396)

[RFC 2732](http://tools.ietf.org/html/rfc2732)

[Character encoding](http://en.wikipedia.org/wiki/Character_encoding)

[ASCII](http://en.wikipedia.org/wiki/ASCII)

[Unicode](http://en.wikipedia.org/wiki/ASCII)

[Universal Coded Character Set](http://en.wikipedia.org/wiki/Universal_Character_Set)

[UTF-16](http://en.wikipedia.org/wiki/UTF-16)

[UTF-8](http://en.wikipedia.org/wiki/UTF-8)

[ Uniform Resource Identifier (URI): Generic Syntax](http://tools.ietf.org/html/rfc3986)

[HTTP Client](http://hc.apache.org/httpclient-3.x/)

[Apache Commons Codec](http://commons.apache.org/proper/commons-codec/)

[UrlRewriteFilter](http://tuckey.org/urlrewrite/)

[Seam](http://www.seamframework.org)

[Apache mod_rewrite](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html)

[Apache web server](http://httpd.apache.org)

[原文地址](http://blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding)


