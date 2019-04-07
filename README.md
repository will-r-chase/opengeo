
<!-- README.md is generated from README.Rmd. Please edit that file -->

# opengeo

<!-- badges: start -->

<!-- badges: end -->

The goal of opengeo is to provide a standardized and hassle-free
interface for geocoding and reverse geocoding in R. Currently this is a
proto-package which is still in the planning phase of development. This
repository contains the bones of a package, but it is not functional.
Currently, this will serve as a place for discussion and planning, and
hopefully will enter active development in the near future. Please
engage with discussion in the **issues** section of this repository,
after reviewing our [rules for discussion](#discussion-guidelines). I do
not have time to develop the package by myself, and am hoping to make
this a community driven effort. We are actively seeking maintainers and
contributers.

Some definitions before we begin:

  - Geocoding: returning the longitude and latitude given a place name
  - Reverse geocoding: returning the place name of a given longitude and
    latitude

*Note that I will often use ‘geocoding’ to refer to both geocoding and
reverse geocoding*

Traditionally, commercial services like Goolge Maps were used for these
tasks through the R package `ggmap`. However, Google Maps recently
required users to provide an API key for their service, and then reduced
the free tier of their service and increased the price of geocoding by
1,400%. Apparently it is still possible to perform small tasks for free
with the Google Maps API, but they have designed their interface to
compel users to provide their credit card information, which makes many
people justifiably uncomfortable. In any case, large geocoding tasks are
now impossible on Google Maps without paying huge sums.

There are some open-source geocoding services, but the capabilities for
interfacing with them through R are hit or miss. Additionally, most
open-sources providers smartly limit the number of requests per some
time period to avoid people clogging their servers. This has led the
community to develop several “hacks” for geocoding and reverse
geocoding, but these solutions are scattered, brittle, and may require a
variety of expertise that the average user does not possess. The
overarching goal of `opengeo` is to collect these various solutions and
provide a flexible, user-friendly, and standardized interface that can
apply to a wide range of geocoding tasks without worrying about API
keys, complex geospatial operations, or any of the other technical
annoyances of geocoding. I understand that these are lofty goals which
might not all be possible. We are aiming high, and will evaluate
compromises if it becomes clear that we cannot reasonably achieve all of
our goals. With that said, we’re also prioritzing the [core design
principles](#core-design-principles), and note that speed or accuracy
are not among these goals. While we will certainly aim to have as much
speed and accuracy as possible, but it is not realistic to expect our
solutions to give geocoding down to extremely fine detail levels (ie
street address), this is simply not possible without extremely
sophisticated systems. We do aim to be transparent about the level of
accuracy we can achieve, but we will always prioritize scalability and
openness.

-----

## Contributing

I am neither a geospatial expert nor a programming expert, so this
package will rely heavily on community contribution. I am happy to guide
the overall design of the package, to contribute code and documentation,
and to coordinate the various contributers. But I have neither the time
nor expertise to realize this package by myself. I am actively seeking
both *maintainers* and *contributers* for `opengeo`. The role of a
*contributer* is to contribute code or documentation. We are
particularly interested in anyone that has solutions for geocoding or
reverse geocoding that fulfill our [Core design
principles](#core-design-principles). The role of a *maintainer* is to
provide feedback and code-review of contributed solutions, standardize
the implementation of solutions, and manage pull requests and merge
conflicts. The most important job of *maintainers* is to prioritize and
synthesize the different methods of geocoding into high-level
user-facing functions. If you are interested in becoming a *maintainer*
please get in touch with me, I need your help desperately.

### How to contribute

We are particularly interested in anyone that has solutions for
geocoding or reverse geocoding that fulfill our [Core design
principles](#core-design-principles). In the future we may welcome pull
requests, but at the moment, please post any solutions in the **issues**
page of this repository. We encourage solutions that are well commented,
and follow the
<a href="https://style.tidyverse.org/" target="_blank">tidyverse style
guide</a> (if you want an easy way to style your code, check out the
`styler` package). We also welcome you to request features or comment on
the design/implementation of `opengeo`. Please do so in the **issues**
section, and be sure to tag your posts and follow the [discussion
guidelines](#discussion-guidelines).

### Discussion guidelines

Please post discussion, feature requests, questions, or contributions in
the **issues** section. Be sure to tag your posts appropriately using
the provided tags. I will try to moderate the discussion and respond as
much as possible, and it will make my life infinitely easier if you use
tags and clearly explain your motivations. *Maintainers* are also
welcomed to moderate discussion. I will not tolerate **any** harassment
or rudeness in the discussion section. This is meant to be a welcoming
environment for newcomers and underrepresented people, so please be
curtious, keep in mind that others might not have the same level of
experience as you, and attenuate any snark accordingly.

-----

## Challenges

Geocoding is hard. There are two main challenges: accuracy and coverage.
Accuracy refers to the correctness of the returned latitude/longitude or
place-name. Coverage refers to the number of queries that return a
match. There is often a trade-off between accuracy and coverage. For
example, if you can identify the nearest city to a given set of
coordinates, but that city is a large distance away from the
coordinates, should you return the city, perhaps sacrificing accuracy,
or say no city was found, sacrificing coverage? `opengeo` will attempt
to address both issues to some degree. For many open-source APIs and
custom hacks, coverage is a serious problem. For example, I recently
tried reverse geocoding 100,000 unique locations within the United
States, and the open-source Photon API only returned a match for \~10%
of the queries. `opengeo` will address the coverage issue by combining
the results of multiple methods (more details below). Accuracy is a
tricky problem. If relying on a web-based API, accuracy information is
often unavailable. But if using a manual geocoding that tries to match
locations to places based on a distance-matrix with a local dataset,
accuracy can be reported as the ditance between the query and the match.
`opengeo` will attempt to address accuracy using distance reporting or
estimation when possible, and informative messages or warnings when no
other option is available.

-----

## Core design pinciples

The vision for this package is to collect and combine multiple solutions
for geocoding, but it is not to be a grab-bag of scattered and
disorganized functions. While specifics remain to be determined, below
are the core principles (in no particular order) that will guide the
design and development of `opengeo`. Any contributed code must follow
these guidelines, with an understanding that some principles may have to
be sacrificed to further others. If your solution violates any of these
principles, please provide an explanation as to why and it will be
considered by maintainers.

### Open source

This might be obvious given the name, but first and foremost `opengeo`
will rely only on open-source solutions that do not require API keys or
authentication tokens. We also hope to provide solutions that are
entirely *offline*, since many geocoding tasks contain sensitive
information that should not be sent over the cloud.

### Standardization

`opengeo` will standardize solutions so that there are only a few
user-facing functions which all have standardized and intuitive inputs
and outputs. There will be only one way to accomplish each task in
`opengeo`. This will inevitably lead to a very high-level interface, but
that is OK with me.

### Scalable

`opengeo` will be infinitely scalable. If you want to geocode 1 billion
locations, it will be possible with `opengeo` (although it might take a
while 😉). It is acceptable to include solutions that use APIs which
might throttle the requests per reasonable time period, as long as they
are encapsulated in functions that ship groups of requests and sleep
inbetween to avoid lockout. A “reasonable” time period is obviously
subjective, but I consider anything less than daily caps to be
acceptable. Generally, services that *throttle* requests, like Photon,
are acceptable, but services that *cap* requests, like Bing or Mapquest,
are not acceptable. This is open for discussion.

### Only do what is needed

`opengeo` will have multiple levels of detail for geocoding, and will
only do what is strictly required. The specifics of this are up for
discussion, but I think it seems sensible to allow users to specify if
they want results at the following levels of detail: continent, country,
county (USA), city. This will allow solutions that are faster and more
reliable if a user only wants a higher level of detail, and although
continent or country level detail might not be considered typical
geocoding, I think it’s useful for many users.

### Reduce user burden

Whenever possible, `opengeo` will aim to reduce the burden and pain on
the user. The general assumption should be that a novice R user could
geocode their data with a few lines of code. Users will not be required
to heavily pre-process their data, or have expertise in geospatial
analysis or other domain knowledge. One example of how to reduce user
pain is automatically reducing an input dataset to unique locations.
Users might provide a full dataset that has many duplicate locations,
but to improve speed it’s better to only geocode unique locations.
`opengeo` will handle duplicate removal without any user interaction,
and then automatically join the results to the full input dataset.

### Transparency

`opengeo` will be transparent about exactly how geocoding is performed,
and the limitations of our methods. This includes writing informative
documentation, informative messages and warnings, and providing as much
accuracy/uncertainty information as possible.

-----

## Existing solutions

There are lots of existing solutions to geocoding in R, here I’ll list
some that might be useful to incorporate or not.

  - `ggmap` is the solution that interfaces with Google Maps API,
    already discussed why this is not a good solution for us
  - `revgeo` interfaces with the Bing and Photon APIs for reverse
    geocoding. Bing has daily caps which I think are limiting, but
    Photon does not, it does have some issues with coverage though
  - `nominatim` is by Bob Rudis and interfaces with the Mapquest
    Nominatim API. The documentation in the R package says you need an
    API key and requests are limited to 15,000 per month, but I can’t
    find these usage details anywhere on the Nominatim wiki or Mapquest
    website. Other than vague warnings against bulk usage and a 1
    request per second limit, I can’t find any hard caps… we should look
    into this
  - `opencage` by ropensci interfaces with the OpenCage API, but this
    has a daily cap and requires an API key
  - `OfflineGeocodeR` seems promising. Uses a Docker container to
    geocode without internet from R. Seems like there would need to be a
    good bit of development however, since it’s not reasonable to ask
    users to understand how to install docker and set up a container

-----
