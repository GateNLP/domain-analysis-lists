# Domain Analysis

## Introduction
This is the updated version of the domain analysis service which is deployed on gate cloud and can be
accessed once your gate cloud account has been given the relevant permissions.  
The demo/UI version can be found here: [https://cloud.gate.ac.uk/shopfront/displayItem/url-domain-analysis](https://cloud.gate.ac.uk/shopfront/displayItem/url-domain-analysis)

In short, the service takes in text, extracts any URLs from it, and then looks through its sources to see if it has credibility
information on any of the URLs found. In long, read on.

## What does the domain analysis service aim to do?
The domain analysis service intends to bring you information from multiple sources related to whether a domain
or social media account may or may not be credible. Please note that this **does not mean** it is intended to directly tell
you whether or not a domain is credible. Instead, it aims to tell you what information multiple organisations who
analyse the credibility of online content have to say about a given domain. This is sometimes explicitly deeming a site
untrustworthy, e.g: "junk, conspiracy", sometimes merely commentary on a domain e.g: "satire" and sometimes, lists the
types of information that may have appeared on the domain itself.

We therefore collect information about domains and social media accounts from several different sources. A user is then
able to feed a/many URLs into the service. From here, we try to inform the  user whether any of the sources we collect
from have any information on the given domains or accounts, and feed this back to the user  along with any evidence the
sources may have provided. The idea is that the user can evaluate the label given itself, the source of the label
alongside any evidence to help determine how trustworthy a domain may be.

Note that the sources we collect from (and by extension, the service) do not give information on *specific endpoints*
on a domain. The aim here is to see if the domain or social media account itself is present in any of the sources we
collect from - no matter the specific endpoint. There is a caveat to this: where we say *domains*, we actually work with *URL prefixes* which can include a little more
than just the domain of a URL, but does not extend to the entire URL or specific endpoint.

An example of this distinction is the following:
- One of our sources, the Dukes Reporters' Lab, lists *thelogicalindian.com/fact-check* as a "fact checker"
- Therefore, should the user pass in a URL such as *https://thelogicalindian.com/trending/pottangi-administration-started-a-cleanliness-drive-in-deomaliodisha-31947*,
  technically, if we were only looking at the domain of both the source and the URL the user passed in, we would tell the
  user that the Reporters' Lab has listed *thelogicalindian.com* as a fact checker.
- We don't do this: instead, we make sure that the entire URL prefix listed in the source matches part of the URL prefix given by the user
- So: only if the user passed in a URL such as *https://thelogicalindian.com/fact-check/rashid-alvi-31901* would we say that Dukes
  Reporters' Lab had listed *https://thelogicalindian.com/fact-check/*  as a "fact checker" - since this is actually true.

This means there is some level of granularity on a domain, but it does not extend to a specific endpoint. It also means
across this document, where "domain" has been used, it technically refers to the more specific "URL prefix".


## Why is there a distinction between "domain" and "account"
In the case of non social platform URLs (such as https://www.independent.co.uk/) it makes sense to look at the domain itself
since generally, this is what we want to analyse for credibility. However, in the case of social media accounts of any given
platform such as https://twitter.com/Independent, it doesn't really make sense to look at the main domain - we want to analyse
The Independent here - not Twitter. There are two places where this  distinction is taken into account:
- *Data collection:* where we directly read in domains from other sources, we try to keep a mapping of equivalent social media account names
  so that these will not be ignored if a user passes a social media URL
- *Using the service:* when users send in a URL, the service looks out for any Facebook, Twitter or
  Tiktok URLs and should they be found, the linked account name rather than the domain is used for further processing


## How does the process work end to end?
- Every Sunday, a set of scripts are run to collect any of the latest data from a set of different sources about
  domains
- When a user sends a set of text to the service, it first analyses the text to see if any URLs can be found
- If any URLs can be found, for each of them:
  - If the URL is a facebook, twitter or tiktok URL, the account rather than the domain is used in further processing
  - Otherwise, the main domain is extracted, and this is used for further processing
  - The service then checks the domain/account to see if it is present in any data from any collected sources
  - Any match found is added to the response json, allowing users to see any labels given, descriptions or evidence given by the source

## Diagram
A basic flowchart of the domain analysis process (with much of the finer detail abstracted away) is shown below.
![source-cred-process](images/SourceCredibility.png?raw=true "Source credibility process")


# Data Collection

## What sources are the data collected from?
We currently collect data from 5 sources:
- [OpenSources](https://github.com/OpenSourcesGroup/opensources)
- [Duke Reporters' Lab](https://reporterslab.org/fact-checking/)
- [The Database of Known Fakes](https://weverify-demo.ontotext.com/)
- [Global Disinformation Index](https://disinformationindex.org/)
- [EuVsDisinfo](https://euvsdisinfo.eu/)

These are split into what we term "positive", "caution" or "mixed" sources.
This does not mean the sources themselves are positive/negative etc. Instead, it tells you if that source assigns positive,
negative or both types of labels to a source.  
For example, Reporter's Lab only gives us their data on sources they deem to be fact checkers, so is labelled positive.  
EuvsDisinfo only produces data on what they label disinformation cases, so we can guarantee it will only assign negative labels to a source.
It is therefore labelled caution. This was previously labelled "negative", but we also have cases like OpenSources where 
some labels will say "junk" which are negative, but others will just list a source as "satire", which is not necessarily 
negative - but you should probably be wary of this when reading or citing anything from it. So it is labelled caution.  
Finally, we have those which are labelled "mixed", such as the GDI-MMRR source. This gives a mix of domains it considers to 
be trusted and untrusted, so the data is responds with cannot be easily categorised into "take warning" or "potentially trustworthy". 
Each response for this source must be categorised individually, so the source itself is categorised as "mixed". 


## What is the data collection process like?
The data we collect comes in many different forms. Therefore, the collection (and social media account linking) looks
a little different for each source. For example, in some cases, social media accounts are given with every domain from the
source itself. In others, they are not, so we try to link as many as we can. Some sources update regularly. Others do not.
Since this affects the consistency of the data and  may be useful to know when using the service, below are details on 
both the sources used and data collection methods from the various sources.

### Open Sources:
***Source type: caution (gives a mix of positive and negative labels, but we discard the positive on account of there being few)***

OpenSources is a resource for assessing online information sources available for public use, headed by Merrimack College.
Their ideas and methodology can be found here for assessment: https://github.com/OpenSourcesGroup/opensources .
For this source, we directly read in the sources csv they provide on their Github. This is not updated often (it does not 
seem like it has been updated in a few years, actually), so we read these every week.  

We filter out any data from “facebook”,  “twitter”, “youtube”, “google”, “instagram”, “flickr”, “tiktok” or “archive.org”
since these would need to  be dealt with separately for their accounts. However, as of yet, there have been none of these
domains in any of the data.  

We also filter out any domains which have a more neutral or positive label attached to them. This is because of the 834 domains 
opensources has labelled (assuming these are not updated), only 47 of those domains do not have labels which imply caution 
should be taken. These domains either have labels which include "reliable", or are just "political" by itself. Note that 
we keep domains which are labelled "political" *and* something else such as "political, conspiracy". By doing this, we keep 
Opensources as a source which labels domains to be treated with caution rather than a source with mixed domains. We store these 
domains along with their labels as data from "OpenSources".  

Since we are not given the social media accounts for each of the domains listed by default, the social media data
is handled a fair bit differently to other sources. Where possible, there are mappings in place which link a domain to the
relevant social media account (e.g facebook or twitter) or from a url to other urls (e.g naturalnews.com to hundred of
other urls hosting the same thing). After every collection of OpenSources domains, we see if we can find the relevant
social media mappings. If we can, we record these social media accounts as also having been given a label by OpenSources.
Of course, this is only possible where we actually have the mapping, so there will be cases where the domain may be picked up
as having been flagged by OpenSources, but the linked social media account may not be.


### Duke Reporters' Lab:
#### Source type: positive (do not try to list anything but sources they class as fact checkers)
The Duke Reporters' Lab is a centre for journalism research at Duke University. They maintain a list of fact checking websites
here: https://reporterslab.org/fact-checking/. This data collection is therefore fairly straightforward. It takes the various domains listed
on the fact checking page, and labels them as a "fact checker" as reported by Reporters' Lab. In this case, there is no
need to map between the sites given and social media pages. The page itself provides all details of linked Facebook and Twitter accounts.
Again, since this page is not updated very often, data is collected once a week.


### Database of Known Fakes:
***Source type: mixed (sources can be in here both for hosting or disproving disinformation)***

Whereas the previous two sources involve using pre-existing lists, this source works differently. The Database of Known Fakes is
a database of IFCN debunk articles in one place which can be found here: https://weverify-demo.ontotext.com .
In all cases, where they have collected a debunk article, they have tried to keep record of which urls the article lists as 
having spread the potential misinformation.

We collect the urls listed in the DBKF as having spread misinformation in these debunk articles. We then extract the
domain or social media account in these URLs and keep count of how many times that one source has appeared in the database.
Therefore, if a user enters a domain, we can tell them the number of times it appeared in the Database of Known Fakes as a potential
source of misinformation.

Since the data in the Database of Known Fakes is both automatically ingested on a huge scale and not necessarily intended
to be used this way (as opposed to the previous two sources which were pre-compiled lists we read in), normal domains and
social media domains are collected and handled separately.

We have then done three  things to reduce the amount of times
a URL is flagged as being present in a debunk article - but is actually in there for reasons *other than* having spread
disinformation (such as reporting on disinformation, or archiving it):
- *Post process the non social media domains* to only include sources which are already flagged by other organisations as being potentially
  untrustworthy (social media domains are left alone since those are more often than not, listed in the DBKF as a source rather
  than reporter of misinformation)
- *Provide the debunk articles where this domain was found as a source of misinformation*: this way, users can check that the
  debunk article actually lists the domain as a source of misinformation
- *Add a whitelist*: should a domain have been listed by the service as having appeared in the DBKF, but on checking the debunk article
  it is clear that actually, it was not listed as a source of misinformation but as something else, the domain can be added
  to an open domain whitelist. Any domains/accounts in this list will no longer be flagged with the DBKF as a source. Of course,
  if another source (such as OpenSources) highlights the domain URL, it will not be removed from *that* source, and so will
  still be highlighted from that point of view

### Global Disinformation Index
The Global Disinformation Index is an organisation which aims to provide predominantly disinformation risk ratings for
media sites around the world. Their FAQs, and with it, their methodology paper on how they come to their conclusions can be
found here: https://disinformationindex.org/about/ .
They regularly publish reports in two major areas: analysing the media market risk rating by country (analysing how trust
worthy major media sites across a nation are), and listing their examples of disinformation on media sites alongside the
advertisers who advertise on these sites.

For this source, we have taken these two types of reports and extracted various media sites from them. These reports
can be found here: https://disinformationindex.org/research/ .

#### MMRR Reports
***Source type: mixed (a domain can be here for either being low or high risk in terms of trustworthiness)***

For the media market risk rating reports, there is a table at the beginning of each report which highlights the various
domains they have studied as a part of report. An example of this can be found here: https://disinformationindex.org/wp-content/uploads/2021/12/GDI_Disinformation-Risk-Report_Kenya-Nov-21.pdf .

We extract all of these domains, and add them to our sources, with a label
saying they are present in a GDI report and a link to the relevant report. This way, the user is able to check the report
to find out more information about the given domain. In this case, a small number of reports was excluded on the basis that
the text could not be automatically extracted from the reports. A full list of the excluded files will be added soon. Note
that if any new reports are added post writing these initial scripts, the change to include these will need to be a manual
one. The manual process is relatively short, but it should be worth noting that this source is not automatically updated.
The most recent GDI MMRR report in our sources was published on June 8, 2023.

#### Ad Reports
***Source type: caution (domains are only listed if the GDI think they have spread disinformation)***

The ad reports look a little different. For these, each report consists of a series of images depicting what the GDI deem
to be some kind of disinformation. Alongside this, they have the site it was hosted on (and also the companies which
advertise on these sites - one of their aims is to advise brands or where not to advertise), and the "type" of disinformation
they consider it to be. An example of this can be found here: https://disinformationindex.org/wp-content/uploads/2020/12/Dec_18_2020-DisinfoAds-Popular-Brands-Election-disinformation.pdf .
We therefore collect the domains from these reports and list them as present in a GDI ad report, alongside giving the
types of misinformation they have been said to host, and links to the reports so users can check for themselves whether
this is valid. We have to both post process and exclude results in some cases. This is because the reports do not always give
the direct URL: instead, they give something like "American Greatness" for "https://amgreatness.com/". At this point in time,
since this source only generates a few domain paths, it is possible to manually check which site each name is referring to by
checking the report. But both this and the process to update the source to include more results are manual. Once again
(excluding the post processing checks), the manual update step is relatively simple, but is worth mentioning since it means
this step will not be auto-updated every week like other sources. The most recent GDI Ad report in our sources was published 
on January 4, 2023.


### EUvsDisinfo (StratCom)
***Source type: caution (domains are only listed if the StratCom think they have spread disinformation)***

EuvsDisinfo (https://euvsdisinfo.eu/about/) was set up in 2015. One of their main aims is to identify, compile and expose
disinformation originating in pro-Russian media. As such, they have a database of over 13000 claims made by pro-Russian sources
which they then counter or endeavour to disprove in their database. Their database of: claims they consider to be disinformation,
their listed evidence for it being so and the sites/domains they claim have spread it can be found here:
https://euvsdisinfo.eu/disinformation-cases/.

We collect the data from this database, which includes domains StratCom list as having spread disinformation. Then, for any domain
entered which is present in StratCom's database as having spread disinformation, we list how many cases this
domain was present in. We also provide links to the relevant case reports so the user can assess for themselves
the evidence provided by StratCom that this domain did indeed, spread disinformation.

There is minimal post processing for this source - we mainly only remove any references to youtube links since we have
not yet implemented any way to figure out which user has posted the link based on the domain alone. This does comprise
quite a large number of the links given by StratCom, so it may later be worth implementing a way to do this.
Since StratCom can update fairly regularly, we update this data weekly (automatically). Note that this source is currently
the only one which also reports on the spread of information through telegram accounts - something we also now extract and
store alongside facebook, twitter and tiktok accounts listed.

# API details

## Request/response structure
The service can be accessed by the following API:

**REQUEST TYPE:**  `POST`    
**ENDPOINT:**  `https://cloud-api.gate.ac.uk/process/url-domain-analysis`    
**REQUEST DETAILS:** The request must be sent with Basic HTTP authentication. The username and password are an API key
and password which can be generated or found via your GATE cloud account. The request body should be plain text, and should
include any URLs your would like analysed.


**RESPONSE STRUCTURE**  
The service will send the following structure back. Should no URLs be found, the same structure will be returned, but the
```entities``` object will be empty
```
{
  "text": (string) original text sent by the user
  "entities":  { 
    "URL": ( [UrlObject] ) list of objects representing each URL found in the text,
    "SourceCredibility": ( [SourceCredibilityObject] ) list of objects showing information on credibility for different domains
  }
}
```

A ```URL``` object is added to the list for each individual URL found in the original text. This object has the following structure:
```
{
  "indices":          ([int, int]) start and end position in text where this URL is found
  "rule":             (string) to be ignored - used for debugging
  "category":         (string) from the POS tagger. indicates if token is NN, JJ etc. always URL for URLs
  "kind":             (string) type of token (word, number etc.) always URL for URLs
  "length":           (int) length of URL string
  "string":           (string) the URL itself
  "replaced":         (int) number of tokens merged to create URL
  "resolved-url":     (string) the URL once resolved to final location: in case URL entered redirects.
  "resolved-domain":  (string) domain of URL being evaluated
}
```

There could be several different sources which have a label for any URL. If a credibility label is found for any of the
URLs in the text, one domain analysis object is generated per source-domain pair. This means if you have information
from two different sources (e.g: Open Sources and Reporters Lab) on the same URL, you will get two individual objects in
the list. A ```SourceCredibility``` object has the following structure:
```
{
  "indices":        ([int, int]) start and end position in text where this URL is found
  "rule":           (string) to be ignored - used for debugging
  "category":       (string) from the POS tagger. indicates if token is NN, JJ etc. always assigned URL for URLs
  "kind":           (string) type of token (word, number etc.) always assigned URLs
  "length":         (int) length of URL string
  "string":         (string) the URL itself
  "replaced":       (int) number of tokens merged to create URL
  "resolved-url":   (string) the URL once resolved to final location: in case URL entered redirects.
  "source":         (string) the source from which this credibility information is taken
  "source-type":    (string) whether the source gives information on trusted media, media where it may be worth being wary or both.   
                    these correspond to a value of "positive", "caution" or "mixed"
  :domainOrAccount  (string) whether the credibility information in about a domain or a social media page( e.g: facebook, twitter).  
                     can be "domain" or "account"
  :updated          (string, format: yyyyMMdd) when the data from this source was last updated
  :labels*          (string) the labels given to the domain by this source. comma separated.
  :description      (string) any extra information given by the source about the domain path
  :evidence*        (string) usually urls giving potential evidence that the label given by the source is correct. comma separated.
}
```

**NB: this version includes the fields "labels" and "evidence". These replace what was in previous versions, "type" and "debunks".
These labels have now been deprecated, but for the case of backward compatability can still be accessed from the response
(with the same values).
They will over time however, be removed from the response.  If you happen to have used either of these in development,
please update to the latest: "labels" and "evidence".**

In the domain analysis object part of the response
- The source-type field can be
  - positive
  - caution
  - mixed
- The source field can be
  - Duke Reporters' Lab
  - OpenSources
  - DBKF
  - StratCom
  - GDI-Ads
  - GDI-MMRR
- The following fields **are optionally returned**:
  - description 
  - evidence 
  - All other fields should be present in all responses.

## Example
Assuming I sent in the following URL to the service: https://bit.ly/3oZRRyt .
The service will forward this URL to its final location (a news article on https://www.dailytelegraph.com.au), recognises
that "dailytelegraph.com.au" is the domain to look for and sends back the following:

```json
{
  "text": "https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE170_a&amp;dest=https%3A%2F%2Fwww.dailytelegraph.com.au%2Fnews%2Fnsw%2Ftreasurer-matt-kean-will-hand-down-the-halfyearly-budget-review%2Fnews-story%2Fcdaa6d5cc30a6dbc38951f4d44d53b09&amp;memtype=anonymous&amp;mode=premium&amp;v21=dynamic-cold-control-noscore&amp;V21spcbehaviour=append",
  "entities": {
    "URL": [
      {
        "indices": [
          0,
          72
        ],
        "rule": "URL",
        "category": "URL",
        "kind": "URL",
        "length": 72,
        "string": "https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "replaced": 24,
        "resolved-url": "https://www.dailytelegraph.com.au/remote/check_cookie.html?url=https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "resolved-domain": "www.dailytelegraph.com.au"
      }
    ],
    "SourceCredibility": [
      {
        "indices": [
          0,
          72
        ],
        "rule": "URL",
        "updated": "20211214",
        "source-type": "caution",
        "labels": "bias,rumor",
        "description": "australian tabloid mag",
        "type": "bias,rumor",
        "source": "OpenSources",
        "replaced": 24,
        "resolved-domain": "www.dailytelegraph.com.au",
        "resolved-url": "https://www.dailytelegraph.com.au/remote/check_cookie.html?url=https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "domainOrAccount": "domain",
        "category": "URL",
        "string": "https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "kind": "URL",
        "length": 72
      },
      {
        "indices": [
          0,
          72
        ],
        "source-type": "caution",
        "debunks": "https://disinformationindex.org/wp-content/uploads/2021/09/GDI_QUT-Australia-Disinformation-Risk-Assessment-Report-21.pdf",
        "labels": "present in GDI news reports",
        "kind": "URL",
        "description": "This domain has appeared in 1 GDI media market based report(s)",
        "resolved-url": "https://www.dailytelegraph.com.au/remote/check_cookie.html?url=https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "length": 72,
        "replaced": 24,
        "string": "https://www.dailytelegraph.com.au/subscribe/news/1/?sourceCode=DTWEB_WRE",
        "evidence": "https://disinformationindex.org/wp-content/uploads/2021/09/GDI_QUT-Australia-Disinformation-Risk-Assessment-Report-21.pdf",
        "type": "present in GDI news reports",
        "updated": "20211215",
        "resolved-domain": "www.dailytelegraph.com.au",
        "source": "GDI-MMR",
        "domainOrAccount": "domain",
        "rule": "URL",
        "category": "URL"
      }
    ]
  }
}
```

It has found information about the daily telegraph (au) in data collected from OpenSources and the GDI and returned the
relevant details in the domain analysis objects.
`
