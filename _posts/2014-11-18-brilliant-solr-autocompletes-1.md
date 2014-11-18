---
layout: post
author: brandonwestcott
title: "Brilliant Autocompletes with Apache Solr - Part 1: Crafting your index"
description: "Crafting a Solr index for brilliant autocompletes"
tags: [solr, apache solr, autocomplete, typeahead, fuzzy search, lucene]
comments: true
---
Whether you are looking for a new restaurant on tripadvisor, your future home on zillow, or a doctor or healthcare facility on [vitals.com](http://www.vitals.com/), many sites today rely on search indexes to power these experiences. Over the last 3 years, our consumer website of [vitals.com](http://www.vitals.com/) as well as our [Vitals Choice](http://www.vitals.com/about/healthplans) SaaS-based health care transparency and member engagement platform have utilized Apache Solr for such functionality. Having been one of the primary engineers involved in crafting these search indexes, I have decided to write my first set of blog posts about my experiences on building a splendid Solr search. While there are many features of Solr (and under the hood Lucene) to fall in love with, this series of posts will primarily focus on full text searching and how at Vitals we have structured, queried, and analyzed our Solr documents for our autocomplete searches. After some basics, today's post will focus on crafting your index by creating custom field types for use in autocompletes. The next post in this series will be primarily focused on constructing queries against the fields we have created during this post. Along the way, I will be detailing some of the tools and methods we have used for analyzing and understanding what’s happening in our Solr documents. Let’s get started:

In Solr, our primary resource is a [document](https://cwiki.apache.org/confluence/display/solr/Overview+of+Documents%2C+Fields%2C+and+Schema+Design), which is defined by a schema comprised of a set of fields with given types. [Field types](https://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties) are the most important piece to explore as they tell Solr how we want to treat our data coming in. Utilizing the right field types is imperative in creating a great search experience. While Solr does come with basic types for our standard data types (strings, floats, booleans), our needs require us to create custom types.

A given fieldType can contain a number of properties (described in detail [here](h ttps://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties)) as well as a set of powerful tokenizers and filters that manipulate our data during index and query analysis. Lets take a look at our very basic text field type definition:

~~~xml
<fieldType name="text" class="solr.TextField"
  positionIncrementGap="100"
  stored="false"
  multiValued="true"
  indexed="true"
>
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
~~~

Inside of our fieldType node, we have 2 analyzers - one for indexing, one for querying (to note, when both index and query are the same definition, you can specify a single analyzer omitting the type attribute). Solr works primarily in two simple ways - you give it information and then ask it questions to find information. Providing Solr documents is done through "indexing" and finding those documents at a later time is done through querying. These set of tokenizers and filters take an input and process it through the pipeline above producing a set of tokens that are indexed and queried with.  Again, back to the wiki for [some in depth info]( https://cwiki.apache.org/confluence/display/solr/Understanding+Analyzers%2C+Tokenizers%2C+and+Filters).

In my experience, crafting a great autocomplete and fuzzy matching experience is all about creating a number of fieldTypes that contain specific rules to answer specific parts of a query. When you bring these many parts together, you get both a wide range of questions that can be asked combined with the granularity needed to provide your users the most relevant results. Let’s dig into some of the different fields we use and what their function is.

The basic text field above is our standard, fully tokenized field ([what are tokens?](https://cwiki.apache.org/confluence/display/solr/Understanding+Analyzers%2C+Tokenizers%2C+and+Filters)). In this field, a text value (at both query time and index time) goes through the following steps:

* StandardTokenizer - This tokenizer splits the text field into tokens, treating whitespace and punctuation as delimiters.
* ASCIIFoldingFilterFactory - Converts non "Basic Latin" Unicode alphabetic, numeric, and symbolic character into their ASCII equivalents
* LowerCaseFilterFactory - makes all characters lowercase.

Given the following input, Andrés Iniesta, the two tokens that would be created are *andres* and *iniesta*. In this case, if we had indexed *Andrés Iniesta*, and the user had searched *ANDRES INIESTA*, we would have a match. Understanding what a field does is fundamental in building your index, which is why Solr comes built in with a tool to do this - awesome right!?

On any server with Solr 4, in admin select your core from the left and then click the **Analysis** button, taking you to a path like `/#{instance}/#/#{core}/analysis`. Here you can select any of your FieldTypes and input both the Indexed Value and the Query Value. After clicking "Analyse Values", you will get a step by step view into how a given input is transformed into its final indexed/queried value. In addition, the interface will indicate which tokens have matched between the index and query inputs. This is an invaluable tool in understanding a set of complex filters and how user inputs will match or not match your indexed values. Here is a screenshot using our text field above with the *Andrés Iniesta* example:
![Solr Analysis Example]({{ site.url }}/images/posts/solr-analysis.png)
Here you can see the different steps and tokens created during each step, with the final line containing the indexed and queried value. As expected, both parts of our input matched so both indexed tokens are highlighted blue indicating a successful match.


Now that we understand how to analyze our fieldTypes, let's look at a few more definitions that are used in our searches.

~~~xml
<fieldType name="text_autocomplete" class="solr.TextField"
  positionIncrementGap="1"
  stored="false"
  multiValued="true"
  indexed="true"
>
  <analyzer type="index">
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
    <filter class="solr.WordDelimiterFilterFactory"
      generateWordParts="1"
      generateNumberParts="1"
      catenateWords="0"
      catenateNumbers="0"
      catenateAll="0"
      splitOnCaseChange="0"
    />
    <filter class="solr.EdgeNGramFilterFactory"
      minGramSize="1"
      maxGramSize="15"
      side="front"
    />
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
    <filter class="solr.WordDelimiterFilterFactory"
      generateWordParts="1"
      generateNumberParts="1"
      catenateWords="0"
      catenateNumbers="0"
      catenateAll="0"
      splitOnCaseChange="0"
    />
  </analyzer>
</fieldType>
~~~

This field is a modification of the text field above, crafted specifically for autocomplete. At index time, after going through the basic filters, Solr applies the EdgeNGramFilterFactory filter for our splits words down into partial gram token as specified by the params. In our case, we create tokens for each sequential character (minGramSize="1") up to a total of 15 sequential character (maxGramSize="15"). So for example, using our **Andrés Iniesta** indexed value, we would get the following tokens: `a an and andr andre andres i in ini inie inies iniest iniesta`. Now, if the user is doing a traditional typeahead and enters "Inies" we would get a match on the "inies" token. Note that here, we aren't applying the EdgeNGramFilter on the user input as doing so would result in a ridiculous amount of matches - "Inies" would be broken down and any name that started with an "i" would be matched. Also to note, that autocomplete behavior can also be achieved with the wildcard (\*) character. However, this approach limits the flexibility of what can be queried and is less performant than using EdgeNGramFilterFactory when there are just a couple of tokens.

Doing a normal edismax query using the two fields above will get a pretty good, functional autocomplete. However, in some cases that just might not be enough. For instance, our autocompletes for medical specialties contain additional fuzzy and sounds like/ phonetic matches to help users (and myself) spell out these rather tricky words - Gastroenterologists, Endocrinologists, Ophthalmologists, Rheumatologists, Maxillofacial Surgeons - yikes!

Solr comes with a half dozen or so different phonetic algorithms. The "best" algorithm to use is completely dependent upon the requirements of the situation. At Vitals, we have primarily relied on the BeiderMorse and DoubleMetaphone algorithms for phonetic matching. Here is our BeiderMorse based field that is used for our phonetic specialty queries:

~~~xml
<fieldType name="text_bmpm" class="solr.TextField"
  positionIncrementGap="100"
  stored="false"
  multiValued="true"
  indexed="true"
>
  <analyzer>
    <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.ASCIIFoldingFilterFactory"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.BeiderMorseFilterFactory"
        nameType="GENERIC"
        ruleType="APPROX"
        concat="true"
        languageSet="auto"
      />
  </analyzer>
</fieldType>
~~~

Again, very similar to a regular text field, but applies the phonetic algorithm before indexing and querying. I suggest defining all the phonetic field types in your Solr config and testing them out with the Analysis tool in order to find which one is right for your situation. To take phonetics to the next level, here is our phonetic typeahead field for specialties.

~~~xml
<fieldType name="text_bmpm_autocomplete" class="solr.TextField"
  positionIncrementGap="1"
  stored="false"
  multiValued="true"
  indexed="true"
>
  <analyzer type="index">
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
    <filter class="solr.WordDelimiterFilterFactory"
      generateWordParts="1"
      generateNumberParts="1"
      catenateWords="0"
      catenateNumbers="0"
      catenateAll="0"
      splitOnCaseChange="0"
    />
    <filter class="solr.neiderMorseFilterFactory"
      nameType="GENERIC"
      ruleType="APPROX"
      concat="true"
      languageSet="auto"
    />
    <filter class="solr.EdgeNGramFilterFactory"
      minGramSize="1"
      maxGramSize="15"
      side="front"
    />
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory" />
    <filter class="solr.WordDelimiterFilterFactory"
      generateWordParts="1"
      generateNumberParts="1"
      catenateWords="0"
      catenateNumbers="0"
      catenateAll="0"
      splitOnCaseChange="0"
    />
    <filter class="solr.BeiderMorseFilterFactory"
      nameType="GENERIC"
      ruleType="APPROX"
      concat="true"
      languageSet="auto"
    />
  </analyzer>
</fieldType>
~~~

This allows our autocompletes to easily take advantage of the phonetic features. The indexed value for this field ends up being gram tokens based on the output of the BeiderMorse filter. For instance, given we had a stored value of "Chiropractor", our user could enter "Kirop" and get a successful phonetic match.

The final part of our autocomplete relies on fuzzy matching capabilities of Solr. While there are some filters that can be utilized for this purpose, such as NGramFilterFactory, our implementations have instead relied on the fuzzy matching operator (~), which applies the levenshtein distance algorithm against already existing text fields in our index. While creating bi-grams or tri-grams with NGramFilter can be an elegant and performant solution on large text fields, our fields usually contained less than a handful of tokens. That combined with the fact that lucene got some [big performance boosts to levenshtein in 4.0](http://blog.mikemccandless.com/2011/03/lucenes-fuzzyquery-is-100-times-faster.html), it was clearly the right choice for us. 

So with the few fields we have above, we are now ready to start building out Solr queries.  In part 2, we will look at how we’ve built up the queries against these fields, and some other tips and tricks we’ve used to create brilliant autocompletes.  In addition, the next post will contain a detailed look into one of my favorite tools for understanding Solr queries and responses.
